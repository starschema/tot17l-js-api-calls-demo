# Underlying Data
## Useing Tableau JSApi without embedding
### Description
The basic concept: http://databoss.starschema.net/tableau-javascript-api-without-embedding/

Everybody knows how to add a website view to any dashboards. 

Not everyone knows that, from this website view, we can reach the properties of vizualization. This little demo file do this exactly.

In this example, we show how to use 3 basis functionality of the JSApi. (Underlying data; Selected data; Get filters)

First things first, we created a little library to reach the properties of the vizualization. (`tableaLib.js`)
In this example we will only use the `getCurrentWorksheet()` method, to get our current worksheet. 

The HTML part of it is very simple. Just a headline, 3 buttons, and 3 tables. Bootstrap will do the CSS part. The javascript part of this example is also not too complicated. We have to implement 3 click event listeners, for each button, we have to get the current worksheet with the `getCurrentWorksheet()` method, and use 2 Tableau JSApi functions (`getUnderlyingDataAsync()`, `getFiltersAsync()`). And thats basically thats all. The only thing we have to do after, is to handle the function responses. We build a HTML table from them, but you can do whatever you want to do ;)

## Technical documentation
### HTML
**Just a headline**
```html
<div class="row center">
    <h2 class="headline">JavaScript API without Embedding</h2>
</div>
```

**Define buttons**
```html
<div class="row center button-row">
    <button class="btn btn-default" onclick="onUnderlyingDataClicked()">Underlying Data</button>
    <button class="btn btn-default" onclick="onCurrentSelectionClicked()">Current Selection</button>
    <button class="btn btn-default" onclick="onCurrentFiltersClicked()">Current Filter</button>
</div>
```
*onclick* methods must exist in the javascript part

**Data tables**
```html
<div class="row data-table-row">
    <table class="table table-hover" id="underlyingDataTable">
    </table>
    <table class="table table-hover" id="currentSelectionTable">
    </table>
    <table class="table table-hover" id="currentFilterTable">
    </table>
</div>
```

### JavaScript
**We have to include Tableau JSApi, and the TableauLib**
```html
<!-- Replace tableau.server to your server domain name -->
<script src="http://tableau.server/javascripts/api/tableau-2.1.1.min.js"></script>
<script src="tableauLib.js"></script>
```

**Define worksheet**
```js
var TBLib = new TableauLib();
var worksheet = TBLib.getCurrentWorksheet();
```

**Click handlers**
```js
// Click handlers
function onUnderlyingDataClicked() {
    hideOtherTables();
    queryUnderlyingData(true, onUnderlyingDataSuccess);
}

function onCurrentSelectionClicked() {
    hideOtherTables();
    queryUnderlyingData(false, onCurrentSelectionSuccess);
}

function onCurrentFiltersClicked() {
    hideOtherTables();
    queryCurrentFilters(onCurrentFiltesSuccess);
}
```
All table is hidden by default. If you click a button, the related table will be visible, after all the data recieved. `hideOtherTables()` method hide all visible table, to provide place for the selected table. 

**Query functions**
```js
// Query functions
function queryUnderlyingData(ignoreSelection, cb) {
    worksheet.getUnderlyingDataAsync({
        maxRows: 0,
        ignoreSelection: ignoreSelection,
        includeAllColumns: true,
        ignoreAliases: true
    }).then(cb, onDataLoadError);
}

function queryCurrentFilters(cb) {
    worksheet.getFiltersAsync().then(cb);
}
```
`queryUnderlyingData()` gets two parametes. First one is a *boolean*, to ignore selection or not. The second one is a *callback function*. This method calls the `getUnderlyingDataAsync()` function of Tableau JSApi.

`queryCurrentFiltes()` gets one parameter, a *callback function*. This method calls the `getFiltersAsync()` function of Tableau JSApi.

**Callback functions**
```js
// Callback functions
function onDataLoadError(err) {
    return console.error("Error during Tableau Async request:", err);
}

function onUnderlyingDataSuccess(table) {
    var tableData = table.getData();
    var tableCols = table.getColumns();
    buildHTMLTableFromDataArray(tableCols, tableData, "underlyingDataTable");
}

function onCurrentSelectionSuccess(table) {
    var tableData = table.getData();
    var tableCols = table.getColumns();
    buildHTMLTableFromDataArray(tableCols, tableData, "currentSelectionTable");
}

function onCurrentFiltesSuccess(table) {
    buildFiltersTableFromTableData(table, "currentFilterTable");
}  
```
`onDataLoadError()` function is our error handler. Not too complicated. Just print the error to the console.

`onUnderlyingDataSuccess()` function handles the response of the `queryUnderlyingData()` function. This method gets a parameter, which is an *object*. We can get the tabel data by calling the `getData()` function. Also we want to get the name of the columns, by calling the `getColumns()` function. After that we can build our HTML table by calling the `buildHTMLTableFromDataArray()` function. 

`onCurrentSelectionSuccess()` is basically the same. The only difference is the third parameter of the `buildHTMLTableFromDataArray()` method, which is the tableID.

`onCurrentFiltesSuccess()` will handle the response of the `getFiltersAsync()`. It will call the `buildFiltersTableFromTableData()` method, to build the HTML table.

**DOM manipulators**
```js
// DOM manipulators
function buildHTMLTableFromDataArray(tableCols, tableData, tableId) {
    var table = $('#'+tableId);
    table.empty();

    var tableHeading = '<tr>';
    tableCols.forEach( col => {
        tableHeading += '<th>' + col.getFieldName() + '</th>';
    });
    tableHeading += '</tr>';
    table.append(tableHeading);

    tableData.forEach( row => {
        var rowHTML = '';

        rowHTML += '<tr>';
        row.forEach( col => {
            rowHTML += '<td>' + col.formattedValue + '</td>';
        });
        rowHTML += '</tr>';

        table.append(rowHTML);
    } );

    table.show();
}

function buildFiltersTableFromTableData(table, tableId) {
    var tableHtml = $('#' + tableId);
    tableHtml.empty();

    var row = '';
    
    row += '<tr>';
    row += '<th>Worksheet Name</th>';
    row += '<th>Filter Type</th>';
    row += '<th>Field Name</th>';
    row += '</tr>';   
    tableHtml.append(row);

    for (var i = 0; i < table.length; i++) {
        row = '<tr>';
        row += '<td>' + table[i].getWorksheet().getName() + '</td>';
        row += '<td>' + table[i].getFilterType() + '</td>';
        row += '<td>' + table[i].getFieldName() + '</td>';
        row += '</tr>';   
        tableHtml.append(row);
    }

    tableHtml.show();
}

function hideOtherTables() {
    $('table').hide();
}
```
With no comment. Just some ugly dom append functions. 