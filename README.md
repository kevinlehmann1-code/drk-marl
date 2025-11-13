<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8" />
<title>Leitstelle – DRK Kreis Recklinghausen</title>

<style>
  :root { color-scheme: dark; }
  body {
    font-family: system-ui, -apple-system, "Segoe UI", sans-serif;
    margin: 0;
    background: #020617;
    color: #e5e7eb;
  }
  header {
    background: linear-gradient(90deg, #1d4ed8, #dc2626);
    padding: 12px 18px;
    display: flex;
    align-items: center;
    gap: 10px;
    box-shadow: 0 4px 16px rgba(0,0,0,0.5);
  }
  header h1 {
    margin: 0;
    font-size: 1.4rem;
  }
  header span {
    font-size: 0.8rem;
    opacity: 0.9;
  }
  main {
    padding: 16px;
    display: grid;
    grid-template-columns: 1.2fr 1.5fr;
    gap: 16px;
  }
  .left, .right {
    display: flex;
    flex-direction: column;
    gap: 16px;
  }
  .card {
    background: #020617;
    border-radius: 12px;
    padding: 12px 14px;
    border: 1px solid #1f2937;
    box-shadow: 0 4px 16px rgba(0,0,0,0.4);
  }
  .card h2 {
    margin: 0 0 8px 0;
    font-size: 1rem;
  }
  .card h3 {
    margin: 10px 0 4px;
    font-size: 0.9rem;
  }
  label {
    display: block;
    font-size: 0.8rem;
    margin-top: 6px;
    margin-bottom: 2px;
    color: #9ca3af;
  }
  input, select, button {
    font: inherit;
    padding: 6px 8px;
    border-radius: 6px;
    border: 1px solid #1f2937;
    background: #020617;
    color: #e5e7eb;
  }
  input:focus, select:focus {
    outline: 2px solid #2563eb;
    outline-offset: 1px;
  }
  button {
    border: none;
    cursor: pointer;
    background: linear-gradient(90deg, #2563eb, #dc2626);
    font-weight: 600;
    margin-top: 8px;
    padding: 7px 10px;
  }
  button:hover {
    filter: brightness(1.1);
  }
  table {
    width: 100%;
    border-collapse: collapse;
    font-size: 0.8rem;
    margin-top: 10px;
  }
  th, td {
    border-bottom: 1px solid #111827;
    padding: 6px 4px;
    text-align: left;
    vertical-align: middle;
  }
  th {
    color: #9ca3af;
    font-weight: 500;
  }
  .status-badge {
    display: inline-block;
    padding: 2px 6px;
    border-radius: 999px;
    font-size: 0.7rem;
  }
  .status-frei        { background: rgba(34,197,94,0.15);   color:#4ade80; }
  .status-einsatz     { background: rgba(249,115,22,0.15);  color:#fb923c; }
  .status-ausserdienst{ background: rgba(148,163,184,0.15); color:#e5e7eb; }

  .form-row {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
    gap: 8px;
  }
  .small {
    font-size: 0.75rem;
    color: #9ca3af;
  }
  .stats-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
    gap: 8px;
    font-size: 0.8rem;
  }
  .stat-box {
    background: #020617;
    border-radius: 8px;
    border: 1px solid #111827;
    padding: 6px 8px;
  }
  .stat-label { color: #9ca3af; }
  .stat-value { font-weight: 700; margin-top: 2px; }

  .top-buttons {
    display: flex;
    justify-content: flex-end;
    gap: 8px;
    margin-bottom: 8px;
  }

  .filter-row {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    align-items: center;
    margin-top: 8px;
    margin-bottom: 4px;
  }
  .filter-row select {
    min-width: 130px;
  }

  @media (max-width: 900px) {
    main { grid-template-columns: 1fr; }
  }

  /* Druck-Ansicht */
  @media print {
    body { background: white; color: black; }
    header {
      background: white;
      color: black;
      box-shadow: none;
      border-bottom: 1px solid #ccc;
    }
    main { display: block; }
    .card {
      box-shadow: none;
      border: 1px solid #ddd;
      margin-bottom: 10px;
    }
    input, select, button { display: none !important; }
  }
</style>
</head>
<body>

<header>
  <div>🚑</div>
  <div>
    <h1>Leitstelle – DRK Kreis Recklinghausen (lokal)</h1>
    <span>Fahrzeuge manuell eintragen, Einsätze, Historie & Statistik – offline im Browser</span>
  </div>
</header>

<main>
  <div class="left">
    <!-- Statistik -->
    <div class="card">
      <h2>📊 Statistik</h2>
      <div id="statsContent" class="stats-grid"></div>
      <span class="small">Daten automatisch aus Fahrzeugen und Einsätzen berechnet.</span>
    </div>

    <!-- Fahrzeuge im Einsatz -->
    <div class="card">
      <h2>🚨 Fahrzeuge im Einsatz</h2>
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Funkname</th>
            <th>Typ</th>
            <th>OV</th>
            <th>Status</th>
          </tr>
        </thead>
        <tbody id="activeVehicleTableBody"></tbody>
      </table>
      <span class="small">Hier werden nur Fahrzeuge mit Status „Einsatz“ angezeigt.</span>
    </div>

    <!-- Fahrzeuge gesamt -->
    <div class="card">
      <h2>Fahrzeug hinzufügen</h2>
      <div class="form-row">
        <div>
          <label>Funkname</label>
          <input id="fahrzeugName" placeholder="z.B. RK MAR 02-RTW-1">
        </div>
        <div>
          <label>Typ</label>
          <input id="fahrzeugTyp" placeholder="RTW / KTW-B / BtKombi ...">
        </div>
        <div>
          <label>Ortsverein</label>
          <select id="fahrzeugOv">
            <option value="Recklinghausen">Recklinghausen</option>
            <option value="Marl">Marl</option>
            <option value="Gladbeck">Gladbeck</option>
            <option value="Dorsten">Dorsten</option>
            <option value="Oer-Erkenschwick">Oer-Erkenschwick</option>
            <option value="Datteln">Datteln</option>
            <option value="Einsatzeinheit">Einsatzeinheit</option>
            <option value="Sonstige">Sonstige</option>
          </select>
        </div>
      </div>
      <button onclick="addVehicle()">Fahrzeug hinzufügen</button>

      <div class="filter-row">
        <select id="filterOvSel">
          <option value="">OV: alle</option>
          <option value="Recklinghausen">Recklinghausen</option>
          <option value="Marl">Marl</option>
          <option value="Gladbeck">Gladbeck</option>
          <option value="Dorsten">Dorsten</option>
          <option value="Oer-Erkenschwick">Oer-Erkenschwick</option>
          <option value="Datteln">Datteln</option>
          <option value="Einsatzeinheit">Einsatzeinheit</option>
          <option value="Sonstige">Sonstige</option>
        </select>
        <select id="filterTypeSel">
          <option value="">Typ: alle</option>
          <option value="RTW">RTW</option>
          <option value="NEF">NEF</option>
          <option value="KTW">KTW</option>
          <option value="KTW-B">KTW-B</option>
          <option value="BtKombi">BtKombi</option>
          <option value="GW-San">GW-San</option>
          <option value="MTF">MTF</option>
        </select>
        <button type="button" onclick="reportFreeRTWs()">Freie RTWs melden</button>
      </div>

      <h2 style="margin-top:10px;">Alle Fahrzeuge</h2>
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Funkname</th>
            <th>Typ</th>
            <th>OV</th>
            <th>Status</th>
            <th>Ändern</th>
          </tr>
        </thead>
        <tbody id="vehicleTableBody"></tbody>
      </table>
    </div>
  </div>

  <div class="right">
    <div class="card">
      <div class="top-buttons">
        <button onclick="window.print()">🖨 Einsatzliste drucken</button>
      </div>

      <h2>Einsatz anlegen</h2>
      <div class="form-row">
        <div>
          <label>Beschreibung</label>
          <input id="einsatzBeschreibung" placeholder="z.B. VU, Person eingeklemmt">
        </div>
        <div>
          <label>Priorität</label>
          <select id="einsatzPrio">
            <option value="niedrig">niedrig</option>
            <option value="mittel" selected>mittel</option>
            <option value="hoch">hoch</option>
          </select>
        </div>
      </div>
      <button onclick="addIncident()">Einsatz anlegen</button>

      <h2 style="margin-top:18px;">Aktuelle Einsätze</h2>
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Beschreibung</th>
            <th>Prio</th>
            <th>Fahrzeuge</th>
            <th>Status</th>
            <th>Status ändern</th>
            <th>Dauer</th>
          </tr>
        </thead>
        <tbody id="incidentTableBody"></tbody>
      </table>

      <h3>Fahrzeug zuordnen</h3>
      <label>Einsatz wählen</label>
      <select id="assignIncidentSelect"></select>
      <label>Fahrzeug wählen</label>
      <select id="assignVehicleSelect"></select>
      <button onclick="assignVehicle()">Zuordnen</button>
      <span class="small">
        Zuordnung setzt Fahrzeug auf „Einsatz“. Wenn kein offener Einsatz mehr zu einem Fahrzeug gehört,
        wird es automatisch wieder „frei“ (außer „außer Dienst“).
      </span>

      <h2 style="margin-top:18px;">Einsatz-Historie (erledigt)</h2>
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Beschreibung</th>
            <th>Prio</th>
            <th>Dauer</th>
            <th>Beginn</th>
            <th>Ende</th>
          </tr>
        </thead>
        <tbody id="historyTableBody"></tbody>
      </table>
    </div>
  </div>
</main>

<script>
let vehicles = [];
let incidents = [];
let vehicleId = 1;
let incidentId = 1;

/* Keys für lokalen Speicher */
const KEY_V = "leitstelle_vehicles_drk_re_MANUELL_v1";
const KEY_I = "leitstelle_incidents_drk_re_MANUELL_v1";

function loadFromStorage() {
  try {
    vehicles = JSON.parse(localStorage.getItem(KEY_V) || "[]");
    incidents = JSON.parse(localStorage.getItem(KEY_I) || "[]");
    if (!Array.isArray(vehicles)) vehicles = [];
    if (!Array.isArray(incidents)) incidents = [];

    vehicleId = vehicles.reduce((m, v) => Math.max(m, v.id || 0), 0) + 1;
    incidentId = incidents.reduce((m, i) => Math.max(m, i.id || 0), 0) + 1;
  } catch (e) {
    vehicles = [];
    incidents = [];
    vehicleId = 1;
    incidentId = 1;
  }
}

function saveToStorage() {
  localStorage.setItem(KEY_V, JSON.stringify(vehicles));
  localStorage.setItem(KEY_I, JSON.stringify(incidents));
}

function formatDuration(ms) {
  const totalSec = Math.floor(ms / 1000);
  const h = Math.floor(totalSec / 3600);
  const m = Math.floor((totalSec % 3600) / 60);
  const s = totalSec % 60;
  const pad = (x) => String(x).padStart(2, "0");
  if (h > 0) return `${h}:${pad(m)}:${pad(s)} h`;
  return `${m}:${pad(s)} min`;
}

/* Fahrzeugstatus automatisch aus Einsätzen ableiten */
function recalcVehicleStatusFromIncidents() {
  vehicles.forEach(v => {
    if (v.status === "außer Dienst") return;
    const hasActive = incidents.some(i =>
      Array.isArray(i.vehicle_ids) &&
      i.vehicle_ids.includes(v.id) &&
      i.status !== "erledigt"
    );
    if (!hasActive && v.status === "Einsatz") {
      v.status = "frei";
    }
  });
}

/* Fahrzeuge */

function addVehicle() {
  const nameInput = document.getElementById("fahrzeugName");
  const typeInput = document.getElementById("fahrzeugTyp");
  const ovInput   = document.getElementById("fahrzeugOv");

  const name = nameInput.value.trim();
  const type = typeInput.value.trim();
  const ov   = ovInput.value;

  if (!name || !type) {
    alert("Bitte Funkname und Typ eingeben!");
    return;
  }

  vehicles.push({
    id: vehicleId++,
    name,
    type,
    ov,
    status: "frei"
  });

  nameInput.value = "";
  typeInput.value = "";
  ovInput.value = "Recklinghausen";

  saveToStorage();
  renderAll();
}

function reportFreeRTWs() {
  const freeRtws = vehicles.filter(
    v => v.status === "frei" && v.type.toUpperCase().includes("RTW")
  );
  if (freeRtws.length === 0) {
    alert("Es sind aktuell keine freien RTWs im Kreis eingetragen.");
    return;
  }
  const text = freeRtws.map(v => `${v.name} (${v.ov})`).join("\n");
  alert("Freie RTWs im Kreis:\n\n" + text);
}

function renderVehicles() {
  const tbody = document.getElementById("vehicleTableBody");
  tbody.innerHTML = "";

  const ovFilter = document.getElementById("filterOvSel").value;
  const typeFilter = document.getElementById("filterTypeSel").value;

  let list = vehicles.slice();
  if (ovFilter) {
    list = list.filter(v => v.ov === ovFilter);
  }
  if (typeFilter) {
    list = list.filter(v => v.type.toUpperCase() === typeFilter.toUpperCase());
  }

  list.forEach(v => {
    const tr = document.createElement("tr");
    const badgeClass =
      v.status === "frei"
        ? "status-frei"
        : v.status === "Einsatz"
        ? "status-einsatz"
        : "status-ausserdienst";

    tr.innerHTML = `
      <td>${v.id}</td>
      <td>${v.name}</td>
      <td>${v.type}</td>
      <td>${v.ov || ""}</td>
      <td><span class="status-badge ${badgeClass}">${v.status}</span></td>
      <td>
        <select data-v-id="${v.id}">
          <option value="frei"${v.status==="frei"?" selected":""}>frei</option>
          <option value="Einsatz"${v.status==="Einsatz"?" selected":""}>Einsatz</option>
          <option value="außer Dienst"${v.status==="außer Dienst"?" selected":""}>außer Dienst</option>
        </select>
      </td>
    `;
    tbody.appendChild(tr);
  });

  tbody.querySelectorAll("select[data-v-id]").forEach(sel => {
    sel.addEventListener("change", () => {
      const id = Number(sel.getAttribute("data-v-id"));
      const v = vehicles.find(x => x.id === id);
      if (!v) return;
      v.status = sel.value;
      saveToStorage();
      renderAll(); // damit auch "Fahrzeuge im Einsatz" & Statistik aktualisiert werden
    });
  });
}

/* Fahrzeuge im Einsatz */

function renderActiveVehicles() {
  const tbody = document.getElementById("activeVehicleTableBody");
  tbody.innerHTML = "";

  const active = vehicles.filter(v => v.status === "Einsatz");

  active.forEach(v => {
    const tr = document.createElement("tr");
    tr.innerHTML = `
      <td>${v.id}</td>
      <td>${v.name}</td>
      <td>${v.type}</td>
      <td>${v.ov || ""}</td>
      <td><span class="status-badge status-einsatz">${v.status}</span></td>
    `;
    tbody.appendChild(tr);
  });

  if (active.length === 0) {
    const tr = document.createElement("tr");
    tr.innerHTML = `<td colspan="5" class="small">Derzeit keine Fahrzeuge im Status „Einsatz“.</td>`;
    tbody.appendChild(tr);
  }
}

/* Einsätze */

function addIncident() {
  const descInput = document.getElementById("einsatzBeschreibung");
  const prioInput = document.getElementById("einsatzPrio");

  const description = descInput.value.trim();
  const priority = prioInput.value;

  if (!description) {
    alert("Bitte eine Einsatzbeschreibung eingeben!");
    return;
  }

  incidents.push({
    id: incidentId++,
    description,
    priority,
    status: "offen",
    vehicle_ids: [],
    createdAt: Date.now(),
    closedAt: null
  });

  descInput.value = "";
  prioInput.value = "mittel";

  saveToStorage();
  renderAll();
}

function renderIncidents() {
  const tbody = document.getElementById("incidentTableBody");
  tbody.innerHTML = "";

  incidents.forEach(i => {
    const tr = document.createElement("tr");
    const vehiclesStr = i.vehicle_ids && i.vehicle_ids.length
      ? i.vehicle_ids.join(", ")
      : "–";

    tr.innerHTML = `
      <td>${i.id}</td>
      <td>${i.description}</td>
      <td>${i.priority}</td>
      <td>${vehiclesStr}</td>
      <td>${i.status}</td>
      <td>
        <select data-i-id="${i.id}">
          <option value="offen"${i.status==="offen"?" selected":""}>offen</option>
          <option value="zugewiesen"${i.status==="zugewiesen"?" selected":""}>zugewiesen</option>
          <option value="unterwegs"${i.status==="unterwegs"?" selected":""}>unterwegs</option>
          <option value="erledigt"${i.status==="erledigt"?" selected":""}>erledigt</option>
        </select>
      </td>
      <td><span class="incident-duration" data-i-id="${i.id}"></span></td>
    `;
    tbody.appendChild(tr);
  });

  tbody.querySelectorAll("select[data-i-id]").forEach(sel => {
    sel.addEventListener("change", () => {
      const id = Number(sel.getAttribute("data-i-id"));
      const i = incidents.find(x => x.id === id);
      if (!i) return;
      const newStatus = sel.value;

      if (newStatus === "erledigt") {
        if (!i.closedAt) i.closedAt = Date.now();
      } else {
        i.closedAt = null;
      }
      i.status = newStatus;

      recalcVehicleStatusFromIncidents();
      saveToStorage();
      renderAll();
    });
  });

  updateIncidentTimers();
}

function renderHistory() {
  const tbody = document.getElementById("historyTableBody");
  tbody.innerHTML = "";

  const finished = incidents
    .filter(i => i.status === "erledigt")
    .slice()
    .sort((a,b) => (b.closedAt || 0) - (a.closedAt || 0));

  finished.forEach(i => {
    const tr = document.createElement("tr");
    const start = i.createdAt ? new Date(i.createdAt) : null;
    const end   = i.closedAt ? new Date(i.closedAt) : null;
    const durMs = (i.closedAt || Date.now()) - (i.createdAt || Date.now());

    tr.innerHTML = `
      <td>${i.id}</td>
      <td>${i.description}</td>
      <td>${i.priority}</td>
      <td>${formatDuration(durMs)}</td>
      <td>${start ? start.toLocaleString() : "–"}</td>
      <td>${end ? end.toLocaleString() : "–"}</td>
    `;
    tbody.appendChild(tr);
  });
}

function updateIncidentTimers() {
  const now = Date.now();
  incidents.forEach(i => {
    const span = document.querySelector(
      '.incident-duration[data-i-id="' + i.id + '"]'
    );
    if (!span) return;
    const start = i.createdAt || now;
    const end   = i.closedAt || now;
    span.textContent = formatDuration(end - start);
  });
}

/* Zuordnung */

function renderAssignIncidentSelect() {
  const sel = document.getElementById("assignIncidentSelect");
  sel.innerHTML = "<option value=''>Einsatz wählen</option>";
  incidents.forEach(i => {
    sel.innerHTML += "<option value='" + i.id + "'>#"
      + i.id + " – " + i.description + "</option>";
  });
}

function renderAssignVehicleSelect() {
  const sel = document.getElementById("assignVehicleSelect");
  sel.innerHTML = "<option value=''>Fahrzeug wählen</option>";
  vehicles.forEach(v => {
    sel.innerHTML += "<option value='" + v.id + "'>#"
      + v.id + " – " + v.name + " (" + (v.ov || "") + ")</option>";
  });
}

function assignVehicle() {
  const incId = Number(document.getElementById("assignIncidentSelect").value);
  const vehId = Number(document.getElementById("assignVehicleSelect").value);

  if (!incId || !vehId) {
    alert("Bitte Einsatz und Fahrzeug wählen!");
    return;
  }

  const inc = incidents.find(i => i.id === incId);
  const veh = vehicles.find(v => v.id === vehId);
  if (!inc || !veh) {
    alert("Einsatz oder Fahrzeug nicht gefunden.");
    return;
  }

  if (!Array.isArray(inc.vehicle_ids)) inc.vehicle_ids = [];
  if (!inc.vehicle_ids.includes(vehId)) {
    inc.vehicle_ids.push(vehId);
  }

  veh.status = "Einsatz";
  if (inc.status === "offen") inc.status = "zugewiesen";

  saveToStorage();
  renderAll();
}

/* Statistik */

function renderStats() {
  const el = document.getElementById("statsContent");
  const totalV = vehicles.length;
  const vFrei   = vehicles.filter(v => v.status === "frei").length;
  const vEinsatz= vehicles.filter(v => v.status === "Einsatz").length;
  const vAusser = vehicles.filter(v => v.status === "außer Dienst").length;

  const totalI = incidents.length;
  const iOffen  = incidents.filter(i => i.status === "offen").length;
  const iZugw   = incidents.filter(i => i.status === "zugewiesen").length;
  const iUnterw = incidents.filter(i => i.status === "unterwegs").length;
  const iErled  = incidents.filter(i => i.status === "erledigt").length;

  el.innerHTML = `
    <div class="stat-box">
      <div class="stat-label">Fahrzeuge gesamt</div>
      <div class="stat-value">${totalV}</div>
      <div class="small">frei: ${vFrei}, Einsatz: ${vEinsatz}, a.D.: ${vAusser}</div>
    </div>
    <div class="stat-box">
      <div class="stat-label">Einsätze gesamt</div>
      <div class="stat-value">${totalI}</div>
      <div class="small">
        offen: ${iOffen}, zugewiesen: ${iZugw}, unterwegs: ${iUnterw}, erledigt: ${iErled}
      </div>
    </div>
  `;
}

/* Gesamtrendering & Init */

function renderAll() {
  recalcVehicleStatusFromIncidents();
  renderVehicles();
  renderActiveVehicles();
  renderIncidents();
  renderHistory();
  renderAssignIncidentSelect();
  renderAssignVehicleSelect();
  renderStats();
}

loadFromStorage();
renderAll();
setInterval(updateIncidentTimers, 1000);

document.getElementById("filterOvSel").addEventListener("change", renderVehicles);
document.getElementById("filterTypeSel").addEventListener("change", renderVehicles);
</script>

</body>
</html>
