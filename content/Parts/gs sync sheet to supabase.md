2026-08-27 11:42

Status:

Tags: [[Google Script]]


# gs sync sheet to supabase




/**

 * Syncs the Raw Data and Coaching Feedback tabs to Supabase.

 * Your existing Sheets stay exactly as they are — this just tails new rows

 * in each tab and pushes them onward.

 */

  

// Each entry: which sheet tab, which Supabase table, how columns map,

// and which columns form the "don't duplicate this" key (on_conflict).

const SYNC_CONFIGS = [

  {

    sheetName: 'Raw Data',

    tableName: 'call_scores',

    propertyKey: 'LAST_SYNCED_ROW_RAWDATA',

    onConflict: 'date_range,agent_name,call_id,category',

    columnMap: {

      date_range: 1,   // column A

      agent_name: 2,   // column B

      call_id: 3,      // column C

      category: 4,     // column D

      criteria: 5,     // column E

      score: 6,        // column F

    },

  },

  {

    sheetName: 'Coaching Feedback',

    tableName: 'coaching_feedback',

    propertyKey: 'LAST_SYNCED_ROW_COACHING',

    onConflict: 'agent_name,date_range',

    columnMap: {

      agent_name: 1,        // column A

      date_range: 2,        // column B

      top_wins: 3,          // column C

      top_improvements: 4,  // column D

    },

  },

];

  

function syncAll() {

  for (const config of SYNC_CONFIGS) {

    syncSheetTab(config);

  }

}

  

function syncSheetTab(config) {

  const props = PropertiesService.getScriptProperties();

  const supabaseUrl = props.getProperty('SUPABASE_URL');

  const serviceKey = props.getProperty('SUPABASE_SERVICE_KEY');

  // Safely parse memory. If it's broken, corrupted, or 0, FORCE it to 1 so we skip headers.

  let rawProp = props.getProperty(config.propertyKey);

  let lastSyncedRow = parseInt(rawProp, 10);

  if (isNaN(lastSyncedRow) || lastSyncedRow < 1) {

      lastSyncedRow = 1;

  }

  

  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(config.sheetName);

  if (!sheet) {

    Logger.log(`Sheet tab "${config.sheetName}" not found — skipping.`);

    return;

  }

  

  const lastRow = sheet.getLastRow();

  // If the user wipes the sheet using the "Archive & Reset" button, start over!

  if (lastRow < lastSyncedRow) {

      lastSyncedRow = 1;

  }

  if (lastRow <= lastSyncedRow) {

    Logger.log(`${config.sheetName}: no new rows.`);

    return;

  }

  

  const numRows = lastRow - lastSyncedRow;

  const range = sheet.getRange(lastSyncedRow + 1, 1, numRows, sheet.getLastColumn());

  const values = range.getValues();

  

  const records = values

    .map((row) => {

      const record = {};

      for (const [col, idx] of Object.entries(config.columnMap)) {

        let val = row[idx - 1];

        if (val instanceof Date) val = val.toISOString();

        // --- BULLETPROOF DATA TYPING ---

        // If this is the score column, make absolutely sure it is a real number

        if (col === 'score') {

            if (val === '' || val === null || isNaN(Number(val))) {

                val = null; // Convert text/blanks to a safe database null

            } else {

                val = Number(val);

            }

        } else {

            val = val === '' ? null : String(val).trim();

        }

        record[col] = val;

      }

      return record;

    })

    // Skip completely blank rows AND ensure we never accidentally pass header text

    .filter((r) => r.agent_name && r.agent_name !== 'Agent Name');

  

  if (records.length === 0) {

    props.setProperty(config.propertyKey, String(lastRow));

    return;

  }

  

  const conflictCols = config.onConflict.split(',');

  const deduped = new Map();

  const genericTracker = {}; // Tracks generic calls to split them safely

  

  for (const record of records) {

    // --- SMART SPLIT LOGIC FOR RAW DATA ---

    if (config.sheetName === 'Raw Data') {

      const callStr = String(record.call_id || '').trim();

      const isGeneric = /^(call|unknown)/i.test(callStr);

      if (isGeneric) {

        // Track how many times we've seen this exact category for this exact generic call name

        const trackKey = `${record.agent_name}||${record.date_range}||${callStr}||${record.category}`;

        if (!genericTracker[trackKey]) {

            genericTracker[trackKey] = 0;

        }

        genericTracker[trackKey]++;

        if (genericTracker[trackKey] > 1) {

            // Modify the call_id before Supabase or the Deduper sees it!

            record.call_id = `${callStr} (Split ${genericTracker[trackKey]})`;

        }

      }

    }

    // --------------------------------------

  

    const key = conflictCols.map((c) => record[c]).join('||');

    deduped.set(key, record);

  }

  const dedupedRecords = Array.from(deduped.values());

  if (dedupedRecords.length < records.length) {

    Logger.log(`${config.sheetName}: dropped ${records.length - dedupedRecords.length} in-batch duplicate(s).`);

  }

  

  const url = `${supabaseUrl}/rest/v1/${config.tableName}?on_conflict=${config.onConflict}`;

  const response = UrlFetchApp.fetch(url, {

    method: 'post',

    contentType: 'application/json',

    headers: {

      apikey: serviceKey,

      Authorization: `Bearer ${serviceKey}`,

      Prefer: 'resolution=merge-duplicates',

      'User-Agent': 'GoogleAppsScript-CallGradingSync/1.0',

    },

    payload: JSON.stringify(dedupedRecords),

    muteHttpExceptions: true,

  });

  

  if (response.getResponseCode() >= 300) {

    Logger.log(`${config.sheetName} sync failed: ${response.getContentText()}`);

    return; // don't advance the marker — retry next run

  }

  

  props.setProperty(config.propertyKey, String(lastRow));

  Logger.log(`${config.sheetName}: synced ${dedupedRecords.length} row(s).`);

}

### References
