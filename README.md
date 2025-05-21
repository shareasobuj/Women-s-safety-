

<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Women's safety </title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <script src="https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f8f8f8;
      padding: 20px;
    }
    h1 {
      text-align: center;
      color: #b30000;
    }
    input, textarea, button {
      width: 100%;
      margin: 10px 0;
      padding: 12px;
      font-size: 16px;
      border: 1px solid #ccc;
      border-radius: 6px;
    }
    button {
      background-color: #b30000;
      color: white;
      cursor: pointer;
    }
    button:hover {
      background-color: #900;
    }
    table {
      width: 100%;
      margin-top: 20px;
      border-collapse: collapse;
    }
    th, td {
      border: 1px solid #ccc;
      padding: 8px;
      text-align: center;
    }
    th {
      background: #eee;
    }
  </style>
</head>
<body>

<h1>Women's safety </h1>

<input type="text" id="name" placeholder="Your Full Name" required />
<input type="text" id="mobile" placeholder="Mobile Number" required />
<textarea id="message" placeholder="Your Emergency Message" rows="3" required></textarea>
<button onclick="sendSOS()">Send SOS</button>
<button onclick="exportToExcel()">Download Excel</button>

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Mobile</th>
      <th>Message</th>
      <th>Location</th>
      <th>Time</th>
    </tr>
  </thead>
  <tbody id="sosTable"></tbody>
</table>

<script>
  const table = document.getElementById('sosTable');

  function sendSOS() {
    const name = document.getElementById('name').value.trim();
    const mobile = document.getElementById('mobile').value.trim();
    const message = document.getElementById('message').value.trim();

    if (!name || !mobile || !message) {
      alert("Please fill in all fields.");
      return;
    }

    navigator.geolocation.getCurrentPosition(async (pos) => {
      const lat = pos.coords.latitude.toFixed(4);
      const lon = pos.coords.longitude.toFixed(4);
      const time = new Date().toLocaleString();

      let location = `Lat: ${lat}, Lon: ${lon}`;
      try {
        const response = await fetch(`https://nominatim.openstreetmap.org/reverse?format=json&lat=${lat}&lon=${lon}`);
        const data = await response.json();
        if (data.display_name) location = data.display_name;
      } catch {}

      const entry = { name, mobile, message, location, time };

      // Add to table
      addRow(entry);

      // Save to localStorage
      const logs = JSON.parse(localStorage.getItem("sosLogs") || "[]");
      logs.push(entry);
      localStorage.setItem("sosLogs", JSON.stringify(logs));
    }, () => {
      alert("Location access is required.");
    });
  }

  function addRow(entry) {
    const row = document.createElement("tr");
    row.innerHTML = `
      <td>${entry.name}</td>
      <td>${entry.mobile}</td>
      <td>${entry.message}</td>
      <td>${entry.location}</td>
      <td>${entry.time}</td>
    `;
    table.appendChild(row);
  }

  function exportToExcel() {
    const logs = JSON.parse(localStorage.getItem("sosLogs") || "[]");
    if (logs.length === 0) {
      alert("No data to export.");
      return;
    }

    const worksheet = XLSX.utils.json_to_sheet(logs);
    const workbook = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(workbook, worksheet, "SOS_Logs");
    XLSX.writeFile(workbook, "SOS_Logs.xlsx");
  }

  // Load saved data on page load
  window.onload = () => {
    const logs = JSON.parse(localStorage.getItem("sosLogs") || "[]");
    logs.forEach(addRow);
  };
</script>

</body>
</html>
