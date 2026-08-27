2026-08-27 11:43

Status:

Tags: [[gs]]


# gs private for data transfer



/**

 * ============================================================================

 * GOOGLE APPS SCRIPT: CALL EVALUATIONS + SHARED DASHBOARD SYNC (v3.24)

 * ============================================================================

 */

  

// ── CONFIGURATION ─────────────────────────────────────────────────────────────

  

const SHARED_SS_ID = "1-N0IJxjzrdM_mlmIn9QYMHj_PPMfOopP7CReYW5m5IQ";

  

const SYSTEM_TABS = ["Sheet1", "Raw Data", "Master Data", "Coaching Feedback", "Summary CF"];

  

const NAME_MAP = {};

  

// ── RUBRIC DEFINITIONS (36 Items) ───────────────────────────────────────────

  

const RUBRIC_CRITERIA = [

  { id: "BG 1",   criteria: "First 30s Tone" },

  { id: "BG 2",   criteria: "Greeting & Intro" },

  { id: "ARC 1",  criteria: "Early ARC" },

  { id: "ARC 2",  criteria: "Recognize Emotion" },

  { id: "ARC 3",  criteria: "Maintain/Elevate Tone" },

  { id: "ARC 4",  criteria: "Build Product Trust" },

  { id: "OE 1",   criteria: "Prioritize Customer Interest" },

  { id: "OE 2",   criteria: "Take Responsibility" },

  { id: "OE 3",   criteria: "Evidence of Effort" },

  { id: "PE 1",   criteria: "Customer Specific Info" },

  { id: "PE 2",   criteria: "Discovery Questions" },

  { id: "PE 3",   criteria: "Success Identification" },

  { id: "PE 4",   criteria: "Educational Solutions" },

  { id: "PE 5",   criteria: "Next Step Education" },

  { id: "QC 1",   criteria: "Natural Communication" },

  { id: "QC 2",   criteria: "Proper Acknowledgment" },

  { id: "QC 3",   criteria: "Clear/Concise/Fluent" },

  { id: "QC 4",   criteria: "Active Listening" },

  { id: "QC 5",   criteria: "Empowering Language" },

  { id: "QC 6",   criteria: "Confidence & Competence" },

  { id: "QC 7",   criteria: "Assumed Success/Sale" },

  { id: "QC 8",   criteria: "Appropriate Tone" },

  { id: "CL 1",   criteria: "Summarize Resolution" },

  { id: "CL 2",   criteria: "Clear Next Steps" },

  { id: "CL 3",   criteria: "Additional Assistance" },

  { id: "CL 4",   criteria: "Professional/Warm Close" },

  { id: "CC 1",   criteria: "Guide Conversation" },

  { id: "CC 2",   criteria: "Focus Primary Concern" },

  { id: "CC 3",   criteria: "Manage Talk Time" },

  { id: "CC 4",   criteria: "Follow Call Flow" },

  { id: "CC 5",   criteria: "VFP Achieved" },

  { id: "COMP 1", criteria: "Verified Account" },

  { id: "COMP 2", criteria: "Verified Email" },

  { id: "COMP 3", criteria: "Survey Request" },

  { id: "COMP 4", criteria: "No Medical Claims" },

  { id: "COMP 5", criteria: "Handled Complaints/AE" }

];

  

// Deep Keyword mapping if AI forgets the ID entirely (BROADENED FOR v3.24)

const CRITERIA_KEYWORDS = {

  "BG 1": "first 30", "BG 2": "greeting", "ARC 1": "early arc", "ARC 2": "emotion",

  "ARC 3": "elevate tone", "ARC 4": "product trust", "OE 1": "advocate|interest", "OE 2": "responsibility",

  "OE 3": "evidence of effort", "PE 1": "specific info", "PE 2": "discovery", "PE 3": "success",

  "PE 4": "educational", "PE 5": "next step", "QC 1": "natural", "QC 2": "acknowledge",

  "QC 3": "clear", "QC 4": "listening", "QC 5": "empowering", "QC 6": "confidence",

  "QC 7": "assumed", "QC 8": "appropriate tone", "CL 1": "summarize", "CL 2": "next step",

  "CL 3": "additional assistance", "CL 4": "warm", "CC 1": "guid",

  "CC 2": "primary concern", "CC 3": "talk time", "CC 4": "call flow", "CC 5": "vfp",

  "COMP 1": "verified account", "COMP 2": "verified email", "COMP 3": "survey",

  "COMP 4": "medical claim|treating", "COMP 5": "complaint|adverse"

};

  

