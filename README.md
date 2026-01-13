<!doctype html>
<html lang="nl">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Bus 341 – Wijk bij Duurstede ⇄ Utrecht CS</title>

<style>
body {
  font-family: system-ui, Arial, sans-serif;
  margin: 20px;
  background: #f7f7f7;
}
h1 { margin-bottom: 4px; }
h2 { margin-top: 32px; }
.meta { color:#555; margin-bottom:16px; }

table {
  width:100%;
  border-collapse:collapse;
  background:#fff;
  box-shadow:0 2px 6px rgba(0,0,0,.1);
}
th, td {
  padding:10px;
  border-bottom:1px solid #e0e0e0;
  text-align:left;
}
th {
  font-size:12px;
  text-transform:uppercase;
  color:#444;
}
.badge {
  border:1px solid #ccc;
  border-radius:999px;
  padding:2px 8px;
  font-size:12px;
}
.muted { color:#777; }
</style>
</head>

<body>

<h1>Bus 341</h1>
<div class="meta">Realtime – automatisch bijgewerkt – vanaf nu</div>

<h2>De Geer (Wijk bij Duurstede) → Utrecht CS</h2>
<table>
<thead>
<tr><th>Vertrek</th><th>Over</th><th>Bestemming</th></tr>
</thead>
<tbody id="geer">
<tr><td colspan="3" class="muted">Laden…</td></tr>
</tbody>
</table>

<h2>Utrecht CS → Busstation Wijk bij Duurstede</h2>
<table>
<thead>
<tr><th>Vertrek</th><th>Over</th><th>Bestemming</th></tr>
</thead>
<tbody id="utrecht">
<tr><td colspan="3" class="muted">Laden…</td></tr>
</tbody>
</table>

<script>
/* ================================
   VUL HIER DE JUISTE TPC’S IN
   ================================ */

// De Geer – richting Utrecht
const TPC_GEER = "XXXXXXXX";

// Utrecht CS – richting Wijk bij Duurstede
const TPC_UTRECHT = "YYYYYYYY";

/* ================================ */

const LINE = "341";
const REFRESH = 30000;

function fmt(t){return new Date(t).toLocaleTimeString("nl-NL",{hour:"2-digit",minute:"2-digit"})}
function mins(t){return Math.round((new Date(t)-Date.now())/60000)}
function dep(p){return p.ExpectedDepartureTime||p.TargetDepartureTime}

async function load(tpc, target, destMatch){
  const r = await fetch(`https://v0.ovapi.nl/tpc/${tpc}/departures`);
  const j = await r.json();
  const list = Object.values(j[tpc].Passes||{})
    .filter(p =>
      p.LinePublicNumber === LINE &&
      p.DestinationName50?.includes(destMatch) &&
      new Date(dep(p)) >= new Date()
    )
    .sort((a,b)=>new Date(dep(a))-new Date(dep(b)));

  const tb = document.getElementById(target);
  tb.innerHTML = "";

  if(!list.length){
    tb.innerHTML = `<tr><td colspan="3" class="muted">Geen ritten</td></tr>`;
    return;
  }

  for(const p of list){
    const m = mins(dep(p));
    tb.innerHTML += `
      <tr>
        <td>${fmt(dep(p))}</td>
        <td><span class="badge">${m<=0?"nu":m+" min"}</span></td>
        <td>${p.DestinationName50}</td>
      </tr>`;
  }
}

function refresh(){
  load(TPC_GEER,"geer","Utrecht");
  load(TPC_UTRECHT,"utrecht","Wijk bij Duurstede");
}

refresh();
setInterval(refresh, REFRESH);
</script>

</body>
</html>
