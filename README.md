<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Author Entries Splitter Tool</title>
  <style>
    :root {
      --bg: #f3f6fb;
      --surface: #ffffff;
      --surface-soft: #f8faff;
      --text: #1f2937;
      --muted: #6b7280;
      --primary: #1d4ed8;
      --primary-dark: #1e40af;
      --border: #dbe4f2;
      --success-bg: #ecfdf3;
      --success-text: #166534;
    }

    * { box-sizing: border-box; }

    body {
      margin: 0;
      font-family: "Inter", "Segoe UI", Arial, sans-serif;
      color: var(--text);
      background: linear-gradient(180deg, #eef4ff 0%, var(--bg) 25%, var(--bg) 100%);
      min-height: 100vh;
      padding: 30px 16px 50px;
    }

    .page {
      max-width: 980px;
      margin: 0 auto;
    }

    .hero {
      text-align: center;
      margin-bottom: 24px;
    }

    .hero h1 {
      margin: 0;
      font-size: clamp(1.5rem, 3vw, 2.1rem);
      color: #0f172a;
      letter-spacing: 0.2px;
    }

    .hero p {
      margin: 10px auto 0;
      color: var(--muted);
      max-width: 680px;
      line-height: 1.45;
    }

    .grid {
      display: grid;
      gap: 18px;
      grid-template-columns: repeat(auto-fit, minmax(310px, 1fr));
      align-items: start;
    }

    .card {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 16px;
      box-shadow: 0 10px 25px rgba(17, 24, 39, 0.06);
      padding: 20px;
    }

    .card h2 {
      margin: 0 0 8px;
      font-size: 1.1rem;
      color: #0f172a;
    }

    .card p {
      margin: 0 0 16px;
      color: var(--muted);
      font-size: 0.95rem;
      line-height: 1.4;
    }

    .field {
      display: flex;
      flex-direction: column;
      gap: 6px;
      margin-bottom: 14px;
    }

    label {
      font-size: 0.88rem;
      color: #374151;
      font-weight: 600;
    }

    input[type="file"],
    input[type="number"],
    textarea {
      width: 100%;
      border: 1px solid var(--border);
      border-radius: 10px;
      padding: 10px 12px;
      background: var(--surface-soft);
      color: var(--text);
      font-size: 0.92rem;
    }

    textarea {
      min-height: 120px;
      resize: vertical;
      line-height: 1.35;
    }

    .actions {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin-top: 8px;
    }

    button {
      border: 0;
      border-radius: 10px;
      padding: 10px 14px;
      font-weight: 600;
      cursor: pointer;
      transition: 0.15s ease;
      background: var(--primary);
      color: #fff;
    }

    button:hover { background: var(--primary-dark); }

    button.secondary {
      background: #e5edff;
      color: #1d4ed8;
    }

    button.secondary:hover {
      background: #dbe7ff;
      color: #1e3a8a;
    }

    .output {
      margin-top: 14px;
      border-top: 1px solid var(--border);
      padding-top: 12px;
    }

    .part-item {
      margin-bottom: 9px;
      font-size: 0.92rem;
    }

    a {
      color: #1d4ed8;
      text-decoration: none;
      font-weight: 600;
    }

    a:hover { text-decoration: underline; }

    .status {
      margin-top: 10px;
      border: 1px solid #b7e4c7;
      background: var(--success-bg);
      color: var(--success-text);
      border-radius: 10px;
      padding: 10px 12px;
      font-size: 0.9rem;
      line-height: 1.4;
      white-space: pre-line;
    }
  </style>
</head>
<body>
  <main class="page">
    <header class="hero">
      <h1>Author Entries Splitter Tool</h1>
      <p>
        Split large author-entry files into parts and remove unsubscribed or bounced email entries in one clean workspace.
      </p>
    </header>

    <section class="grid">
      <article class="card">
        <h2>Split Entries File</h2>
        <p>Upload an entries file and choose how many entries should be included in each output file.</p>

        <div class="field">
          <label for="fileInput">Entries file (.txt)</label>
          <input type="file" id="fileInput" accept=".txt" />
        </div>

        <div class="field">
          <label for="entryCount">Entries per file</label>
          <input type="number" id="entryCount" value="690" min="1" />
        </div>

        <div class="actions">
          <button onclick="splitFile()">Split File</button>
          <button id="downloadAllBtn" class="secondary" style="display:none;" onclick="downloadAll()">Download All Files</button>
        </div>

        <div id="splitOutput" class="output"></div>
      </article>

      <article class="card">
        <h2>Remove Unsubscribed/Bounced Entries</h2>
        <p>Upload entries plus removal emails (file or pasted list), then download a cleaned entries file.</p>

        <div class="field">
          <label for="entriesFile">1) Entries file (.txt)</label>
          <input type="file" id="entriesFile" accept=".txt" />
        </div>

        <div class="field">
          <label for="removeFile">2) Remove entries file (.txt, .csv, .xls, .xlsx, optional)</label>
          <input type="file" id="removeFile" accept=".txt,.csv,.xls,.xlsx" />
        </div>

        <div class="field">
          <label for="removeEmailsText">Or paste emails to remove (one per line / comma-separated)</label>
          <textarea id="removeEmailsText" placeholder="bounce@example.com