// ── CUSTOM MENU ──────────────────────────────────────────────────────────────

  

function onOpen() {

  SpreadsheetApp.getUi()

    .createMenu('🎯 Call Evals')

    .addItem('⚡ Process New Raw Evaluations', 'processAllRawData')

    .addItem('📝 Format Selected Cell Evaluation', 'formatSelectedCell')

    .addSeparator()

    .addItem('🔄 Sync to Shared Dashboard Now', 'syncToSharedSheet')

    .addSeparator()

    .addItem('💬 Setup Coaching Feedback Tab', 'initCoachingFeedbackSheet')

    .addItem('📦 Archive & Reset Current Month', 'archiveMonthAndReset')

    .addSeparator()

    .addItem('⏰ Setup Auto-Sync Timer (5 min)', 'createSyncTrigger')

    .addToUi();

}

  

// ── BATCH PROCESSOR ─────────────────────────────────────────────

  

function processAllRawData() {

  const ss = SpreadsheetApp.getActiveSpreadsheet();

  const rawSheet = ss.getSheetByName("Raw Data") || ss.getSheetByName("Sheet1") || ss.getActiveSheet();

  const data = rawSheet.getDataRange().getValues();

  

  if (data.length === 0) return;

  

  let processedCount = 0;

  

  for (let i = 0; i < data.length; i++) {

    const rawCell    = String(data[i][0] || "").trim();

    const statusCell = String(data[i][1] || "").trim();

  

    if (statusCell.startsWith("PROCESSED")) continue;

  

    const looksLikeEval =

      rawCell.includes("Detailed Adherence Report") ||

      rawCell.includes("| Category |") ||

      rawCell.includes("Top 3 Wins") ||

      rawCell.includes("ID: BG") || rawCell.includes("ID: ARC") ||

      rawCell.includes("ID: OE") || rawCell.includes("ID: PE")  ||

      rawCell.includes("ID: QC") || rawCell.includes("ID: CC")  ||

      rawCell.includes("ID: CL") || rawCell.includes("ID: COMP") ||

      rawCell.includes("cl.corp.google.com"); // Added to catch URLs

  

    if (!looksLikeEval) continue;

  

    const callNameHint = data[i][2] ? String(data[i][2]).trim() : "";

    const agentHint    = data[i][3] ? String(data[i][3]).trim() : "";

  

    const parsed     = parseRawEvaluationReport(rawCell, callNameHint, agentHint);

    const agentSheet = getOrCreateAgentSheet(ss, parsed.agentName);

  

    const isNew = writeScoresToGrid(agentSheet, parsed);

    if (isNew) {

      writeCoachingFeedback(ss, parsed);

      processedCount++;

    }

  

    const ts = Utilities.formatDate(new Date(), Session.getScriptTimeZone(), "yyyy-MM-dd HH:mm");

    rawSheet.getRange(i + 1, 2).setValue("PROCESSED: " + ts).setFontColor("#274e13");

  }

  

  if (processedCount > 0) {

    ss.toast(`Added ${processedCount} new call evaluation(s)!`, 'Success', 5);

    syncToSharedSheet(true);

  }

}

  

// ── SINGLE CELL FORMATTER ────────────────────────────────────────────────────

  

