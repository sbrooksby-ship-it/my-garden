

2026-08-27 11:14

Status:

Tags: [[Github]] [[Google Script]]


# spreadsheet hoper gs adverse



var SYNC_COL_COUNT = 12; // columns A–L get synced from private → shared; anything past L is left alone

  

function onChangeSync(e) {

  // ── NEW LOCK SERVICE (The "Bouncer") ──

  var lock = LockService.getScriptLock();

  try {

    // Force parallel triggers to wait in line for up to 30 seconds

    lock.waitLock(30000);

  } catch (e) {

    console.error('Could not obtain lock. Another instance is running.');

    return;

  }

  

  try {

    var privateSS = SpreadsheetApp.getActiveSpreadsheet();

    var sourceSheet = privateSS.getSheetByName('Sheet1');

  

    var sharedSS = SpreadsheetApp.openById('1DJR0CqWI8P-68VZv-9QlL7e3_4EW-HWCKx3tsOcd1Y0');

    var targetSheet = sharedSS.getSheetByName('Downloads');

  

    var sourceLastRow = sourceSheet.getLastRow();

    if (sourceLastRow < 1) return;

  

    var sourceData = sourceSheet.getRange(1, 1, sourceLastRow, SYNC_COL_COUNT).getValues();

  

    // build a map of key (column A value) -> row number in the target sheet

    var targetLastRow = targetSheet.getLastRow();

    var targetKeyToRow = {};

    if (targetLastRow > 0) {

      var targetKeys = targetSheet.getRange(1, 1, targetLastRow, 1).getValues();

      for (var t = 0; t < targetKeys.length; t++) {

        var key = String(targetKeys[t][0]).trim();

        if (key !== '') targetKeyToRow[key] = t + 1; // 1-indexed row number

      }

    }

  

    var rowsToAppend = [];

    var processedKeysInBatch = {};

  

    for (var i = 0; i < sourceData.length; i++) {

      var rowData = sourceData[i];

      var key = String(rowData[0]).trim();

      if (key === '') continue; // skip blank rows

  

      // Skips duplicates within the exact same batch

      if (processedKeysInBatch[key]) {

        continue;

      }

      processedKeysInBatch[key] = true;

  

      if (targetKeyToRow.hasOwnProperty(key)) {

        // existing row — update only columns A:L

        var targetRowNum = targetKeyToRow[key];

        targetSheet.getRange(targetRowNum, 1, 1, SYNC_COL_COUNT).setValues([rowData]);

      } else {

        // new row — queue it to append

        rowsToAppend.push(rowData);

      }

    }

  

    if (rowsToAppend.length > 0) {

      var startRow = targetSheet.getLastRow() + 1;

      targetSheet.getRange(startRow, 1, rowsToAppend.length, SYNC_COL_COUNT).setValues(rowsToAppend);

    }

  

    hideNoneRows(targetSheet);

  } finally {

    // ── ALWAYS RELEASE THE LOCK ──

    // This lets the next script in line start working

    lock.releaseLock();

  }

}

  

function hideNoneRows(sheet) {

  var lastRow = sheet.getLastRow();

  if (lastRow < 1) return;

  

  var hValues = sheet.getRange('H1:H' + lastRow).getValues();

  

  for (var i = 0; i < hValues.length; i++) {

    var cellValue = String(hValues[i][0]).trim().toLowerCase();

    var rowNum = i + 1;

    if (cellValue === 'none') {

      sheet.hideRows(rowNum);

    } else {

      sheet.showRows(rowNum);

    }

  }

}

  

function createTrigger() {

  ScriptApp.newTrigger('onChangeSync')

    .forSpreadsheet(SpreadsheetApp.getActive())

    .onChange()

    .create();

}


### References
