# excelize-wasm

<p align="center"><img width="500" src="https://github.com/xuri/excelize-wasm/raw/main/excelize-wasm.svg" alt="excelize-wasm logo"></p>

<p align="center">
    <a href="https://www.npmjs.com/package/excelize-wasm"><img src="https://img.shields.io/npm/v/excelize-wasm.svg" alt="NPM version"></a>
    <a href="https://github.com/xuri/excelize-wasm/actions/workflows/go.yml"><img src="https://github.com/xuri/excelize-wasm/actions/workflows/go.yml/badge.svg" alt="Build Status"></a>
    <a href="https://codecov.io/gh/xuri/excelize-wasm"><img src="https://codecov.io/gh/xuri/excelize-wasm/branch/main/graph/badge.svg" alt="Code Coverage"></a>
    <a href="https://goreportcard.com/report/github.com/xuri/excelize-wasm/cmd"><img src="https://goreportcard.com/badge/github.com/xuri/excelize-wasm/cmd" alt="Go Report Card"></a>
    <a href="https://pkg.go.dev/github.com/xuri/excelize/v2"><img src="https://img.shields.io/badge/go.dev-reference-007d9c?logo=go&logoColor=white" alt="go.dev"></a>
    <a href="https://opensource.org/licenses/BSD-3-Clause"><img src="https://img.shields.io/badge/license-bsd-orange.svg" alt="Licenses"></a>
</p>

