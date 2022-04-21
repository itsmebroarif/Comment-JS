# Comment-JS

#### Library Js Untuk membuat input data menuju google sheets.

---

## 1. Create a new Google Sheet

- Pertama, Buka [Google Sheets](https://docs.google.com/spreadsheets) Dan `Start a new spreadsheet` Dengan `Blank` template.
- Beri Nama `Email Subscribers`. Atau Apapun, Itu Tidak masalah.
- Buat Tabel Seperti Dibawah ini :

  | | A | B | C | D |
  | --- | :--: | :---: | :-: | :-: |
  | 1 | date | nama | email |pesan |

> To learn how to add additional input fields, [checkout section 7 below](#7-adding-additional-form-data).

## 2. Buat Sebuat Google Apps Script

- Klik `Extension > Apps Script`
- Ubah Namanya Menjadi `Submit Form to Google Sheets`. Setelah itu Save.
- Sekarang, delete `function myFunction() {}` Dan ganti dengan script yang ada dibawah ini

---

# code.gs

```js
var sheetName = "Sheet1";
var scriptProp = PropertiesService.getScriptProperties();
function intialSetup() {
  var activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  scriptProp.setProperty("key", activeSpreadsheet.getId());
}
function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(10000);
  try {
    var doc = SpreadsheetApp.openById(scriptProp.getProperty("key"));
    var sheet = doc.getSheetByName(sheetName);
    var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
    var nextRow = sheet.getLastRow() + 1;
    var newRow = headers.map(function (header) {
      return header === "date" ? new Date() : e.parameter[header];
    });
    sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow]);
    return ContentService.createTextOutput(
      JSON.stringify({ result: "success", row: nextRow })
    ).setMimeType(ContentService.MimeType.JSON);
  } catch (e) {
    return ContentService.createTextOutput(
      JSON.stringify({ result: "error", error: e })
    ).setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}
```

## 3. Menjalankan setup function

- Setelah itu, Pergi ke `Run > Run Function > initialSetup` Untuk Menjalankan Code yang sudah kalian buat.
- Nanti akan ada `Authorization Required` dialog, Buka `Review Permissions`.
- Masuk / Ambil Google Account Untuk Menyimpan data program ini agar dapat dijalankan.
- Kamu pasti bakal ngeliat ini , `Hi {Your Name}`, `Submit Form to Google Sheets wants to`...
- Klik `Allow` saja, Ini google meminta perizinan untuk Fetching Data

## 4. Mari Buat Trigger Baru

- Klik`Edit > Current project’s triggers`. / Logo Jam Di Sebelah Kiri
- Terus `No triggers set up. Click here to add one now.`
- Pas dibagian (Choose which function to run) Ubah `InitialSetup` jadi `doPost`
- bikin events Fields dari `From spreadsheet` dan `On form submit`
- Setelah itu, Simpan Datanya `Save`

## 5. Aplikasi nya udah selesai, mari di publish

- Klik `Publish `dan jadikan sebagai web app dengan cara` Deploy as web app…`.
- Set `Project Version` ke `New` dan taruh `initial version` in the input field below.
- `Execute the app as:` arahkan ke email kalian / `Me(your@address.com)`.
- untuk akses `Who has access to the app:` pilih `Anyone, even anonymous`.
- Abis itu langsung `Deploy`.
- In the popup, salin data web app urlnya `Current web app URL` dari dialog.
- And click `OK`, Selesai dari situ pindah link nya ke const nya js

## 6. Testing Aplikasi (Via Javascript)

#### 6.1 Testing Dengan Menggunakan Js untuk dijadikan webapps

```js
<form name="namaForm">
  <input name="email" type="email" placeholder="Email" required>
  <button type="submit">Send</button>
</form>

<script>
  const scriptURL = '<SCRIPT Web App Yang Tadi>'
  const form = document.forms['namaForm']

  form.addEventListener('submit', e => {
    e.preventDefault()
    fetch(scriptURL, { method: 'POST', body: new FormData(form)})
      .then(response => console.log('Success!', response))
      .catch(error => console.error('Error!', error.message))
  })
</script>
```

#### 6.2 Testing Menggunakan Api testing

- Buka Aplikasi Postman Kalian
- Set Key nya menjadi post
- Tempel link yang sudah kalian dapatkan dari google apps script(https://script.google.com/a/yourdomain.com/macros/s/XXXX…)
- Masukan Key Yang Kalian buat seperti contoh kalian memiliki form lain selain email ['nama','email','pesan']
  | | A | B | C | D |
  | --- | :--: | :---: | :-: | :-: |
  | 1 | date | nama | email |pesan |
