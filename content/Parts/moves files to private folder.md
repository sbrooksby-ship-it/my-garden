2026-08-27 11:41

Status:

Tags: [[Google Script]]


# moves files to private folder


/**

 * ============================================================================

 * GOOGLE APPS SCRIPT: AI CALL TRANSCRIPT AUTO-FILER (WITH AUDIO PROTECTION)

 * ============================================================================

 *

 * WHAT IT DOES:

 *  Every 5 minutes, checks the "AI Call Transcripts" folder for any files

 *  sitting directly in it (i.e. not yet filed into a week subfolder).

 *  For each new file found:

 *    1. Skips any .wav audio files completely.

 *    2. Copies it into your private folder (backup/archive copy).

 *    3. Parses the date from the filename (expects a leading

 *       YYYY-MM-DD, e.g. "2026-08-17T14_08_32.143Z_9311109.txt").

 *    4. Figures out which Mon–Fri work-week that date falls in and

 *       builds the matching subfolder name, e.g. "8/17-8/21".

 *    5. Finds that subfolder under "AI Call Transcripts" (creating it

 *       if it doesn't exist yet) and MOVES the original file into it.

 *

 *  Files that don't match the expected filename date pattern are left

 *  alone (not moved, not copied) so nothing gets mis-filed silently.

 *

 * SETUP:

 *  1. Paste this into a new Apps Script project (script.google.com).

 *  2. Update PRIVATE_FOLDER_ID / SOURCE_FOLDER_NAME below if needed.

 *  3. Run `setupTrigger` once from the editor (top toolbar ▶ Run,

 *     function = setupTrigger). Authorize when prompted.

 *  4. Done — it will now run automatically every 5 minutes.

 *

 *  To stop it: run `removeTrigger` once.

 * ============================================================================

 */

  

// ── CONFIGURATION ─────────────────────────────────────────────────────────────

  

// Your private/backup folder — copies of every new file go here.

const PRIVATE_FOLDER_ID = "1syH-GIOSnJwcJRIXa8iMva_ULGtqg-w4";

  

// The folder new transcripts land in. Looked up by name (must be unique

// in your Drive). If you'd rather pin it by ID, set SOURCE_FOLDER_ID

// below and it will be used instead.

const SOURCE_FOLDER_NAME = "AI Call Transcripts";

const SOURCE_FOLDER_ID   = ""; // optional — leave blank to use the name lookup above

  

// ── MAIN WORKER ────────────────────────────────────────────────────────────

  

function checkForNewFiles() {

  const sourceFolder = getSourceFolder();

  const privateFolder = DriveApp.getFolderById(PRIVATE_FOLDER_ID);

  

  // getFiles() only returns files DIRECTLY in this folder — files already

  // moved into a week subfolder won't show up here again.

  const files = sourceFolder.getFiles();

  

  let filedCount = 0;

  let skippedCount = 0;

  

  while (files.hasNext()) {

    const file = files.next();

    const fileName = file.getName();

  

    // ==========================================

    // 🛡️ NEW SAFETY FILTER: IGNORE AUDIO FILES

    // ==========================================

    if (fileName.toLowerCase().endsWith('.wav')) {

      // It's an audio file, leave it alone and move to the next file!

      continue;

    }

  

    try {

      const fileDate = parseDateFromFilename(fileName);

  

      if (!fileDate) {

        skippedCount++;

        continue; // unrecognized filename pattern — leave it in place

      }

  

      // 1. Copy to private folder first (safety net before we touch the original)

      file.makeCopy(fileName, privateFolder);

  

      // 2. Work out the Mon–Fri week folder name

      const weekFolderName = getWeekFolderName(fileDate);

  

      // 3. Find or create that subfolder under the source folder

      const weekFolder = getOrCreateSubfolder(sourceFolder, weekFolderName);

  

      // 4. Move the original into it

      file.moveTo(weekFolder);

  

      filedCount++;

    } catch (err) {

      console.error(`Failed to process file "${fileName}": ${err.message}`);

      skippedCount++;

    }

  }

  

  if (filedCount > 0 || skippedCount > 0) {

    console.log(`Filed ${filedCount} file(s), skipped ${skippedCount}.`);

  }

}

  

// ── HELPERS ────────────────────────────────────────────────────────────────

  

function getSourceFolder() {

  if (SOURCE_FOLDER_ID) {

    return DriveApp.getFolderById(SOURCE_FOLDER_ID);

  }

  const folders = DriveApp.getFoldersByName(SOURCE_FOLDER_NAME);

  if (!folders.hasNext()) {

    throw new Error(`No folder named "${SOURCE_FOLDER_NAME}" found.`);

  }

  const folder = folders.next();

  if (folders.hasNext()) {

    console.warn(`Multiple folders named "${SOURCE_FOLDER_NAME}" found — using the first one. Consider setting SOURCE_FOLDER_ID instead.`);

  }

  return folder;

}

  

/**

 * Parses a leading YYYY-MM-DD from a filename like

 * "2026-08-17T14_08_32.143Z_9311109.txt". Returns a Date or null.

 */

function parseDateFromFilename(filename) {

  const m = filename.match(/^(\d{4})-(\d{2})-(\d{2})/);

  if (!m) return null;

  const year  = parseInt(m[1], 10);

  const month = parseInt(m[2], 10) - 1; // JS months are 0-indexed

  const day   = parseInt(m[3], 10);

  return new Date(year, month, day);

}

  

/**

 * Given a date, returns the "M/d-M/d" name of the Mon–Fri work week

 * it falls in, e.g. "8/17-8/21". Weekend dates are filed into the

 * work week that just ended (the preceding Mon–Fri).

 */

function getWeekFolderName(date) {

  const dayOfWeek = date.getDay(); // 0 = Sun, 1 = Mon, ... 6 = Sat

  

  // Days to subtract to reach that week's Monday.

  // Mon=1 -> 0, Tue=2 -> 1, ... Sat=6 -> 5, Sun=0 -> 6 (counts back to the PRECEDING Monday)

  const daysSinceMonday = (dayOfWeek + 6) % 7;

  

  const monday = new Date(date);

  monday.setDate(date.getDate() - daysSinceMonday);

  

  const friday = new Date(monday);

  friday.setDate(monday.getDate() + 4);

  

  const fmt = (d) => `${d.getMonth() + 1}/${d.getDate()}`;

  return `${fmt(monday)}-${fmt(friday)}`;

}

  

function getOrCreateSubfolder(parentFolder, name) {

  const existing = parentFolder.getFoldersByName(name);

  if (existing.hasNext()) {

    return existing.next();

  }

  return parentFolder.createFolder(name);

}

  

// ── TRIGGER SETUP ──────────────────────────────────────────────────────────

  

function setupTrigger() {

  removeTrigger(); // avoid duplicates if run more than once

  

  ScriptApp.newTrigger('checkForNewFiles')

    .timeBased()

    .everyMinutes(5)

    .create();

  

  console.log('Trigger created — checkForNewFiles will now run every 5 minutes.');

}

  

function removeTrigger() {

  ScriptApp.getProjectTriggers().forEach(trigger => {

    if (trigger.getHandlerFunction() === 'checkForNewFiles') {

      ScriptApp.deleteTrigger(trigger);

    }

  });

}

  

// Run this manually any time you want to test without waiting for the timer.

function runNow() {

  checkForNewFiles();

}



### References
