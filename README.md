Special Characters Identifiers - Fix the References

<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Text Extraction</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f2f2f2;
            margin: 0;
            padding: 0;
        }

        .container {
            display: flex;
            flex-direction: column;
            align-items: center;
            max-width: 900px;
            margin: 50px auto;
            background-color: #ffffff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
        }

        .left-container,
        .right-container {
            flex: 1;
            width: 100%;
            margin-bottom: 20px;
        }

        h2 {
            text-align: center;
            margin-top: 0;
            color: #333;
        }

        textarea {
            width: 100%;
            height: 120px;
            resize: vertical;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            margin-bottom: 15px;
        }

        .box-container {
            display: flex;
            align-items: center;
            margin-bottom: 15px;
        }

        .box-container textarea {
            flex: 1;
            margin-right: 10px;
        }

        button {
            padding: 10px 15px;
            background-color: #4CAF50;
            color: #ffffff;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
        }

        .extract-button {
            width: 100%;
            margin-bottom: 20px;
        }

        .copy-button {
            background-color: #2196F3;
        }

        .fix-button {
            background-color: #FF9800;
        }

        .refresh-button {
            padding: 10px 15px;
            background-color: #03A9F4;
            color: #ffffff;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
            margin-left: 10px;
        }

        .footer {
            text-align: center;
            font-size: 12px;
            margin-top: 20px;
            color: #777;
        }

        .doi-circle {
            position: fixed;
            top: 50%;
            right: 20px;
            transform: translateY(-50%);
            width: 50px;
            height: 50px;
            border-radius: 50%;
            background-color: #888;
            display: flex;
            justify-content: center;
            align-items: center;
            font-weight: bold;
            font-size: 20px;
            color: #fff;
        }

        .abstract-container {
            margin-top: 20px;
        }

        #extractedAbstract,
        #extractedReferences {
            white-space: pre-line;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            margin-top: 10px;
            background-color: #f9f9f9;
        }

        .highlight {
            font-weight: bold;
            color: fuchsia;
        }
    </style>
</head>

<body>
    <div class="container">
        <div class="left-container">
            <h2>Fix References - PPH 
                <button class="refresh-button" onclick="location.reload()">Refresh</button>
            </h2>
            <textarea id="input" placeholder="Paste the text here"></textarea>
            <button class="extract-button" onclick="extractText()">Extract</button>
        </div>
        <div class="right-container">
            <div class="references-container">
                <h2>Extracted References</h2>
                <div id="extractedReferences" contenteditable="true"></div>
                <div class="box-container">
                    <button class="copy-button" onclick="copyToClipboard()">Copy</button>
                    <button class="fix-button" onclick="fixHighlightedText()">Fix</button>
                </div>
            </div>
        </div>
    </div>
    <div class="doi-circle" id="doiCircle"></div>
    <div class="footer">
        <p>By Prakash with &#128522;</p>
    </div>
    <script>
        function extractText() {
            var inputText = document.getElementById("input").value;
            var referencesText = extractSection(inputText, "References:");
            document.getElementById("extractedReferences").innerHTML = highlightSpecialCharacters(referencesText);

            var doiCircle = document.getElementById("doiCircle");
            var lastTwoDigits = extractDOI(inputText).slice(-2);
            doiCircle.innerText = lastTwoDigits;
        }

        function extractSection(text, startMarker, endMarker) {
            var startIndex = text.indexOf(startMarker);
            var endIndex = endMarker ? text.indexOf(endMarker, startIndex + startMarker.length) : text.length;
            return text.substring(startIndex + startMarker.length, endIndex).trim();
        }

        function extractDOI(text) {
            var doiStartIndex = text.indexOf("http://dx.doi.org/") + "http://dx.doi.org/".length;
            var doiEndIndex = text.indexOf("\n", doiStartIndex);
            return text.substring(doiStartIndex, doiEndIndex).trim();
        }

        function highlightSpecialCharacters(text) {
            var specialCharacters = /[^a-zA-Z0-9\s\.,;:()\-]/g;
            return text.replace(specialCharacters, function (match) {
                return '<span class="highlight">' + match + '</span>';
            });
        }

        function fixHighlightedText() {
            var extractedReferences = document.getElementById("extractedReferences");
            var spans = extractedReferences.getElementsByTagName("span");
            for (var i = 0; i < spans.length; i++) {
                var span = spans[i];
                var normalText = span.innerText.normalize("NFD").replace(/[\u0300-\u036f]/g, "");
                span.innerHTML = normalText;
            }
        }

        function copyToClipboard() {
            var extractedReferences = document.getElementById("extractedReferences");
            var range = document.createRange();
            range.selectNode(extractedReferences);
            window.getSelection().removeAllRanges();
            window.getSelection().addRange(range);
            document.execCommand("copy");
            window.getSelection().removeAllRanges();
        }
    </script>
</body>

</html>