function formatSelectedCell() {

  const ss      = SpreadsheetApp.getActiveSpreadsheet();

  const cell    = ss.getActiveCell();

  const rawText = cell.getValue().toString().trim();

  

  if (!rawText) {

    SpreadsheetApp.getUi().alert("Please select a cell containing evaluation text.");

    return;

  }

  

  const row          = cell.getRow();

  const sheet        = cell.getSheet();

  const callNameHint = sheet.getRange(row, 2).getValue().toString().trim();

  const agentHint    = sheet.getRange(row, 3).getValue().toString().trim();

  

  let parsed = parseRawEvaluationReport(rawText, callNameHint, agentHint);

  

  if (parsed.agentName === "Unknown Agent") {

    const tabName  = sheet.getName();

    const isSystem = SYSTEM_TABS.includes(tabName);

    if (!isSystem) {

      parsed.agentName = tabName;

    } else {

      const ui  = SpreadsheetApp.getUi();

      const res = ui.prompt('Agent Name Required', 'Enter Agent Name (e.g. Carlos Fernandes, Amon):', ui.ButtonSet.OK_CANCEL);

      if (res.getSelectedButton() === ui.Button.OK && res.getResponseText().trim()) {

        parsed.agentName = res.getResponseText().trim();

      }

    }

  }

  

  const agentSheet = getOrCreateAgentSheet(ss, parsed.agentName);

  const isNew = writeScoresToGrid(agentSheet, parsed);

  if (isNew) {

    writeCoachingFeedback(ss, parsed);

    syncToSharedSheet(true);

    ss.toast(`Formatted & synced: ${parsed.callName} (${parsed.agentName})`, 'Complete', 5);

  } else {

    ss.toast(`Skipped: The ID ${parsed.callName} already exists for ${parsed.agentName}`, 'Duplicate Blocked', 5);

  }

  

  const ts = Utilities.formatDate(new Date(), Session.getScriptTimeZone(), "yyyy-MM-dd HH:mm");

  sheet.getRange(row, 2).setValue("PROCESSED: " + ts).setFontColor("#274e13");

}

  

// ── SCORE EXTRACTOR HELPER ───────────────────────────────────────────────────

  

function extractScoreFromChunk(chunk) {

  // 1. Explicit Score indicator

  let m1 = chunk.match(/Score\s*[:\-]?\s*\*?\*?\s*([1-5]|0|N\/?A)\b/i);

  if (m1) return m1[1];

  

  // 2. Table cell | 4 |

  let m2 = chunk.match(/\|\s*\*?\*?\s*([1-5]|0|N\/?A)\s*\*?\*?\s*\|/i);

  if (m2) return m2[1];

  

  // 3. Trailing pipe block | 4

  let m3 = chunk.match(/\|\s*\*?\*?\s*([1-5]|0|N\/?A)\s*\*?\*?\s*$/i);

  if (m3) return m3[1];

  

  // 4. Bracketed [4] or (4)

  let m4 = chunk.match(/[\(\[\*]\s*([1-5]|0|N\/?A)\s*[\)\]\*]/i);

  if (m4) return m4[1];

  

  // 5. Split by pipes, check if any section is purely a number

  if (chunk.includes('|')) {

    let parts = chunk.split('|');

    for (let p of parts) {

      let cleanP = p.trim().replace(/[\*\s]/g, '');

      if (cleanP.match(/^([1-5]|0|N\/?A)$/i)) return cleanP;

    }

  }

  

  // 6. Last isolated number in the chunk

  let m5 = chunk.match(/\b([1-5]|0|N\/?A)\b(?=[^\d]*$)/i);

  if (m5) return m5[1];

  

  return null;

}

  

// ── THE "CHUNKING" PARSER ENGINE (v3.24) ─────────────────────────────────────

  

