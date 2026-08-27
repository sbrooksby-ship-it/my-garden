2026-08-27 11:13

Status:

Tags:[[DP13]] [[Github]]


# Automove gs adverse

// =========================================================================

// ONE-TIME SETUP: Run this function once to build the 1-hour trigger!

// =========================================================================

function setupHourlyTrigger() {

  var triggers = ScriptApp.getProjectTriggers();

  // Check if we already created the trigger so we don't make duplicates

  for (var i = 0; i < triggers.length; i++) {

    if (triggers[i].getHandlerFunction() === "moveAndArchiveTranscripts") {

      console.log("Trigger already exists! No action needed.");

      return;

    }

  }

  // Create the 1-hour timer

  ScriptApp.newTrigger("moveAndArchiveTranscripts")

    .timeBased()

    .everyHours(1)

    .create();

  console.log("SUCCESS: 1-hour trigger has been automatically created!");

}

  

// =========================================================================

// MAIN FUNCTION: Moves and archives transcripts ONLY (skips audio)

// =========================================================================

function moveAndArchiveTranscripts() {

  var sourceFolderId = "1OC6DngtZwWse5o9DTqI8P2sIiSddn5vv";

  var privateFolderId = "1JeiaiAvezuqHqEjo2UPavaUA2e7_LaTR";

  var handledFolderId = "1I4Xfnuvm31-rhEXypFVqFkpqzjR-a5nU";

  var sourceFolder = DriveApp.getFolderById(sourceFolderId);

  var privateFolder = DriveApp.getFolderById(privateFolderId);

  var handledFolder = DriveApp.getFolderById(handledFolderId);

  var files = sourceFolder.getFiles();

  while (files.hasNext()) {

    var file = files.next();

    var fileName = file.getName();

    // =========================================================================

    // FILTER: Skip audio files (.wav) so they stay in the source folder

    // =========================================================================

    if (fileName.toLowerCase().endsWith('.wav') || file.getMimeType().indexOf('audio') !== -1) {

      Logger.log("Audio file detected, leaving in Source folder: " + fileName);

      continue; // Skip copy/move and go to next file

    }

    try {

      // 1. Check if it already exists in the Private folder

      var existingPrivateFiles = privateFolder.getFilesByName(fileName);

      // 2. Only copy if it does NOT exist yet

      if (!existingPrivateFiles.hasNext()) {

        file.makeCopy(fileName, privateFolder);

        Logger.log("Copied to Private: " + fileName);

      } else {

        Logger.log("Already in Private, skipping copy: " + fileName);

      }

      // 3. Always move the original out of the Source folder

      file.moveTo(handledFolder);

      Logger.log("Moved to Handled: " + fileName);

    } catch (e) {

      Logger.log("Error processing " + fileName + ": " + e.message);

    }

  }

}


### References
