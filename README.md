<!DOCTYPE html>
<html lang="nl">
<head>
  <meta charset="UTF-8" />
  <title>Camera Grid - Skiien</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    html, body {
      height: 100%;
    }

    body {
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      background: #020617;
      color: #e5e7eb;
      display: flex;
      flex-direction: column;
    }

    header {
      background: #0f172a;
      border-bottom: 1px solid rgba(148, 163, 184, 0.4);
      padding: 10px 16px;
      display: flex;
      align-items: center;
      gap: 16px;
    }

    .logo {
      font-weight: 700;
      letter-spacing: 0.03em;
    }

    nav {
      display: flex;
      gap: 8px;
      margin-left: auto;
    }

    .menu-btn {
      padding: 6px 14px;
      border-radius: 999px;
      border: 1px solid transparent;
      background: #1e293b;
      color: #e5e7eb;
      font-size: 14px;
      cursor: pointer;
      transition: 0.15s;
    }

    .menu-btn:hover {
      background: #111827;
      border-color: rgba(148, 163, 184, 0.6);
    }

    .menu-btn.active {
      background: #22c55e;
      color: #022c22;
      border-color: #22c55e;
    }

    main {
      flex: 1;
      display: flex;
      min-height: 0; /* important for flex layouts */
    }

    .grid-container {
      flex: 1;
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      grid-template-rows: repeat(2, 1fr);
      gap: 2px;
      background: #020617;
    }

    .cam-cell {
      position: relative;
      background: #020617;
    }

    iframe {
      width: 100%;
      height: 100%;
      border: none;
      display: block;
    }

    /* optie: responsief bij smalle schermen (1 kolom) */
    @media (max-width: 768px) {
      .grid-container {
        grid-template-columns: 1fr;
        grid-template-rows: repeat(4, 1fr);
      }
    }
  </style>
</head>
<body>
  <header>
    <div class="logo">Camera Grid</div>
    <nav>
      <!-- Nu alleen Skiien, later kun je hier meer knoppen toevoegen -->
      <button class="menu-btn active" data-section="skiien">Skiien</button>
    </nav>
  </header>

  <main>
    <!-- Sectie: Skiien -->
    <section id="skiien" class="grid-container">
      <div class="cam-cell">
        <iframe
          src="https://www.feratel.com/en/webcams/austria/tyrol/finkenberg-penkenjoch"
          loading="lazy"
          allowfullscreen
        ></iframe>
      </div>
      <div class="cam-cell">
        <iframe
          src="https://www.feratel.com/nl/webcams/oostenrijk/tirol/mayrhofen-penkenbahn"
          loading="lazy"
          allowfullscreen
        ></iframe>
      </div>
      <div class="cam-cell">
        <iframe
          src="https://www.feratel.com/en/webcams/austria/tyrol/hintertux-gefrorene-wand"
          loading="lazy"
          allowfullscreen
        ></iframe>
      </div>
      <div class="cam-cell">
        <iframe
          src="https://www.feratel.com/en/webcams/austria/tyrol/hintertux-gefrorene-wand"
          loading="lazy"
          allowfullscreen
        ></iframe>
      </div>
    </section>
  </main>

  <script>
    // Klaar voor meerdere menu-items / sections in de toekomst.
    const buttons = document.querySelectorAll(".menu-btn");
    const sections = document.querySelectorAll("main > section");

    buttons.forEach(btn => {
      btn.addEventListener("click", () => {
        buttons.forEach(b => b.classList.remove("active"));
        btn.classList.add("active");

        const target = btn.dataset.section;
        sections.forEach(sec => {
          sec.style.display = (sec.id === target) ? "grid" : "none";
        });
      });
    });
  </script>
</body>
</html>