function parseRawEvaluationReport(rawText, fallbackCallName, fallbackAgentName) {

  // ── URL RESCUER ──

  // Intercept fake Google URLs and stitch the ID back together before the script runs.

  let cleanText = rawText.replace(/\b(BG|ARC|OE|PE|QC|CL|CC|COMP)\b[\s\|]*https?:\/\/[^\|\s]*?cl\.corp\.google\.com\/(\d+)[^\|\s]*/gi, "$1 $2");

  

  // Remove other Markdown links

  cleanText = cleanText.replace(/\[([A-Za-z0-9\s]+)\]\([^)]+\)/g, '$1');

  

  const part4Pos = cleanText.search(/Part\s*4\s*:?\s*Coaching\s*Action\s*Plan/i);

  const coachingText = part4Pos !== -1 ? cleanText.substring(part4Pos) : cleanText;

  let scoreText = part4Pos !== -1 ? cleanText.substring(0, part4Pos) : cleanText;

  

  // Clean out noisy numbers so they don't get misread as scores

  scoreText = scoreText.replace(/https?:\/\/\S+/gi, "")

                       .replace(/\b(?:page|pg\.?)\s*\d+\b/gi, "")

                       .replace(/\/\s*5\b/g, "")

                       .replace(/out\s*of\s*5/gi, "");

  

  let scores = {};

  let foundIds = [];

  

  // PASS 1: Locate every ID in the text and log its exact position

  RUBRIC_CRITERIA.forEach(item => {

    let idRegex = new RegExp("\\b" + item.id.replace(" ", "\\s*[-_]?\\s*") + "\\b", "gi");

    let matches = [...scoreText.matchAll(idRegex)];

    for (let m of matches) {

      foundIds.push({ id: item.id, index: m.index, matchString: m[0] });

    }

  });

  

  // Sort them so they are in reading order (top to bottom)

  foundIds.sort((a, b) => a.index - b.index);

  

  // PASS 2: Chop the text into blocks between each ID and extract the score from that specific block

  for (let i = 0; i < foundIds.length; i++) {

    let current = foundIds[i];

    let nextIndex = (i + 1 < foundIds.length) ? foundIds[i + 1].index : scoreText.length;

  

    let chunk = scoreText.substring(current.index, nextIndex);

    chunk = chunk.replace(current.matchString, ""); // Remove the ID itself

  

    let val = extractScoreFromChunk(chunk);

    if (val) {

      let safeVal = val.toUpperCase().replace('/', '');

      scores[current.id] = (safeVal === 'NA' || safeVal === '0') ? "" : parseInt(safeVal, 10);

    }

  }

  

  // PASS 3: Broadened Keyword Fallback Scan

  RUBRIC_CRITERIA.forEach(item => {

    if (scores[item.id] !== undefined) return;

  

    let kw = CRITERIA_KEYWORDS[item.id] || item.criteria.toLowerCase();

    let kwRegex = new RegExp(kw, "gi");

    let matches = [...scoreText.matchAll(kwRegex)];

  

    if (matches.length > 0) {

      let m = matches[0];

      let chunk = scoreText.substring(m.index, m.index + 150);

      let val = extractScoreFromChunk(chunk);

      if (val) {

        let safeVal = val.toUpperCase().replace('/', '');

        scores[item.id] = (safeVal === 'NA' || safeVal === '0') ? "" : parseInt(safeVal, 10);

      }

    }

  });

  

  // ── AGENT NAME EXTRACTION ──────────────

  let agentName = fallbackAgentName || "";

  if (!agentName) {

    const aM = cleanText.match(/Agent(?:\s*Name)?\s*:\s*([^\n\r(|]+)/i);

    if (aM) {

      const v = aM[1].trim();

      if (!v.toLowerCase().includes("information not provided") && v.length > 0) agentName = v;

    }

  }

  if (!agentName) {

    const sM = cleanText.match(/My name is\s+([A-Z][a-zA-Z]+)/);

    if (sM) agentName = sM[1].trim();

  }

  if (!agentName) agentName = "Unknown Agent";

  

  // AGENT NAME SANITIZER

  agentName = agentName.replace(/\s*\[source:\s*\d+\]/gi, "").trim();

  const lowerName = agentName.toLowerCase();

  if (lowerName.includes("carlos")) agentName = "Carlos Fernandes";

  else if (lowerName.includes("jesus")) agentName = "Jesus Guzman";

  else if (lowerName === "tom") agentName = "Adriel";

  else if (lowerName === "mitchell" || lowerName === "michel") agentName = "Michel Sandoval";

  else if (lowerName === "angela") agentName = "Mariz";

  else if (lowerName === "mark") agentName = "Marcos";

  

  // ── CALL ID EXTRACTOR ─────────────────────────

  let callName = "";

  const fileNameLine = cleanText.match(/File\s*Name\s*:[^\n\r]+/i);

  if (fileNameLine) {

    const allNumbers = fileNameLine[0].match(/\d{4,15}/g);

    if (allNumbers && allNumbers.length > 0) callName = allNumbers[allNumbers.length - 1];

  }

  

  if (!callName) {

    const rawExtensionMatch = cleanText.match(/_(\d{4,15})\.(?:txt|mp3|wav|m4a)/i);

    if (rawExtensionMatch) callName = rawExtensionMatch[1];

  }

  

  if (!callName) {

    const cM = cleanText.match(/(?:Customer|Caller|Call Name|Account Name)\s*:\s*([^\n\r|]+)/i);

    if (cM) {

      const v = cM[1].trim();

      if (!v.toLowerCase().includes("information not provided")) {

        callName = v;

      } else {

        const emailM = v.match(/[\w.-]+@[\w.-]+\.\w+/);

        const quoteM = v.match(/"([^"]+)"/);

        callName = emailM ? emailM[0] : quoteM ? `Caller (${quoteM[1]})` : "";

      }

    }

    if (!callName) {

      callName = "Call " + Utilities.formatDate(new Date(), Session.getScriptTimeZone(), "MM/dd HH:mm");

    }

  }

  

  // ── COACHING FEEDBACK EXTRACTION ──────────────

  const wins = [];

  const winsM = coachingText.match(/Top\s*3\s*Wins\s*:?([\s\S]*?)(?=Top\s*3\s*Areas\s*for\s*Improvement|$)/i);

  if (winsM) {

    winsM[1].split(/(?:^|\n)\s*\d+\.\s*/).map(s => s.trim()).filter(Boolean)

      .forEach((w, idx) => wins.push(`${idx + 1}. ${w.replace(/\s+/g, ' ')}`));

  }

  const condensedWins = wins.join('\n\n') || "N/A";

  

  const improvements = [];

  const impM = coachingText.match(/Top\s*3\s*Areas\s*for\s*Improvement\s*:?([\s\S]*?)$/i);

  if (impM) {

    impM[1].split(/(?:^|\n)\s*\d+\.\s*/).map(s => s.trim()).filter(Boolean)

      .forEach((block, idx) => {

        const blockLines = block.split('\n').map(l2 => l2.trim()).filter(Boolean);

        const title      = blockLines.length > 0 ? blockLines[0].replace(/^[-*]\s*/, '').trim() : "N/A";

        let workOn = "", script = "";

        const woM  = block.match(/What to work on:\s*([\s\S]*?)(?=(?:-?\s*Coaching Script Example:)|$)/i);

        if (woM) workOn = "\n• Work on: " + woM[1].trim().replace(/\s+/g, ' ');

        const scM2 = block.match(/Coaching Script Example:\s*([\s\S]*?)$/i);

        if (scM2) script = "\n• Script: " + scM2[1].trim().replace(/\s+/g, ' ');

        improvements.push(`${idx + 1}. ${title}${workOn}${script}`);

      });

  }

  const condensedImprovements = improvements.join('\n\n') || "N/A";

  

  return {

    scores:                scores,

    agentName:             agentName,

    callName:              callName,

    winsFormatted:         condensedWins,

    improvementsFormatted: condensedImprovements,

    dateString:            getWorkWeekRange(new Date())

  };

}

  

