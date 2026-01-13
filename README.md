<!doctype html>
<html lang="nl">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Bus 341 → Wijk bij Duurstede (vanaf nu)</title>
  <style>
    :root { font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif; }
    body { margin: 16px; line-height: 1.35; }
    header { display: grid; gap: 10px; margin-bottom: 12px; }
    .row { display: flex; flex-wrap: wrap; gap: 10px; align-items: end; }
    label { display: grid; gap: 6px; font-size: 14px; }
    input, button { font-size: 16px; padding: 10px 12px; }
    input { width: 280px; max-width: 100%; }
    button { cursor: pointer; }
    .meta { color: #555; font-size: 13px; }
    .error { color: #b00020; }
    table { width: 100%; border-collapse: collapse; margin-top: 10px; }
    th, td { text-align: left; padding: 10px 8px; border-bottom: 1px solid #e5e5e5; vertical-align: top; }
    th { font-size: 13px; color: #333; text-transform: uppercase; letter-spacing: .04em; }
    .pill { display: inline-block; padding: 2px 8px; border: 1px solid #ddd; border-radius: 999px; font-size: 12px; color: #333; }
    .muted { color: #666; font-size: 13px; }
  </style>
</head>
<body>
  <header>
    <h1 style="margin:0;">Bus 341 → Wijk bij Duurstede</h1>
    <div class="meta">
      Toont alle komende ritten <b>vanaf nu</b> voor lijn <b>341</b> met bestemming die “Wijk bij Duurstede” bevat.
    </div>

    <div class="row">
      <label>
        TPC (halte/perron-code)
        <input id="tpc" inputmode="numeric" placeholder="bijv. 30001953" />
      </label>

      <label>
        (Optioneel) CORS-proxy URL
        <input id="proxy" placeholder="bijv. https://jouwdomein.nl/proxy?url=" />
      </label>

      <button id="load">Laden</button>
      <button id="autorefresh">Auto-refresh: aan</button>
    </div>

    <div id="status" class="meta"></div>
  </header>

  <main>
    <table id="tbl" aria-label="Vertrekken">
      <thead>
        <tr>
          <th>Vertrek</th>
          <th>In</th>
          <th>Lijn</th>
          <th>Bestemming</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody id="rows">
        <tr><td colspan="5" class="muted">Vul een TPC in en klik op “Laden”.</td></tr>
      </tbody>
    </table>

    <p class="muted" style="margin-top:14px;">
      TPC nodig? Je kunt die opzoeken via ovzoeker.nl (klik op een halte) of een TPC-finder. :contentReference[oaicite:2]{index=2}
      <br/>
      Let op: als je browser een CORS-fout geeft, gebruik dan een (eigen) proxy in het veld hierboven.
    </p>
  </main>

<script>
(() => {
  const LINE = "341";
  const DEST_SUBSTR = "Wijk bij Duurstede";
  const REFRESH_MS = 30_000;

  const els = {
    tpc: document.getElementById("tpc"),
    proxy: document.getElementById("proxy"),
    load: document.getElementById("load"),
    autorefresh: document.getElementById("autorefresh"),
    status: document.getElementById("status"),
    rows: document.getElementById("rows")
  };

  let timer = null;
  let auto = true;

  function fmtTime(iso) {
    if (!iso) return "—";
    const d = new Date(iso);
    if (Number.isNaN(d.getTime())) return iso;
    return d.toLocaleTimeString("nl-NL", { hour: "2-digit", minute: "2-digit" });
  }

  function minutesUntil(iso) {
    if (!iso) return null;
    const t = new Date(iso).getTime();
    const now = Date.now();
    if (Number.isNaN(t)) return null;
    return Math.round((t - now) / 60000);
  }

  function pickDepartureTime(p) {
    // OVAPI gebruikt o.a. TargetDepartureTime / ExpectedDepartureTime / ExpectedArrivalTime in verschillende feeds.
    return p.ExpectedDepartureTime || p.ExpectedArrivalTime || p.TargetDepartureTime || p.TargetArrivalTime || null;
  }

  function normalizePasses(jsonForTpc) {
    // Verwacht structuur: { "<TPC>": { Stop: {...}, Passes: { "<id>": {...}, ... } } }
    if (!jsonForTpc || typeof jsonForTpc !== "object") return [];
    const passesObj = jsonForTpc.Passes || {};
    return Object.values(passesObj).filter(v => v && typeof v === "object");
  }

  function lineNumber(p) {
    // Probeer meerdere mogelijke velden
    return (p.LinePublicNumber || p.LinePlanningNumber || p.LineNumber || "").toString();
  }

  function destination(p) {
    return (p.DestinationName50 || p.DestinationName || p.Destination || "").toString();
  }

  function statusText(p) {
    // Soms TripStopStatus aanwezig (bijv. ARRIVED, CANCEL, PASS)
    return (p.TripStopStatus || p.JourneyStopType || p.TransportType || "").toString();
  }

  function render(list) {
    els.rows.innerHTML = "";
    if (!list.length) {
      els.rows.innerHTML = `<tr><td colspan="5" class="muted">Geen komende ritten gevonden voor 341 → ${DEST_SUBSTR}.</td></tr>`;
      return;
    }

    for (const p of list) {
      const depIso = pickDepartureTime(p);
      const mins = minutesUntil(depIso);
      const minsLabel = (mins === null) ? "—" : (mins <= 0 ? "nu" : `${mins} min`);
      const st = statusText(p) || "—";

      const tr = document.createElement("tr");
      tr.innerHTML = `
        <td>${fmtTime(depIso)}</td>
        <td><span class="pill">${minsLabel}</span></td>
        <td>${lineNumber(p) || "—"}</td>
        <td>${destination(p) || "—"}</td>
        <td>${st}</td>
      `;
      els.rows.appendChild(tr);
    }
  }

  async function load() {
    const tpc = (els.tpc.value || "").trim();
    if (!/^\d{6,10}$/.test(tpc)) {
      els.status.innerHTML = `<span class="error">Vul een geldige TPC in (meestal 8 cijfers).</span>`;
      render([]);
      return;
    }

    const baseUrl = `https://v0.ovapi.nl/tpc/${encodeURIComponent(tpc)}/departures`;
    const proxy = (els.proxy.value || "").trim();
    const url = proxy ? (proxy + encodeURIComponent(baseUrl)) : baseUrl;

    els.status.textContent = "Ophalen…";
    try {
      const res = await fetch(url, { cache: "no-store" });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const data = await res.json();

      const node = data[tpc];
      const passes = normalizePasses(node);

      // Filter: lijn 341 + bestemming bevat "Wijk bij Duurstede" + vanaf nu
      const now = Date.now();
      const filtered = passes
        .filter(p => lineNumber(p) === LINE)
        .filter(p => destination(p).toLowerCase().includes(DEST_SUBSTR.toLowerCase()))
        .map(p => ({ p, dep: pickDepartureTime(p) }))
        .filter(x => x.dep && new Date(x.dep).getTime() >= (now - 30_000)) // kleine marge
        .sort((a, b) => new Date(a.dep) - new Date(b.dep))
        .map(x => x.p);

      const stopName = node?.Stop?.TimingPointName ? `${node.Stop.TimingPointName} (${node.Stop.TimingPointTown || ""})` : `TPC ${tpc}`;
      els.status.textContent = `Laatste update: ${new Date().toLocaleTimeString("nl-NL", {hour:"2-digit", minute:"2-digit", second:"2-digit"})} — ${stopName}`;

      render(filtered);
    } catch (e) {
      els.status.innerHTML = `<span class="error">Kon data niet ophalen. Tip: als dit een CORS-fout is, gebruik een proxy. (${String(e.message || e)})</span>`;
      render([]);
    }
  }

  function setAuto(on) {
    auto = on;
    els.autorefresh.textContent = `Auto-refresh: ${auto ? "aan" : "uit"}`;
    if (timer) clearInterval(timer);
    timer = auto ? setInterval(load, REFRESH_MS) : null;
  }

  els.load.addEventListener("click", load);
  els.autorefresh.addEventListener("click", () => setAuto(!auto));

  // handig: Enter in TPC veld
  els.tpc.addEventListener("keydown", (e) => {
    if (e.key === "Enter") load();
  });

  setAuto(true);
})();
</script>
</body>
</html>