unsubscribe@example.com"></textarea>
        </div>

        <div class="actions">
          <button onclick="removeEntriesAndDownload()">Remove &amp; Download Clean File</button>
        </div>

        <div id="removeStatus"></div>
      </article>
    </section>
  </main>

  <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
  <script>
    let splitFiles = [];

    function readFileAsText(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = (event) => resolve(event.target.result || "");
        reader.onerror = () => reject(new Error("Could not read file."));
        reader.readAsText(file);
      });
    }

    function splitEntries(text) {
      return text
        .split(/\r?\n\s*\r?\n/)
        .map(entry => entry.trim())
        .filter(Boolean);
    }

    function splitFile() {
      const file = document.getElementById("fileInput").files[0];
      const entryCount = Number.parseInt(document.getElementById("entryCount").value, 10);

      if (!file) {
        alert("Please select an entries file to split.");
        return;
      }

      if (!entryCount || entryCount <= 0) {
        alert("Please enter a valid entries-per-file number.");
        return;
      }

      readFileAsText(file).then((text) => {
        const entries = splitEntries(text);
        const output = document.getElementById("splitOutput");
        const downloadAllButton = document.getElementById("downloadAllBtn");

        splitFiles = [];
        output.innerHTML = "";

        const baseName = file.name.replace(/\.[^/.]+$/, "");

        for (let i = 0; i < entries.length; i += entryCount) {
          const partEntries = entries.slice(i, i + entryCount);
          const partNumber = Math.floor(i / entryCount) + 1;
          const fileName = `${baseName}_part${partNumber}_${partEntries.length}.txt`;
          const blob = new Blob([partEntries.join("\n\n")], { type: "text/plain" });
          const url = URL.createObjectURL(blob);

          const row = document.createElement("div");
          row.className = "part-item";

          const label = document.createElement("span");
          label.textContent = `Part ${partNumber} — ${partEntries.length} entries — `;

          const link = document.createElement("a");
          link.href = url;
          link.download = fileName;
          link.textContent = `Download ${fileName}`;

          row.appendChild(label);
          row.appendChild(link);
          output.appendChild(row);

          splitFiles.push({ blob, fileName });
        }

        downloadAllButton.style.display = splitFiles.length > 1 ? "inline-block" : "none";
      }).catch(() => {
        alert("Unable to process the selected file.");
      });
    }

    function downloadAll() {
      splitFiles.forEach((file) => {
        const link = document.createElement("a");
        link.href = URL.createObjectURL(file.blob);
        link.download = file.fileName;
        link.click();
      });
    }

    function parseEmailsFromText(text) {
      const matches = text.match(/[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}/gi) || [];
      return new Set(matches.map(email => email.toLowerCase().trim()));
    }

    function parseCsvLine(line) {
      const values = [];
      let current = "";
      let inQuotes = false;

      for (let i = 0; i < line.length; i += 1) {
        const char = line[i];

        if (char === '"') {
          const nextChar = line[i + 1];
          if (inQuotes && nextChar === '"') {
            current += '"';
            i += 1;
          } else {
            inQuotes = !inQuotes;
          }
          continue;
        }

        if (char === "," && !inQuotes) {
          values.push(current.trim());
          current = "";
          continue;
        }

        current += char;
      }

      values.push(current.trim());
      return values;
    }

    function normalizeHeaderName(value) {
      return String(value || "")
        .trim()
        .toLowerCase()
        .replace(/[^a-z0-9]/g, "");
    }

    function findColumnIndex(headerRow, aliases) {
      const normalizedAliases = aliases.map(normalizeHeaderName);
      return headerRow.findIndex((name) => normalizedAliases.includes(name));
    }

    function getBouncedEmailsFromRows(rows) {
      if (!rows.length) {
        return { emails: new Set(), bouncedCount: 0, matchedBounceFormat: false };
      }

      const headerRow = rows[0].map(normalizeHeaderName);
      const eventTypeIndex = findColumnIndex(headerRow, ["eventtype", "event", "status", "type"]);
      const toIndex = findColumnIndex(headerRow, ["to", "recipient", "email", "recipientemail", "toemail"]);

      if (eventTypeIndex === -1 || toIndex === -1) {
        return { emails: new Set(), bouncedCount: 0, matchedBounceFormat: false };
      }

      const bouncedEmails = new Set();
      let bouncedCount = 0;

      rows.slice(1).forEach((row) => {
        const eventType = String(row[eventTypeIndex] || "").trim().toLowerCase();
        if (!eventType.includes("bounce")) {
          return;
        }

        bouncedCount += 1;
        const toValue = String(row[toIndex] || "");
        parseEmailsFromText(toValue).forEach(email => bouncedEmails.add(email));
      });

      return { emails: bouncedEmails, bouncedCount, matchedBounceFormat: true };
    }

    async function getRemovalDataFromFile(file) {
      const extension = file.name.split(".").pop()?.toLowerCase() || "";

      if (extension === "csv") {
        const csvText = await readFileAsText(file);
        const rows = csvText
          .split(/\r?\n/)
          .filter(line => line.trim())
          .map(parseCsvLine);
        return getBouncedEmailsFromRows(rows);
      }

      if (extension === "xls" || extension === "xlsx") {
        if (typeof XLSX === "undefined") {
          throw new Error("Excel parser library is unavailable.");
        }

        const buffer = await file.arrayBuffer();
        const workbook = XLSX.read(buffer, { type: "array" });

        const mergedData = { emails: new Set(), bouncedCount: 0, matchedBounceFormat: false };

        workbook.SheetNames.forEach((sheetName) => {
          const sheet = workbook.Sheets[sheetName];
          const rows = XLSX.utils.sheet_to_json(sheet, { header: 1, blankrows: false });
          const result = getBouncedEmailsFromRows(rows);

          result.emails.forEach(email => mergedData.emails.add(email));
          mergedData.bouncedCount += result.bouncedCount;
          mergedData.matchedBounceFormat = mergedData.matchedBounceFormat || result.matchedBounceFormat;
        });

        if (!mergedData.matchedBounceFormat) {
          const allCellText = workbook.SheetNames
            .map(sheetName => {
              const sheet = workbook.Sheets[sheetName];
              return XLSX.utils.sheet_to_csv(sheet);
            })
            .join("\n");
          return { emails: parseEmailsFromText(allCellText), bouncedCount: 0, matchedBounceFormat: false };
        }

        return mergedData;
      }

      const text = await readFileAsText(file);
      return { emails: parseEmailsFromText(text), bouncedCount: 0, matchedBounceFormat: false };
    }

    function entryHasRemovalEmail(entry, removalSet) {
      const emails = entry.match(/[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}/gi) || [];
      return emails.some(email => removalSet.has(email.toLowerCase().trim()));
    }

    function downloadTextFile(content, filename) {
      const blob = new Blob([content], { type: "text/plain" });
      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.download = filename;
      link.click();
    }

    async function removeEntriesAndDownload() {
      const entriesFile = document.getElementById("entriesFile").files[0];
      const removeFile = document.getElementById("removeFile").files[0];
      const manualEmails = document.getElementById("removeEmailsText").value;
      const statusBox = document.getElementById("removeStatus");

      statusBox.innerHTML = "";

      if (!entriesFile) {
        alert("Please upload the entries file first.");
        return;
      }

      const removalSet = parseEmailsFromText(manualEmails.trim());
      let bouncedCount = 0;
      let matchedBounceFormat = false;

      if (removeFile) {
        try {
          const removeFileData = await getRemovalDataFromFile(removeFile);
          removeFileData.emails.forEach(email => removalSet.add(email));
          bouncedCount = removeFileData.bouncedCount;
          matchedBounceFormat = Boolean(removeFileData.matchedBounceFormat);
        } catch {
          alert("Could not read remove entries file.");
          return;
        }
      }

      if (!removalSet.size) {
        alert("Please provide at least one valid email in remove entries file or text box.");
        return;
      }

      let entriesText = "";
      try {
        entriesText = await readFileAsText(entriesFile);
      } catch {
        alert("Could not read entries file.");
        return;
      }

      const entries = splitEntries(entriesText);
      const keptEntries = entries.filter(entry => !entryHasRemovalEmail(entry, removalSet));
      const removedCount = entries.length - keptEntries.length;
      const outputName = `${entriesFile.name.replace(/\.[^/.]+$/, "")}_Clean.txt`;

      downloadTextFile(keptEntries.join("\n\n"), outputName);

      const bounceStatusLine = matchedBounceFormat
        ? `Bounced emails detected in log file: ${bouncedCount}`
        : "Bounced emails detected in log file: Not found (used all emails from remove file)";

      statusBox.className = "status";
      statusBox.textContent = `Done.\n${bounceStatusLine}\nTotal entries: ${entries.length}\nRemoved entries: ${removedCount}\nRemaining entries: ${keptEntries.length}\nDownloaded file: ${outputName}`;
    }
  </script>
</body>
</html>