// ── WRITE SCORES TO PRIVATE AGENT GRID (VERTICAL BLOCKS) ────────────

  

function writeScoresToGrid(sheet, parsed) {

  const dateToBlock = buildDateBlockMap(sheet);

  

  for (const date in dateToBlock) {

    const block = dateToBlock[date];

    const lastCol = sheet.getLastColumn();

    if (lastCol >= 3 && block.labelRow) {

      const existingCalls = sheet.getRange(block.labelRow, 3, 1, Math.max(lastCol - 2, 1)).getValues()[0];

      const isDuplicate = existingCalls.some(val => String(val).trim() === String(parsed.callName).trim());

      if (isDuplicate) return false;

    }

  }

  

  const scoreValues = RUBRIC_CRITERIA.map(item => [

    parsed.scores[item.id] !== undefined ? parsed.scores[item.id] : ""

  ]);

  

  if (dateToBlock.hasOwnProperty(parsed.dateString)) {

    const block = dateToBlock[parsed.dateString];

  

    if (block.labelRow) {

      sheet.getRange(block.labelRow, block.nextCol)

        .setValue(parsed.callName)

        .setFontWeight('bold')

        .setHorizontalAlignment('center')

        .setWrap(true);

    }

  

    sheet.getRange(block.startRow, block.nextCol, RUBRIC_CRITERIA.length, 1)

      .setValues(scoreValues)

      .setHorizontalAlignment('center');

  

    sheet.setColumnWidth(block.nextCol, 120);

  

  } else {

    const lastRow = sheet.getLastRow();

    const headerRow = lastRow > 0 ? lastRow + 2 : 1;

    const labelRow = headerRow + 1;

    const dataStartRow = labelRow + 1;

  

    sheet.getRange(headerRow, 1, 1, 3).setFontWeight('bold');

    sheet.getRange(headerRow, 1).setValue(parsed.dateString).setFontSize(10);

  

    sheet.getRange(labelRow, 1).setValue('Criteria ID').setFontWeight('bold');

    sheet.getRange(labelRow, 2).setValue('Evaluation Point (Criteria)').setFontWeight('bold');

    sheet.getRange(labelRow, 3)

      .setValue(parsed.callName)

      .setFontWeight('bold')

      .setHorizontalAlignment('center')

      .setWrap(true);

  

    const rubricRows = RUBRIC_CRITERIA.map(item => [item.id, item.criteria]);

    sheet.getRange(dataStartRow, 1, RUBRIC_CRITERIA.length, 2)

      .setValues(rubricRows)

      .setVerticalAlignment('middle');

    sheet.getRange(dataStartRow, 1, RUBRIC_CRITERIA.length, 1).setFontWeight('bold');

  

    sheet.getRange(dataStartRow, 3, RUBRIC_CRITERIA.length, 1)

      .setValues(scoreValues)

      .setHorizontalAlignment('center');

  

    sheet.setColumnWidth(1, 80);

    sheet.setColumnWidth(2, 220);

    sheet.setColumnWidth(3, 120);

  }

  return true;

}

  

