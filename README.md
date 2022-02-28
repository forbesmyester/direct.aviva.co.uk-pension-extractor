# direct.aviva.co.uk pension extractor

My pensions provider Aviva has a pretty, but inaccessible user interface and does not offer any downloads of your pensions data / payments.

Suspect you could get it out by manually re-entering it all or doing a GDPR request.

However if you want your Pensions data the following (crude) JS will give you it as a TSV.

You'll need to go to  Pension > Details > Transaction History > Full History > All then open up the console (F12) and paste it in to run this.

```javascript
let dates = Array.from(document.querySelectorAll('.paymentDetails div > h3')).map(x => x.innerText);
let headers = Array.from(document.querySelectorAll('.paymentDetails ul h3')).map(x => x.innerText);
let data = Array.from(document.querySelectorAll('.paymentDetails ul p')).map(x => x.innerText);
let r = {["Date"]: []};
while (data.length) {
    let d = data.shift();
    let h = headers.shift();
    if (!r.hasOwnProperty(h)) { r[h] = []; }
    r[h].push(d);
    if (h == 'Payment type') {
        r["Date"].push(dates.shift());
    }
}
function zip(a, b) {
    let r = {};
    while (a.length) {
        r[a.shift()] = b.shift();
    }
    return r;
}
let dataExport = [];
while (r["Payment type"].length) {
    let rowKeys = Object.getOwnPropertyNames(r);
    let rowVals = Object.getOwnPropertyNames(r).map(name => {
        return r[name].shift();
    });
    dataExport.push(zip(rowKeys, rowVals));
}
console.log(JSON.stringify(dataExport));


function exportToCSV(rows, filename) {

    var header = Object.getOwnPropertyNames(rows[0]);
    var csv = [header.join("\t")];
    console.log("HEADER: ", header);

    while (rows.length) {
        let row = rows.shift();
        let csvRow = [];
        for(var i=0; i<header.length; i++){
            csvRow.push(
                row.hasOwnProperty(header[i]) ? row[header[i]] : ""
            )
        }
        csv.push(csvRow.join("\t"));
    }

    downloadCSV(csv.join("\n"), filename);
}

function downloadCSV(csv, filename) {
    var csvFile;
    var downloadLink;
    csvFile = new Blob([csv], {type:"text/csv"});
    downloadLink = document.createElement("a");
    downloadLink.download = filename;
    downloadLink.href = window.URL.createObjectURL(csvFile);
    downloadLink.style.display = "none";
    document.body.appendChild(downloadLink);
    downloadLink.click();
}


exportToCSV(dataExport, 'x.tsv');


```
