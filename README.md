<!DOCTYPE html>
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
  h2 {
    color: #007bff;
  }
  .container {
    background: white;
    padding: 25px;
    border-radius: 12px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    max-width: 700px;
    margin: auto;
  }
  textarea {
    width: 100%;
    height: 250px;
    margin-top: 10px;
    padding: 10px;
    border-radius: 8px;
    border: 1px solid #ccc;
    resize: vertical;
  }
  button {
    margin: 10px 5px 0 0;
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
  .output {
    margin-top: 15px;
  }
  a {
    display: inline-block;
    margin: 5px 5px;
    color: #007bff;
    text-decoration: none;
  }
  a:hover {
    text-decoration: underline;
  }
</style>
</head>
<body>
<div class="container">
  <h2>Author Entries Splitter Tool</h2>
  <p>Paste your text entries below or upload a text file:</p>
  <input type="file" id="fileInput" accept=".txt">
  <textarea id="textInput" placeholder="Paste or load your entries here..."></textarea>
  <br>
  <button onclick="splitEntries()">Split Entries</button>
  <button id="downloadAllBtn" style="display:none;" onclick="downloadAll()">Download All Parts</button>
  
  <div id="output" class="output"></div>
</div>

<script>
let splitFiles = [];

document.getElementById('fileInput').addEventListener('change', function(event) {
  const file = event.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = function(e) {
    document.getElementById('textInput').value = e.target.result;
  };
  reader.readAsText(file);
});

function splitEntries() {
  const text = document.getElementById('textInput').value.trim();
  if (!text) {
    alert("Please paste or load a text file first.");
    return;
  }

  const lines = text.split(/\r?\n/).filter(line => line.trim() !== "");
  const chunkSize = 690;
  splitFiles = [];
  let outputDiv = document.getElementById('output');
  outputDiv.innerHTML = "";

  for (let i = 0; i < lines.length; i += chunkSize) {
    const partLines = lines.slice(i, i + chunkSize);
    const blob = new Blob([partLines.join("\n")], {type: "text/plain"});
    const fileName = `entries_part_${Math.floor(i / chunkSize) + 1}.txt`;
    const link = document.createElement("a");
    link.href = URL.createObjectURL(blob);
    link.download = fileName;
    link.textContent = `Download ${fileName}`;
    outputDiv.appendChild(link);
    splitFiles.push({blob, fileName});
  }

  document.getElementById('downloadAllBtn').style.display = splitFiles.length > 1 ? 'inline-block' : 'none';
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