function writeCoachingFeedback(ss, parsed) {

  const sheet = ss.getSheetByName("Coaching Feedback") || initCoachingFeedbackSheet(ss);

  

  sheet.appendRow([

    parsed.dateString,

    parsed.agentName,

    parsed.callName,

    parsed.winsFormatted,

    parsed.improvementsFormatted

  ]);

  

  const lastRow = sheet.getLastRow();

  sheet.getRange(lastRow, 1, 1, 5).setWrap(true).setVerticalAlignment('top');

}

  

// ═════════════════════════════════════════════════════════════════════════════

// SHARED SHEET SYNC ENGINE (Perfect Mirror Copy - CROSS SHEET SAFE)

// ═════════════════════════════════════════════════════════════════════════════

  

function syncToSharedSheet(silent) {

  try {

    const privateSS = SpreadsheetApp.getActiveSpreadsheet();

    const sharedSS  = SpreadsheetApp.openById(SHARED_SS_ID);

  

    let agentsSynced = 0;

  

    privateSS.getSheets().forEach(privateSheet => {

      const privateName = privateSheet.getName();

      if (SYSTEM_TABS.includes(privateName)) return;

      if (privateSheet.getLastRow() === 0) return;

  

      const sharedTabName = resolveSharedTabName(privateName, sharedSS);

  

      let sharedSheet = sharedSS.getSheetByName(sharedTabName);

      if (!sharedSheet) {

        sharedSheet = sharedSS.insertSheet(sharedTabName);

      }

  

      const success = syncAgentScoreTab(privateSheet, sharedSheet);

      if (success > 0) agentsSynced++;

    });

  

    if (!silent) {

      SpreadsheetApp.getActiveSpreadsheet().toast(

        `Sync complete! ${agentsSynced} agent tab(s) perfectly mirrored to the dashboard.`,

        '🔄 Dashboard Synced', 5

      );

    }

  

  } catch (err) {

    SpreadsheetApp.getActiveSpreadsheet().toast(

      'Sync error: ' + err.message,

      '⚠️ Sync Failed', 8

    );

  }

}

  

