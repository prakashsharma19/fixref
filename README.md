<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Author Entries Splitter</title>
<style>
  body {
    font-family: Arial, sans-serif;
    background: #f8f9fa;
    margin: 40px;
    color: #333;
  }
  h2 { color: #007bff; }
  .container {
    background: #fff;
    padding: 25px;
    border-radius: 12px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    max-width: 700px;
    margin: auto;
  }
  input, button {
    margin: 10px 5px 0 0;
  }
  button {
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 8px;
    padding: 10px 18px;
    cursor: pointer;
  }
  button:hover {
    background-color: #0056b3;
  }
  a {
    display: inline-block;
    margin: 5px 0;
    color: #007bff;
    text-decoration: none;
  }
  a:hover { text-decoration: underline; }
</style>
</head>
<body>
<div class="container">
  <h2>Author Entries Splitter Tool</h2>
  <p>Upload your text file containing author entries:</p>
  <input type="file" id="fileInput" accept=".txt">
  <br>
  <button onclick="splitFile()">Split File</button>
  <button id="downloadAllBtn" style="display:none;" onclick="downloadAll()">Download All Parts</button>
  <div id="output"></div>
</div>

<script>
let splitFiles = [];
let baseFileName = "";

function splitFile() {
  const fileInput = document.getElementById('fileInput');
  const file = fileInput.files[0];
  if (!file) {
    alert("Please select a text file first.");
    return;
  }
  baseFileName = file.name.replace(/\.[^/.]+$/, ""); // remove extension

  const reader = new FileReader();
  reader.onload = function(e) {
    const text = e.target.result;
    const entries = text.split(/\n\s*\n/); // split by blank lines between author entries
    const chunkSize = 690;
    splitFiles = [];
    document.getElementById('output').innerHTML = "";

    for (let i = 0; i < entries.length; i += chunkSize) {
      const partEntries = entries.slice(i, i + chunkSize).join("\n\n");
      const blob = new Blob([partEntries], {type: "text/plain"});
      const partNum = Math.floor(i / chunkSize) + 1;
      const fileName = `${baseFileName}_part${partNum}.txt`;

      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.download = fileName;
      link.textContent = `Download ${fileName}`;
      document.getElementById('output').appendChild(link);
      document.getElementById('output').appendChild(document.createElement("br"));

      splitFiles.push({blob, fileName});
    }

    document.getElementById('downloadAllBtn').style.display = splitFiles.length > 1 ? 'inline-block' : 'none';
  };
  reader.readAsText(file);
}

function downloadAll() {
  splitFiles.forEach(file => {
    const link = document.createElement("a");
    link.href = URL.createObjectURL(file.blob);
    link.download = file.fileName;
    link.click();
  });
}
</script>
</body>
</html>