Excelize-wasm is a pure WebAssembly / Javascript port of Go [Excelize](https://github.com/xuri/excelize) library that allow you to write to and read from XLAM / XLSM / XLSX / XLTM / XLTX files. Supports reading and writing spreadsheet documents generated by Microsoft Excel&trade; 2007 and later. Supports complex components by high compatibility. The full API docs can be found at [docs reference](https://xuri.me/excelize/).

Excelize library forked from [excelize-wasm](https://github.com/xuri/excelize-wasm)

## Environment Compatibility

Browser | Version
---|---
Chrome | &ge;57
Chrome for Android and Android Browser | &ge;105
Edge | &ge;16
Safari on macOS and iOS | &ge;11
Firefox | &ge;52
Firefox for Android | &ge;104
Opera | &ge;44
Opera Mobile | &ge;64
Samsung Internet | &ge;7.2
UC Browser for Android | &ge;13.4
QQ Browser | &ge;10.4
Node.js | &ge;8.0.0
Deno | &ge;1.0

## Basic Usage

### Installation

#### Node.js

```bash
npm install --save excelize-wasm
```

#### Browser

```html
<script src="excelize-wasm/index.js"></script>
```

### Create spreadsheet

Here is a minimal example usage that will create spreadsheet file.

```javascript
const { init } = require('excelize-wasm');
const fs = require('fs');

init('./node_modules/excelize-wasm/excelize.wasm.gz').then((excelize) => {
  const f = excelize.NewFile();
  // Create a new sheet.
  const { index } = f.NewSheet('Sheet2');
  // Set value of a cell.
  f.SetCellValue('Sheet2', 'A2', 'Hello world.');
  f.SetCellValue('Sheet1', 'B2', 100);
  // Set active sheet of the workbook.
  f.SetActiveSheet(index);
  // Save spreadsheet by the given path.
  const { buffer, error } = f.WriteToBuffer();
  if (error) {
    console.log(error);
    return;
  }
  fs.writeFile('Book1.xlsx', buffer, 'binary', (error) => {
    if (error) {
      console.log(error);
    }
  });
});
```

Create spreadsheet in browser:

<details>
  <summary>View code</summary>

```html
<html>
<head>
  <meta charset="utf-8">
  <script src="https://<your_hostname>/excelize-wasm/index.js"></script>
</head>
<body>
  <div>
    <button onclick="download()">Download</button>
  </div>
  <script>
  function download() {
    excelizeWASM
      .init('https://<your_hostname>/excelize-wasm/excelize.wasm.gz')
      .then((excelize) => {
        const f = excelize.NewFile();
        // Create a new sheet.
        const { index } = f.NewSheet('Sheet2');
        // Set value of a cell.
        f.SetCellValue('Sheet2', 'A2', 'Hello world.');
        f.SetCellValue('Sheet1', 'B2', 100);
        // Set active sheet of the workbook.
        f.SetActiveSheet(index);
        // Save spreadsheet by the given path.
        const { buffer, error } = f.WriteToBuffer();
        if (error) {
          console.log(error);
          return;
        }
        const link = document.createElement('a');
        link.download = 'Book1.xlsx';
        link.href = URL.createObjectURL(
          new Blob([buffer], {
            type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
          })
        );
        link.click();
      });
  }
  </script>
</body>
```

</details>

### Reading spreadsheet

The following constitutes the bare to read a spreadsheet document.

```javascript
const { init } = require('excelize-wasm');
const fs = require('fs');

init('./node_modules/excelize-wasm/excelize.wasm.gz').then((excelize) => {
  const f = excelize.OpenReader(fs.readFileSync('Book1.xlsx'));
  // Set value of a cell.
  const ret1 = f.GetCellValue('Sheet1', 'B2');
  if (ret1.error) {
    console.log(ret1.error);
    return;
  }
  console.log(ret1.value);
  // Get all the rows in the Sheet1.
  const ret2 = f.GetRows('Sheet1');
  if (ret2.error) {
    console.log(ret2.error);
    return;
  }
  ret2.result.forEach((row) => {
    row.forEach((colCell) => {
      process.stdout.write(`${colCell}\t`);
    });
    console.log();
  });
});
```

### Add chart to spreadsheet file

With excelize-wasm chart generation and management is as easy as a few lines of code. You can build charts based on data in your worksheet or generate charts without any data in your worksheet at all.

<p align="center"><img width="650" src="https://raw.githubusercontent.com/xuri/excelize-wasm/main/chart.png" alt="Create chart by excelize-wasm"></p>

```javascript
const { init } = require('excelize-wasm');
const fs = require('fs');

init('./node_modules/excelize-wasm/excelize.wasm.gz').then((excelize) => {
  const f = excelize.NewFile();
  [
    [null, 'Apple', 'Orange', 'Pear'],
    ['Small', 2, 3, 3],
    ['Normal', 5, 2, 4],
    ['Large', 6, 7, 8],
  ].forEach((row, idx) => {
    const ret1 = excelize.CoordinatesToCellName(1, idx + 1);
    if (ret1.error) {
      console.log(ret1.error);
      return;
    }
    const res2 = f.SetSheetRow('Sheet1', ret1.cell, row);
    if (res2.error) {
      console.log(res2.error);
      return;
    }
  });
  const ret3 = f.AddChart('Sheet1', 'E1', {
    Type: excelize.Col3DClustered,
    Series: [
      {
        Name: 'Sheet1!$A$2',
        Categories: 'Sheet1!$B$1:$D$1',
        Values: 'Sheet1!$B$2:$D$2',
      },
      {
        Name: 'Sheet1!$A$3',
        Categories: 'Sheet1!$B$1:$D$1',
        Values: 'Sheet1!$B$3:$D$3',
      },
      {
        Name: 'Sheet1!$A$4',
        Categories: 'Sheet1!$B$1:$D$1',
        Values: 'Sheet1!$B$4:$D$4',
      },
    ],
    Title: [{
      Text: 'Fruit 3D Clustered Column Chart',
    }],
  });
  if (ret3.error) {
    console.log(ret3.error);
    return;
  }
  // Save spreadsheet by the given path.
  const { buffer, error } = f.WriteToBuffer();
  if (error) {
    console.log(error);
    return;
  }
  fs.writeFile('Book1.xlsx', buffer, 'binary', (error) => {
    if (error) {
      console.log(error);
    }
  });
});
```

### Add picture to spreadsheet file

```javascript
const { init } = require('excelize-wasm');
const fs = require('fs');

init('./node_modules/excelize-wasm/excelize.wasm.gz').then((excelize) => {
  const f = excelize.OpenReader(fs.readFileSync('Book1.xlsx'));
  if (f.error) {
    console.log(f.error);
    return;
  }
  // Insert a picture.
  const ret1 = f.AddPictureFromBytes('Sheet1', 'A2', {
    Extension: '.png',
    File: fs.readFileSync('image.png'),
    Format: { AltText: 'Picture 1' },
  });
  if (ret1.error) {
    console.log(ret1.error);
    return;
  }
  // Insert a picture to worksheet with scaling.
  const ret2 = f.AddPictureFromBytes('Sheet1', 'D2', {
    Extension: '.jpg',
    File: fs.readFileSync('image.jpg'),
    Format: { AltText: 'Picture 2', ScaleX: 0.5, ScaleY: 0.5 },
  });
  if (ret2.error) {
    console.log(ret2.error);
    return;
  }
  // Insert a picture offset in the cell with printing support.
  const ret3 = f.AddPictureFromBytes('Sheet1', 'H2', {
    Extension: '.gif',
    File: fs.readFileSync('image.gif'),
    Format: {
      AltText: 'Picture 3',
      OffsetX: 15,
      OffsetY: 10,
      PrintObject: true,
      LockAspectRatio: false,
      Locked: false,
    },
  });
  if (ret3.error) {
    console.log(ret3.error);
    return;
  }
  // Save spreadsheet by the given path.
  const { buffer, error } = f.WriteToBuffer();
  if (error) {
    console.log(error);
    return;
  }
  fs.writeFile('Book1.xlsx', buffer, 'binary', (error) => {
    if (error) {
      console.log(error);
    }
  });
});
```

### Add pivot table to spreadsheet file
<p align="center"><img width="650" src="https://raw.githubusercontent.com/vembarrajan/excelize-wasm/main/Pivot.png" alt="Create Pivot by excelize-wasm"></p>

```javascript
init('./node_modules/excelize-wasm/excelize.wasm.gz').then((excelize) => {
  const f = new excelize.NewFile();
  // Create some data in a sheet
  const month = [
    'Jan',
    'Feb',
    'Mar',
    'Apr',
    'May',
    'Jun',
    'Jul',
    'Aug',
    'Sep',
    'Oct',
    'Nov',
    'Dec',
  ];
  const year = [2017, 2018, 2019];
  const types = ['Meat', 'Dairy', 'Beverages', 'Produce'];
  const region = ['East', 'West', 'North', 'South'];

  f.SetSheetRow('Sheet1', 'A1', ['Month', 'Year', 'Type', 'Sales', 'Region']);

  for (let row = 2; row < 32; row++) {
    f.SetCellValue('Sheet1', `A${row}`, month[Math.floor(Math.random() * 12)]);
    f.SetCellValue('Sheet1', `B${row}`, year[Math.floor(Math.random() * 3)]);
    f.SetCellValue('Sheet1', `C${row}`, types[Math.floor(Math.random() * 4)]);
    f.SetCellValue('Sheet1', `D${row}`, Math.floor(Math.random() * 5000));
    f.SetCellValue('Sheet1', `E${row}`, region[Math.floor(Math.random() * 4)]);
  }

  try {
    f.AddPivotTable({
      DataRange: 'Sheet1!A1:E31',
      PivotTableRange: 'Sheet1!G2:M34',
      Rows: [{ Data: 'Month', DefaultSubtotal: true }, { Data: 'Year' }],
      Filter: [{ Data: 'Region' }],
      Columns: [{ Data: 'Type', DefaultSubtotal: true }],
      Data: [{ Data: 'Sales', Name: 'Summarize', Subtotal: 'Sum' }],
      RowGrandTotals: true,
      ColGrandTotals: true,
      ShowDrill: true,
      ShowRowHeaders: true,
      ShowColHeaders: true,
      ShowLastColumn: true,
    });
    const { buffer, error } = f.WriteToBuffer();
    if (error) {
      console.log(error);
      return;
    }
    fs.writeFile('Book4.xlsx', buffer, 'binary', (error) => {
      if (error) {
        console.log(error);
      }
    });
  } catch (err) {
    console.log(err);
    return;
  }
});
```

## Contributing

Contributions are welcome! Open a pull request to fix a bug, or open an issue to discuss a new feature or change.

## Licenses

This program is under the terms of the BSD 3-Clause License. See [https://opensource.org/licenses/BSD-3-Clause](https://opensource.org/licenses/BSD-3-Clause).

The Excel logo is a trademark of [Microsoft Corporation](https://aka.ms/trademarks-usage). This artwork is an adaptation.

gopher.{ai,svg,png} was created by [Takuya Ueda](https://twitter.com/tenntenn). Licensed under the [Creative Commons 3.0 Attributions license](http://creativecommons.org/licenses/by/3.0/).