function syncAgentScoreTab(privateSheet, sharedSheet) {

  const privateLastRow = privateSheet.getLastRow();

  const privateLastCol = privateSheet.getLastColumn();

  

  if (privateLastRow === 0 || privateLastCol === 0) return 0;

  

  // Wipe the target sheet clean

  sharedSheet.clear();

  

  const sourceRange = privateSheet.getRange(1, 1, privateLastRow, privateLastCol);

  const targetRange = sharedSheet.getRange(1, 1, privateLastRow, privateLastCol);

  

  // Manually copy the data and formatting so it successfully jumps across different files

  targetRange.setValues(sourceRange.getValues());

  targetRange.setFontWeights(sourceRange.getFontWeights());

  targetRange.setFontColors(sourceRange.getFontColors());

  targetRange.setHorizontalAlignments(sourceRange.getHorizontalAlignments());

  targetRange.setVerticalAlignments(sourceRange.getVerticalAlignments());

  targetRange.setWraps(sourceRange.getWraps());

  targetRange.setFontSizes(sourceRange.getFontSizes());

  targetRange.setBackgrounds(sourceRange.getBackgrounds());

  

  // Copy the column widths over

  for (let c = 1; c <= privateLastCol; c++) {

    sharedSheet.setColumnWidth(c, privateSheet.getColumnWidth(c));

  }

  

  return 1;

}

  

// ── UTILITY FUNCTIONS ────────────────────────────────────────────────────────

  

function buildDateBlockMap(sheet) {

  const map = {};

  const lastRow = sheet.getLastRow();

  if (lastRow < 2) return map;

  

  const colA = sheet.getRange(1, 1, lastRow, 1).getValues();

  const rubricPattern = /^(BG|ARC|OE|PE|QC|CL|CC|COMP)\s*\d+/i;

  

  for (let i = 0; i < colA.length; i++) {

    const val = String(colA[i][0]).trim();

    const cleanDate = val.replace(/^📅\s*/, '').trim();

  

    if (!cleanDate || !(cleanDate.includes('/') || cleanDate.includes('-'))) continue;

  

    let labelRow = null;

    let dataStartRow = null;

  

    if (i + 1 < colA.length && rubricPattern.test(String(colA[i + 1][0]).trim())) {

      dataStartRow = i + 2;

    } else if (i + 2 < colA.length && rubricPattern.test(String(colA[i + 2][0]).trim())) {

      labelRow = i + 2;      

      dataStartRow = i + 3;  

    }

  

    if (!dataStartRow) continue;

  

    const headerRow = i + 1;

    const scanRow = labelRow || dataStartRow;

    const lastCol = Math.max(sheet.getLastColumn(), 3);

    const rowVals = sheet.getRange(scanRow, 3, 1, Math.max(lastCol - 2, 1)).getValues()[0];

  

    let nextCol   = 3;

    let callCount = 0;

    for (let c = 0; c < rowVals.length; c++) {

      if (String(rowVals[c]).trim() !== "") {

        nextCol = c + 4;

        callCount = c + 1;

      }

    }

  

    map[cleanDate] = { headerRow, labelRow, startRow: dataStartRow, nextCol, callCount };

  }

  

  return map;

}

  

