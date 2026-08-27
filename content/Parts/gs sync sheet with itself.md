2026-08-27 11:40

Status:

Tags: [[Google Script]]


# gs sync sheet with itself


/**

 * ============================================================================

 * GOOGLE APPS SCRIPT: AGENT TAB -> RAW DATA SYNC (for Streamlit)

 * ============================================================================

 *

 * WHAT IT DOES:

 *  Reads every agent tab (Adriel, Amon, Arturo, ...) on this spreadsheet,

 *  finds each date block / call score, and flattens it all into one row

 *  per (date, agent, call, criteria, score) on the "Raw Data" tab — which

 *  is presumably what your Streamlit app reads from.

 *

 *  Runs on a 5-minute timer automatically, or on demand via the menu.

 *

 * SETUP:

 *  Open this spreadsheet once (onOpen builds the menu), then from the

 *  "🔄 Sync Dashboard" menu click "⏰ Setup Auto-Sync Timer (5 min)" once

 *  to turn on the automatic timer.

 * ============================================================================

 */

  

function onOpen() {

  const ui = SpreadsheetApp.getUi();

  ui.createMenu('🔄 Sync Dashboard')

      .addItem('Sync Agent Data Now', 'syncAgentTabsToRawData')

      .addSeparator()

      .addItem('⏰ Setup Auto-Sync Timer (5 min)', 'createSyncTrigger')

      .addItem('⏹ Remove Auto-Sync Timer', 'removeSyncTrigger')

      .addToUi();

}

  

function syncAgentTabsToRawData() {

  const ss = SpreadsheetApp.getActiveSpreadsheet();

  const rawDataSheet = ss.getSheetByName('Raw Data');

  const agentTabs = ['Adriel', 'Amon', 'Arturo', 'Carlos', 'Daniel', 'Denia', 'Imelda Cardona', 'Mariz', 'Miguel', 'Miguel R', 'Nestor', 'Tyrone','Michael Nkanta','Alondra Monsivais','Thelma','Danilyn Suaybaguio','Francisco Bautista','Sergio Garibay','Jhonathan Gonzalez','Rose Ferdinand','Marcos Vazquez','Karl Leon','Raul Canales','Valeria Gonzalez','Roman Nunez','Jesus Guzman','Michel Sandoval'];

  

  let dataToAppend = [];

  

  // Loop through all agent tabs

  agentTabs.forEach(agent => {

    const sheet = ss.getSheetByName(agent);

    if (!sheet) return;

  

    const data = sheet.getDataRange().getValues();

    let currentDate = "No Date Provided";

    let currentHeaders = [];

  

    // Scan down the rows one by one

    for (let r = 0; r < data.length; r++) {

      const row = data[r];

      const colA = row[0];

      const colB = row[1];

  

      if (!colA && !colB) continue;

  

      // DETECT DATE ROW: Column A has text, Column B is blank

      if (colA && !colB) {

        currentDate = colA;

        continue;

      }

  

      // DETECT HEADER ROW

      if (colA === "Criteria ID" || colA === "ID" || colB === "Evaluation Point (Criteria)") {

        currentHeaders = row;

        continue;

      }

  

      // DETECT DATA ROW

      if (colA && colB && currentHeaders.length > 0) {

        for (let c = 2; c < currentHeaders.length; c++) {

          const callName = currentHeaders[c];

          const score = row[c];

  

          if (callName && score !== "") {

            dataToAppend.push([currentDate, agent, callName, colA, colB, score]);

          }

        }

      }

    }

  });

  

  if (dataToAppend.length > 0) {

    // Clear the existing data and paste the fresh data

    const lastRow = rawDataSheet.getLastRow();

    if (lastRow > 1) {

      rawDataSheet.getRange(2, 1, lastRow - 1, 6).clearContent();

    }

    rawDataSheet.getRange(2, 1, dataToAppend.length, 6).setValues(dataToAppend);

    ss.toast('All agent data has been successfully synced for Streamlit.', '✅ Success', 5);

  } else {

    ss.toast('No data found to sync.', 'ℹ️ Sync Dashboard', 5);

  }

}

  

// ── TRIGGER SETUP ──────────────────────────────────────────────────────────

  

function createSyncTrigger() {

  removeSyncTrigger(); // avoid duplicates if run more than once

  

  ScriptApp.newTrigger('syncAgentTabsToRawData')

    .timeBased()

    .everyMinutes(5)

    .create();

  

  SpreadsheetApp.getActiveSpreadsheet().toast(

    'Auto-sync timer set! Agent data will sync to Raw Data every 5 minutes.',

    '⏰ Timer Created', 5

  );

}

  

function removeSyncTrigger() {

  ScriptApp.getProjectTriggers().forEach(trigger => {

    if (trigger.getHandlerFunction() === 'syncAgentTabsToRawData') {

      ScriptApp.deleteTrigger(trigger);

    }

  });

}


### References
