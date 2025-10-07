**Step 1: Create or set up a spread sheet with the first row value aligned with your form element's name in your HTML"
eg: <input name="question1" value="yes"/> => This will be write to the "question1" column in the spreadsheet
=> This need to be exact: "question1"-"question1". ("question1"-"question 1" won't work)
- Note: The "Date" column will help you record the create_time of the record

**Step 2: Click extentions -> appscript -> Paste all of the code below, change the sheet name according to the sheet name in your spreadsheet and click run for the first time**

const sheetName = 'Sheet1' //Change this if you change your sheet name, this will allow you to deploy multiple data flow to one spreadsheet
const scriptProp = PropertiesService.getScriptProperties()

function intialSetup () {
  const activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet()
  scriptProp.setProperty('key', activeSpreadsheet.getId())
}

function doPost (e) {
  const lock = LockService.getScriptLock()
  lock.tryLock(10000)

  try {
    const doc = SpreadsheetApp.openById(scriptProp.getProperty('key'))
    const sheet = doc.getSheetByName(sheetName)

    const headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0]
    const nextRow = sheet.getLastRow() + 1
    
    const newRow = headers.map(function(header) {
      if (header === 'Date') {
        return new Date().toLocaleString()
      } else if (e.parameter.hasOwnProperty(header)) {
        return e.parameter[header]
      } else {
        return '' // leave blank if header not found in request
      }
    })

    sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow])

    return ContentService
      .createTextOutput(JSON.stringify({ result: 'success', row: nextRow }))
      .setMimeType(ContentService.MimeType.JSON)
  }

  catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ result: 'error', error: err.message }))
      .setMimeType(ContentService.MimeType.JSON)
  }

  finally {
    lock.releaseLock()
  }
}

**Step 3: Click deploy -> Select type: Web app -> Change "who has access" to "anyone" -> deploy (Google will require you to grant access on your first time using this)**

**Step 4: Copy the Web app URL and replace to the URL in <Script> tag in index7.html**

**=> FINISH. You can try submit your form using local server or put it on github pages to test it**