function resolveSharedTabName(agentName, sharedSS) {

  if (NAME_MAP.hasOwnProperty(agentName)) {

    return NAME_MAP[agentName];

  }

  

  if (sharedSS.getSheetByName(agentName)) {

    return agentName;

  }

  

  if (/^Miguel\s+R/i.test(agentName)) {

    return "Miguel R";

  }

  

  const firstName = agentName.trim().split(/\s+/)[0];

  if (firstName && sharedSS.getSheetByName(firstName)) {

    return firstName;

  }

  

  return agentName;

}

  

function createSyncTrigger() {

  ScriptApp.getProjectTriggers().forEach(trigger => {

    if (trigger.getHandlerFunction() === 'timedSync') {

      ScriptApp.deleteTrigger(trigger);

    }

  });

  

  ScriptApp.newTrigger('timedSync')

    .timeBased()

    .everyMinutes(5)

    .create();

  

  SpreadsheetApp.getActiveSpreadsheet().toast(

    'Auto-sync timer set! Syncs to shared dashboard every 5 minutes.',

    '⏰ Timer Created', 5

  );

}

  

function timedSync() {

  processAllRawData();

}

  

function initCoachingFeedbackSheet(ss) {

  ss = ss || SpreadsheetApp.getActiveSpreadsheet();

  const sheet   = ss.getSheetByName("Coaching Feedback") || ss.insertSheet("Coaching Feedback");

  const headers = ["Date", "Agent Name", "Call / Customer Name", "Top 3 Wins", "Top 3 Areas for Improvement (With Scripts)"];

  sheet.getRange(1, 1, 1, headers.length)

    .setValues([headers])

    .setFontWeight("bold")

    .setHorizontalAlignment("center");

  [100, 150, 160, 360, 450].forEach((w, i) => sheet.setColumnWidth(i + 1, w));

  return sheet;

}

  

function getOrCreateAgentSheet(ss, agentName) {

  if (!agentName || agentName === "Unknown Agent") return ss.getActiveSheet();

  const existing = ss.getSheetByName(agentName);

  if (existing) return existing;

  const newSheet = ss.insertSheet(agentName);

  return newSheet;

}

  

function archiveMonthAndReset() {

  const ui      = SpreadsheetApp.getUi();

  const confirm = ui.alert(

    'Monthly Reset',

    'This will clear ALL agent tabs entirely to prep for the new block layout.\n\nCoaching Feedback is NOT cleared. Continue?',

    ui.ButtonSet.YES_NO

  );

  if (confirm !== ui.Button.YES) return;

  

  SpreadsheetApp.getActiveSpreadsheet().getSheets().forEach(sheet => {

    if (SYSTEM_TABS.includes(sheet.getName())) return;

    sheet.clear();

  });

  

  SpreadsheetApp.getActiveSpreadsheet().toast('Agent tabs completely cleared! Ready for new blocks.', 'Done', 5);

}

  

function getWorkWeekRange(dateObj) {

  const tz = Session.getScriptTimeZone();

  const dateStr = Utilities.formatDate(dateObj, tz, "MM/dd/yyyy");

  const d = new Date(dateStr);

  const day = d.getDay();

  let diffToMonday = 0;

  if (day === 0) {

    diffToMonday = 1;

  } else if (day === 6) {

    diffToMonday = 2;

  } else {

    diffToMonday = 1 - day;

  }

  const monday = new Date(d.getTime());

  monday.setDate(d.getDate() + diffToMonday);

  const friday = new Date(monday.getTime());

  friday.setDate(monday.getDate() + 4);

  return (monday.getMonth() + 1) + "/" + monday.getDate() + "-" + (friday.getMonth() + 1) + "/" + friday.getDate();

}


### References
