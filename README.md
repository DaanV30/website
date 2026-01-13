<!doctype html>
<html lang="nl">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Bus 341 – Wijk bij Duurstede ⇄ Utrecht CS</title>

  <style>
    body {
      font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
      margin: 16px;
      background: #fafafa;
    }
    h1 { margin-bottom: 4px; }
    h2 { margin-top: 32px; margin-bottom: 8px; }
    .meta { color: #555; font-size: 14px; }

    table {
      width: 100%;
      border-collapse: collapse;
      background: #fff;
      box-shadow: 0 1px 3px rgba(0,0,0,.08);
    }
    th, td {
      padding: 10px 8px;
      border-bottom: 1px solid #e5e5e5;
      text-align: left;
    }
    th {
      font-size: 12px;
      text-transform: uppercase;
      color: #444;
    }
    .pill {
      display: inline-block;
      padding: 2px 8px;
      border-radius: 999px;
      border: 1px solid #ddd;
      font-size: 12px;
    }
    .muted { color: #777; }
  </style>
</head>

<body>
  <h1>Bus 341</h1>
  <div class="meta">Automatisch bijgewerkt – toont alleen ritten vanaf nu</div>

  <!-- SECTIE 1 -->
  <h2>De Geer → Utrecht CS</h2>
  <table>
    <thead>
      <tr>
        <th>Vertrek</th>
        <th>Over</th>
        <th>Bestemming</th>
      </tr>
    </thead>
    <tbody id="rows-geer">
      <tr><td colspan="3" class="muted">Laden…</td></tr>
    </tbody>
  </table>

  <!-- SECTIE 2 -->
  <h2>Utrecht CS → Busstation Wijk bij Duurstede</h2>
  <table>
    <thead>
      <tr>
        <th>Vertrek</th>
        <th>Over</th>
        <th>Bestemming</th>
      </tr>
    </thead>
    <tbody id="rows-utrecht">
      <tr><td colspan="3" class="muted">Laden…</td></tr>
    </tbody>
  </table>

<script>
/* =========================
   INSTELLINGEN
   ========================= */

// ⬇️ VUL HIER DE JUISTE TPC-CODES IN ⬇️
const TPC_GEER = "XXXXXXXX";      // De Geer – richting Utrecht
const TPC_UTRECHT = "YYYYYYYY";   // Utrecht CS – richting Wijk bij Duurstede

const LINE = "341";
const REFRESH_MS = 30000;

/* ========================= */

function fmtTime(iso) {
  const d = new Date(iso);
  return d.toLocaleTimeString("nl-NL", { hour: "2-digit", minute: "2-digit" });
}

function minutesUntil(iso) {
  return Math.round((new Date(iso) - Date.now()) / 60000);
}

function pickTime(p) {
  return p.ExpectedDepartureTime || p.TargetDepartureTime || null;
}

function render(tbodyId, passes, destFilter) {
  const tbody = document.getElementById(tbodyId);
  tbody.innerHTML = "";

  if (!passes.length) {
    tbody.innerHTML =
      `<tr><td colspan="3" class="muted">Geen komende ritten</td></tr>`;
    return;
  }

  for (const p of passes) {
    const iso = pickTime(p);
    const mins = minutesUntil(iso);
    const tr = document.createElement("tr");
    tr.innerHTML = `
      <td>${fmtTime(iso)}</td>
      <td><span class="pill">${mins <= 0 ? "nu" : mins + " min"}</span></td>
      <td>${p.DestinationName50 || destFilter}</td>
    `;
    tbody.appendChild(tr);
  }
}

async function loadTPC(tpc, tbodyId, destFilter) {
  const res = await fetch(`https://v0.ovapi.nl/tpc/${tpc}/departures`);
  const data = await res.json();
  const passes = Object.values(data[tpc].Passes || {})
    .filter(p =>
      p.LinePublicNumber === LINE &&
      (p.DestinationName50 || "").includes(destFilter) &&
      new Date(pickTime(p)) >= new Date()
    )
    .sort((a, b) => new Date(pickTime(a)) - new Date(pickTime(b)));

  render(tbodyId, passes, destFilter);
}

async function loadAll() {
  loadTPC(TPC_GEER, "rows-geer", "Utrecht");
  loadTPC(TPC_UTRECHT, "rows-utrecht", "Wijk bij Duurstede");
}

loadAll();
setInterval(loadAll, REFRESH_MS);
</script>
</body>
</html>
