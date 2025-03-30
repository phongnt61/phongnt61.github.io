<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Multi-Format JWT URL Generator</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jsrsasign/10.5.25/jsrsasign-all-min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
    }
    .container {
      max-width: 800px;
      margin: 0 auto;
    }
    .section {
      background-color: #f8f9fa;
      padding: 15px;
      margin-bottom: 20px;
      border-radius: 5px;
      border: 1px solid #dee2e6;
    }
    .section h2 {
      margin-top: 0;
      color: #343a40;
    }
    label, input, select, button {
      display: block;
      margin-bottom: 10px;
    }
    input, select {
      width: 100%;
      padding: 8px;
      box-sizing: border-box;
    }
    button {
      padding: 10px 20px;
      background-color: #007BFF;
      color: white;
      border: none;
      cursor: pointer;
      margin-right: 10px;
    }
    button:hover {
      background-color: #0056b3;
    }
    .button-group {
      display: flex;
      margin-bottom: 10px;
    }
    .output {
      margin-top: 20px;
      padding: 10px;
      background-color: #f9f9f9;
      border: 1px solid #ddd;
      word-wrap: break-word;
    }
    .output p {
      margin: 5px 0;
    }
    .hidden {
      display: none;
    }
    .toggle-button {
      background-color: #6c757d;
    }
    .toggle-button:hover {
      background-color: #5a6268;
    }
    .section.atm {
      border-left: 4px solid #28a745;
    }
    .section.bpk {
      border-left: 4px solid #dc3545;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Multi-Format JWT URL Generator</h1>
    
    <!-- ATM Section -->
    <div class="section atm">
      <h2>ATM Template (HS512)</h2>
      <label for="domain1">Domain:</label>
      <input type="text" id="domain1" placeholder="rr1-ateme.tv360.vn" value="rr1-ateme.tv360.vn">
      
      <label for="uriPath1">URI Path after 'live':</label>
      <input type="text" id="uriPath1" placeholder="eds" value="eds">
      
      <label for="resourceId1">Resource ID:</label>
      <input type="text" id="resourceId1" placeholder="192" value="192">
      
      <label for="streamType1">Stream Type:</label>
      <select id="streamType1">
        <option value="CMAF_Clean_MHL_2s">CMAF Clean</option>
        <option value="DASH_DRM_MHL_2s">DASH DRM</option>
        <option value="HLS_Clean_MHL_2s">HLS Clean</option>
      </select>
      
      <label for="secretKey1">Secret Key:</label>
      <input type="text" id="secretKey1" placeholder="Your secret key" value="your_secret_key_here">
      
      <div class="button-group">
        <button onclick="generateUrls(1)">Generate ATM URLs</button>
        <button class="toggle-button" onclick="toggleOutput(1)">Hide Output</button>
      </div>
      
      <div class="output" id="output1"></div>
    </div>
    
    <!-- BPK Section -->
    <div class="section bpk">
      <h2>BPK Template (Custom Algorithm)</h2>
      <label for="domain2">Domain:</label>
      <input type="text" id="domain2" placeholder="fo-hlc.tv360.vn" value="fo-hlc.tv360.vn">
      
      <label for="resourceId2">Resource ID:</label>
      <input type="text" id="resourceId2" placeholder="194">
      
      <label for="streamType2">Stream Type:</label>
      <select id="streamType2">
        <option value="lowlat">lowlat</option>
        <option value="output">output</option>
        <option value="adaptive">adaptive</option>
      </select>
      
      <label for="secretKey2">Secret Key:</label>
      <input type="text" id="secretKey2" placeholder="Your BPK secret key">
      
      <label for="algorithm2">Algorithm:</label>
      <select id="algorithm2">
        <option value="HS256" selected>HS256</option>
        <option value="HS384">HS384</option>
        <option value="HS512">HS512</option>
      </select>
      
      <div class="button-group">
        <button onclick="generateUrls(2)">Generate BPK URLs</button>
        <button class="toggle-button" onclick="toggleOutput(2)">Hide Output</button>
      </div>
      
      <div class="output" id="output2"></div>
    </div>
  </div>

  <script>
    function generateUrls(sectionNum) {
      const prefix = sectionNum.toString();
      const outputDiv = document.getElementById(`output${prefix}`);
      outputDiv.innerHTML = "";

      if (sectionNum === 1) {
        // ATM Template Generation
        const domain = document.getElementById(`domain${prefix}`).value;
        const uriPath = document.getElementById(`uriPath${prefix}`).value;
        const resourceId = document.getElementById(`resourceId${prefix}`).value;
        const streamType = document.getElementById(`streamType${prefix}`).value;
        const secretKey = document.getElementById(`secretKey${prefix}`).value;

        // Determine file extension based on stream type
        let extension = "mpd";
        if (streamType.includes("HLS")) {
          extension = "m3u8";
        }

        const baseUrl = `https://${domain}/live/${uriPath}/${resourceId}/${streamType}/${resourceId}.${extension}`;
        const payload = {
          exp: Math.floor(Date.now() / 1000) + 3600,
          path: `/live/${uriPath}/${resourceId}/${streamType}/`
        };
        const sHeader = JSON.stringify({ alg: "HS512", typ: 'JWT' });
        const sPayload = JSON.stringify(payload);
        const sJWT = KJUR.jws.JWS.sign("HS512", sHeader, sPayload, secretKey);
        const urlWithToken = `${baseUrl}?cdntoken=${sJWT}`;

        outputDiv.innerHTML += `
          <p><strong>Stream Type:</strong> ${streamType}</p>
          <p><strong>JWT Token:</strong> ${sJWT}</p>
          <p><strong>URL:</strong> <a href="${urlWithToken}" target="_blank">${urlWithToken}</a></p>
          <hr>
        `;
      } else {
        // BPK Template Generation
        const domain = document.getElementById(`domain${prefix}`).value;
        const resourceId = document.getElementById(`resourceId${prefix}`).value;
        const streamType = document.getElementById(`streamType${prefix}`).value;
        const secretKey = document.getElementById(`secretKey${prefix}`).value;
        const algorithm = document.getElementById(`algorithm${prefix}`).value;

        const baseUrl = `https://${domain}/bpk-tv/${resourceId}/${streamType}/index.m3u8`;
        const uriPath = `/bpk-tv/${resourceId}/${streamType}/index.m3u8`;
        
        const payload = {
          uri: uriPath,
          method: "GET",
          exp: Math.floor(Date.now() / 1000) + 3600
        };
        
        const sHeader = JSON.stringify({ alg: algorithm, typ: 'JWT' });
        const sPayload = JSON.stringify(payload);
        const sJWT = KJUR.jws.JWS.sign(algorithm, sHeader, sPayload, secretKey);
        const urlWithToken = `${baseUrl}?auth=${sJWT}`;

        outputDiv.innerHTML += `
          <p><strong>Stream Type:</strong> ${streamType}</p>
          <p><strong>Algorithm:</strong> ${algorithm}</p>
          <p><strong>JWT Token:</strong> ${sJWT}</p>
          <p><strong>URL:</strong> <a href="${urlWithToken}" target="_blank">${urlWithToken}</a></p>
          <hr>
        `;
      }

      outputDiv.classList.remove('hidden');
      document.querySelector(`#output${prefix} + .button-group .toggle-button`).textContent = 'Hide Output';
    }

    function toggleOutput(sectionNum) {
      const outputDiv = document.getElementById(`output${sectionNum}`);
      const toggleButton = document.querySelector(`#output${sectionNum} + .button-group .toggle-button`);
      
      if (outputDiv.classList.contains('hidden')) {
        outputDiv.classList.remove('hidden');
        toggleButton.textContent = 'Hide Output';
      } else {
        outputDiv.classList.add('hidden');
        toggleButton.textContent = 'Show Output';
      }
    }
  </script>
</body>
</html>
