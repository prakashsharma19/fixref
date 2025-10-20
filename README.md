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
    max-width: 750px;
    margin: auto;
  }
  input, button {
    margin: 10px 5px 0 0;
    padding: 10px 14px;
    border-radius: 8px;
    border: 1px solid #ccc;
  }
  button {
    background-color: #007bff;
    color: white;
    border: none;
    cursor: pointer;
  }
  button:hover { background-color: #0056b3; }
  a {
    display: inline-block;
    margin: 5px 0;
    color: #007bff;
    text-decoration: none;
  }
  a:hover { text-decoration: underline; }
  #output { margin-top: 15px; }
  .part-info {
    font-size: 14px;
    color: #555;
    margin-bottom: 5px;
  }
</style>
</head>
<body>
<div class="container">
  <h2>Author Entries Splitter Tool</h2>
  <p>Upload your <b>.txt</b> file containing author entries:</p>
  <input type="file" id="fileInput" accept=".txt">
  <br>
  <label for="entryCount">Entries per file:</label>
  <input type="number" id="entryCount" value="690" min="10" style="width:100px;">
  <br>
  <button onclick="splitFile()">Split File</button>
  <button id="downloadAllBtn" style="display:none;" onclick="downloadAll()">Download All Files</button>
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
  
  const entryCount = parseInt(document.getElementById('entryCount').value);
  if (!entryCount || entryCount <= 0) {
    alert("Please enter a valid number of entries per file.");
    return;
  }

  baseFileName = file.name.replace(/\.[^/.]+$/, ""); // remove extension
  const reader = new FileReader();

  reader.onload = function(e) {
    const text = e.target.result;
    // Split by double line breaks (each author entry separated by blank line)
    const entries = text.split(/\n\s*\n/).filter(e => e.trim() !== "");
    splitFiles = [];
    const outputDiv = document.getElementById('output');
    outputDiv.innerHTML = "";

    for (let i = 0; i < entries.length; i += entryCount) {
      const partEntries = entries.slice(i, i + entryCount);
      const partText = partEntries.join("\n\n");
      const blob = new Blob([partText], {type: "text/plain"});
      const partNum = Math.floor(i / entryCount) + 1;
      const countInPart = partEntries.length;
      const fileName = `${baseFileName}_part${partNum}_${countInPart}.txt`;

      // Info + download link
      const info = document.createElement("div");
      info.className = "part-info";
      info.textContent = `Part ${partNum} — ${countInPart} entries`;

      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.download = fileName;
      link.textContent = `Download ${fileName}`;

      outputDiv.appendChild(info);
      outputDiv.appendChild(link);
      outputDiv.appendChild(document.createElement("br"));

      splitFiles.push({blob, fileName});
    }

    // Show download all button if more than 1 file
    document.getElementById('downloadAllBtn').style.display =
      splitFiles.length > 1 ? 'inline-block' : 'none';
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
