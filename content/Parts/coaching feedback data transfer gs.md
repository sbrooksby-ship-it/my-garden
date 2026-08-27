2026-08-27 11:44

Status:

Tags: [[Google Script]]


# coaching feedback data transfer gs



// =========================================================================

// ONE-TIME SETUP: Run this function once to build the 1-hour trigger!

// =========================================================================

function setupTrigger() {

  const triggers = ScriptApp.getProjectTriggers();

  for (let i = 0; i < triggers.length; i++) {

    if (triggers[i].getHandlerFunction() === "generateAgentDailySummaries") {

      console.log("Trigger already exists! No action needed.");

      return;

    }

  }

  // Changed to run every 1 hour

  ScriptApp.newTrigger("generateAgentDailySummaries")

    .timeBased()

    .everyHours(1)

    .create();

  console.log("SUCCESS: 1-hour trigger has been automatically created!");

}

  

// =========================================================================

// MAIN FUNCTION: Runs every 1 hour via trigger

// =========================================================================

function generateAgentDailySummaries() {

  // 1. Setup API Key, Model, and Shared Sheet ID

  const API_KEY = "AQ.Ab8RN6IEwIz557wmnZ35M5ZPhFC-WcYQPI7nVx8tB28sfzIVtw";

  const MODEL = "gemini-3.5-flash-lite";

  const SHARED_SS_ID = "1-N0IJxjzrdM_mlmIn9QYMHj_PPMfOopP7CReYW5m5IQ";

  // 2. Connect to your Private Sheets

  const ss = SpreadsheetApp.getActiveSpreadsheet();

  const sourceSheet = ss.getSheetByName("Coaching Feedback"); // Unorganized raw feedback

  const targetSheet = ss.getSheetByName("Summary CF");       // Private Agent trend reports

  // 3. Connect to the Shared Sheet

  let sharedSummarySheet = null;

  try {

    const sharedSS = SpreadsheetApp.openById(SHARED_SS_ID);

    sharedSummarySheet = sharedSS.getSheetByName("Coaching Feedback");

  } catch (e) {

    console.error("Could not connect to Shared Spreadsheet. Check ID permissions.");

  }

  

  // 4. Grab all raw data

  const rawData = sourceSheet.getDataRange().getValues();

  if (rawData.length <= 1) {

    console.log("No data to process.");

    return;

  }

  const headers = rawData[0];

  const agentColIndex = headers.findIndex(h => h.toString().toLowerCase().includes("agent"));

  const dateColIndex  = headers.findIndex(h => h.toString().toLowerCase().includes("date"));

  const agentIdx = agentColIndex !== -1 ? agentColIndex : 1;

  const dateIdx  = dateColIndex  !== -1 ? dateColIndex  : 0;

  

  // 5. Load existing summaries from "Summary CF" to prevent duplicate runs

  const existingSummaries = targetSheet.getDataRange().getValues();

  const processedKeys = new Set();

  // Start scanning at i=0 so we don't miss Row 1

  for (let i = 0; i < existingSummaries.length; i++) {

    let agentVal = existingSummaries[i][0];

    if (!agentVal || agentVal.toString().toLowerCase().includes("agent")) continue;

    let existingAgent = agentVal.toString().trim().toLowerCase();

    let existingDateRange = getWorkWeekRange(existingSummaries[i][1]);            

    processedKeys.add(`${existingAgent}_${existingDateRange}`);

  }

  

  // 6. Group raw rows by Agent + Date Range

  const groupedData = {};

  for (let i = 1; i < rawData.length; i++) {

    let row = rawData[i];

    let agent = row[agentIdx] ? row[agentIdx].toString().trim() : "Unknown Agent";

    let dateRangeStr = getWorkWeekRange(row[dateIdx]);

    let groupKey = `${agent.toLowerCase()}_${dateRangeStr}`;

    if (processedKeys.has(groupKey)) continue;

    if (!groupedData[groupKey]) {

      groupedData[groupKey] = {

        agentName: agent,

        dateRange: dateRangeStr,

        rows: []

      };

    }

    groupedData[groupKey].rows.push(row);

  }

  

  const keysToProcess = Object.keys(groupedData);

  if (keysToProcess.length === 0) {

    console.log("No new Agent/Date Range combinations to process.");

    return;

  }

  

  const MAX_PER_RUN = 8;

  let processedCount = 0;

  

  // 7. Process each Agent + Date Range group through Gemini

  for (let key of keysToProcess) {

    if (processedCount >= MAX_PER_RUN) {

      console.log("Reached maximum batch limit for this run. Remaining items will process next cycle.");

      break;

    }

  

    let group = groupedData[key];

    let groupJson = JSON.stringify(group.rows);

    let totalCalls = group.rows.length;

  

    // 8. The Upgraded Prompt Structure

    let prompt = `

      You are an expert Quality Assurance Analyst and Call Center Coach.

      Review the following call feedback entries for Agent: "${group.agentName}" for the week of: "${group.dateRange}".

      TOTAL CALLS IN THIS BATCH: ${totalCalls}

  

      Synthesize these calls and extract the overarching trends.

      You MUST output your response as a valid JSON object with EXACTLY these two keys:

      "top3Wins": An array of strings representing the top 3 strengths.

      "top3Improvements": An array of strings representing the top 3 areas for improvement.

  

      CRITICAL FORMATTING FOR "top3Improvements":

      Each of the 3 improvements in the array MUST strictly follow this exact layout and phrasing:

      [Number]. [Skill/Category Name] ([Rubric ID like OE 1 or CL 2]) (Noticed in [X] out of ${totalCalls} calls)

      - What to work on: [Provide a highly detailed explanation of what the agent missed based on the scoring criteria. Explain why it matters.]

      - Coaching Script Example: "[Provide a direct, actionable, quoted script the agent can use next time]"

  

      Example of a perfect improvement entry:

      "1. Purposeful Discovery (OE 1 / PE 2) (Noticed in 4 out of ${totalCalls} calls)\n- What to work on: The agent is acting as a passive order-taker. Per the criteria, a baseline score of 3 requires asking an initial question about the current state of the account. Currently, the agent processes requests immediately without offering alternative pathways.\n- Coaching Script Example: \"I'd be happy to help you adjust your shipments. Just to make sure we find the best solution, are you taking a break from the routine?\""

  

      Call Feedback Entries to Analyze:

      ${groupJson}

    `;

  

    let payload = {

      "contents": [{

        "parts": [{"text": prompt}]

      }],

      "generationConfig": {

        "responseMimeType": "application/json"

      }

    };

  

    let options = {

      "method": "post",

      "contentType": "application/json",

      "payload": JSON.stringify(payload),

      "muteHttpExceptions": true

    };

  

    let url = `https://generativelanguage.googleapis.com/v1beta/models/${MODEL}:generateContent?key=${API_KEY}`;

    try {

      let response = UrlFetchApp.fetch(url, options);

      let json = JSON.parse(response.getContentText());

  

      if (json.error) {

        console.error(`API Error for ${group.agentName} (${group.dateRange}): `, json.error.message);

        continue;

      }

  

      let rawText = json.candidates[0].content.parts[0].text;

      let aiData = JSON.parse(rawText);

      let wins = Array.isArray(aiData.top3Wins) ? aiData.top3Wins.join('\n\n') : (aiData.top3Wins || "N/A");

      let improvements = Array.isArray(aiData.top3Improvements) ? aiData.top3Improvements.join('\n\n') : (aiData.top3Improvements || "N/A");

  

      wins = String(wins).trim();

      improvements = String(improvements).trim();

  

      // --- APPEND TO PRIVATE SHEET ---

      targetSheet.appendRow([group.agentName, group.dateRange, wins, improvements]);

      // --- APPEND TO SHARED SHEET ---

      if (sharedSummarySheet) {

        sharedSummarySheet.appendRow([group.agentName, group.dateRange, wins, improvements]);

      }

      processedCount++;

  

      if (processedCount < keysToProcess.length && processedCount < MAX_PER_RUN) {

        Utilities.sleep(30000);

      }

  

    } catch (e) {

      console.error(`Execution error for ${group.agentName}: `, e);

    }

  }

}

  

// ── DATE CALCULATOR HELPER ───────────────────────────────────────────────────

function getWorkWeekRange(val) {

  if (!val) return "Unknown Date Range";

  if (typeof val === 'string' && val.includes('-')) {

    return val.trim();

  }

  let dateObj = val;

  if (!(val instanceof Date)) {

    dateObj = new Date(val);

  }

  if (isNaN(dateObj.getTime())) return val.toString().trim();

  

  const tz = Session.getScriptTimeZone();

  const dateStr = Utilities.formatDate(dateObj, tz, "MM/dd/yyyy");

  const d = new Date(dateStr);

  const day = d.getDay();

  let diffToMonday = 0;

  if (day === 0) diffToMonday = 1;

  else if (day === 6) diffToMonday = 2;

  else diffToMonday = 1 - day;

  const monday = new Date(d.getTime());

  monday.setDate(d.getDate() + diffToMonday);

  const friday = new Date(monday.getTime());

  friday.setDate(monday.getDate() + 4);

  return (monday.getMonth() + 1) + "/" + monday.getDate() + "-" + (friday.getMonth() + 1) + "/" + friday.getDate();

}


### References
