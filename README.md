<!DOCTYPE html>
<html lang="nl">
<head>
  <meta charset="UTF-8" />
  <title>3D Walk Game (WASD)</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    }

    body {
      background: #020617;
      color: #e5e7eb;
      overflow: hidden;
    }

    #ui {
      position: fixed;
      inset: 0;
      display: flex;
      align-items: center;
      justify-content: center;
      pointer-events: none;
    }

    .panel {
      background: rgba(15, 23, 42, 0.92);
      border-radius: 18px;
      padding: 18px 22px;
      box-shadow: 0 20px 40px rgba(0,0,0,0.7);
      max-width: 320px;
      text-align: center;
      pointer-events: auto;
      border: 1px solid rgba(148, 163, 184, 0.4);
    }

    h1 {
      font-size: 20px;
      margin-bottom: 6px;
    }

    p {
      font-size: 13px;
      line-height: 1.5;
      margin-bottom: 6px;
    }

    .keys {
      display: inline-flex;
      gap: 4px;
      margin: 6px 0;
    }

    .key {
      border-radius: 6px;
      border: 1px solid rgba(148, 163, 184, 0.7);
      padding: 2px 7px;
      font-size: 12px;
      background: rgba(15, 23, 42, 0.9);
    }

    .btn {
      margin-top: 10px;
      display: inline-block;
      background: #22c55e;
      color: #022c22;
      font-weight: 600;
      padding: 7px 16px;
      border-radius: 999px;
      border: none;
      font-size: 13px;
      cursor: pointer;
      transition: transform 0.12s ease, box-shadow 0.12s ease, background 0.12s ease;
    }

    .btn:hover {
      transform: translateY(-1px);
      box-shadow: 0 8px 18px rgba(34, 197, 94, 0.4);
      background: #16a34a;
    }

    .hint {
      opacity: 0.8;
      font-size: 12px;
      margin-top: 4px;
    }

    #crosshair {
      position: fixed;
      top: 50%;
      left: 50%;
      width: 12px;
      height: 12px;
      transform: translate(-50%, -50%);
      pointer-events: none;
      opacity: 0;
    }

    #crosshair::before,
    #crosshair::after {
      content: "";
      position: absolute;
      background: rgba(248, 250, 252, 0.9);
      border-radius: 99px;
    }

    #crosshair::before {
      width: 10px;
      height: 2px;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }

    #crosshair::after {
      width: 2px;
      height: 10px;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }
  </style>
</head>
<body>
  <div id="ui">
    <div class="panel" id="instructions">
      <h1>3D Walk Game</h1>
      <p>Loop rond in een 3D-wereld.</p>
      <div class="keys">
        <div class="key">W</div>
        <div class="key">A</div>
        <div class="key">S</div>
        <div class="key">D</div>
      </div>
      <p>Gebruik de muis om rond te kijken.</p>
      <button class="btn" id="startBtn">Klik om te starten</button>
      <div class="hint">
        Tip: beweeg je muis en houd <span class="key">Shift</span> ingedrukt om sneller te lopen.
      </div>
    </div>
  </div>

  <div id="crosshair"></div>

  <!-- Three.js + PointerLockControls via CDN -->
  <script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.160.0/examples/js/controls/PointerLockControls.js"></script>

  <script>
    let camera, scene, renderer;
    let controls;
    let moveForward = false;
    let moveBackward = false;
    let moveLeft = false;
    let moveRight = false;
    let canJump = false;

    const velocity = new THREE.Vector3();
    const direction = new THREE.Vector3();
    let prevTime = performance.now();

    const clock = new THREE.Clock();

    init();
    animate();

    function init() {
      // Renderer
      renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setPixelRatio(window.devicePixelRatio);
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.setClearColor(0x020617);
      document.body.appendChild(renderer.domElement);

      // Scene
      scene = new THREE.Scene();
      scene.fog = new THREE.Fog(0x020617, 10, 120);

      // Camera
      camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 200);
      camera.position.set(0, 2, 5);

      // Licht
      const hemiLight = new THREE.HemisphereLight(0xbfd4ff, 0x0f172a, 0.7);
      scene.add(hemiLight);

      const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
      dirLight.position.set(10, 20, 5);
      scene.add(dirLight);

      // Controls (pointer lock)
      controls = new THREE.PointerLockControls(camera, document.body);

      const instructions = document.getElementById("instructions");
      const startBtn = document.getElementById("startBtn");
      const ui = document.getElementById("ui");
      const crosshair = document.getElementById("crosshair");

      function lockPointer() {
        controls.lock();
      }

      startBtn.addEventListener("click", lockPointer);

      controls.addEventListener("lock", () => {
        ui.style.display = "none";
        crosshair.style.opacity = "1";
      });

      controls.addEventListener("unlock", () => {
        ui.style.display = "flex";
        crosshair.style.opacity = "0";
      });

      scene.add(controls.getObject());

      // Bodem
      const floorGeo = new THREE.PlaneGeometry(400, 400);
      const floorMat = new THREE.MeshStandardMaterial({
        color: 0x0f172a,
        roughness: 0.9,
        metalness: 0.0
      });
      const floor = new THREE.Mesh(floorGeo, floorMat);
      floor.rotation.x = -Math.PI / 2;
      floor.receiveShadow = true;
      scene.add(floor);

      // Een paar "gebouwen" / blokken om rond te lopen
      const boxGeo = new THREE.BoxGeometry(3, 6, 3);
      const boxMat1 = new THREE.MeshStandardMaterial({ color: 0x38bdf8 });
      const boxMat2 = new THREE.MeshStandardMaterial({ color: 0xf97316 });
      const boxMat3 = new THREE.MeshStandardMaterial({ color: 0x22c55e });

      function addBox(x, z, mat) {
        const m = new THREE.Mesh(boxGeo, mat);
        m.position.set(x, 3, z);
        m.castShadow = true;
        m.receiveShadow = true;
        scene.add(m);
      }

      addBox(0, -15, boxMat1);
      addBox(15, -5, boxMat2);
      addBox(-18, 8, boxMat3);
      addBox(10, 20, boxMat1);
      addBox(-12, -22, boxMat2);
      addBox(20, 5, boxMat3);

      // Kleine random blokjes
      const smallGeo = new THREE.BoxGeometry(1.6, 1.6, 1.6);
      const smallMat = new THREE.MeshStandardMaterial({ color: 0x64748b });

      for (let i = 0; i < 40; i++) {
        const m = new THREE.Mesh(smallGeo, smallMat);
        m.position.set(
          (Math.random() - 0.5) * 80,
          0.8,
          (Math.random() - 0.5) * 80
        );
        scene.add(m);
      }

      // Sterren / hemel sfeer
      const starGeo = new THREE.BufferGeometry();
      const starCount = 400;
      const positions = new Float32Array(starCount * 3);
      for (let i = 0; i < starCount; i++) {
        const r = 80 + Math.random() * 60;
        const angle = Math.random() * Math.PI * 2;
        const y = 30 + Math.random() * 40;
        positions[i * 3] = Math.cos(angle) * r;
        positions[i * 3 + 1] = y;
        positions[i * 3 + 2] = Math.sin(angle) * r;
      }
      starGeo.setAttribute("position", new THREE.BufferAttribute(positions, 3));
      const starMat = new THREE.PointsMaterial({
        size: 0.6,
        color: 0xffffff
      });
      const stars = new THREE.Points(starGeo, starMat);
      scene.add(stars);

      // Keyboard events
      const onKeyDown = function (event) {
        switch (event.code) {
          case "KeyW":
            moveForward = true;
            break;
          case "KeyA":
            moveLeft = true;
            break;
          case "KeyS":
            moveBackward = true;
            break;
          case "KeyD":
            moveRight = true;
            break;
          case "Space":
            if (canJump === true) velocity.y += 7;
            canJump = false;
            break;
        }
      };

      const onKeyUp = function (event) {
        switch (event.code) {
          case "KeyW":
            moveForward = false;
            break;
          case "KeyA":
            moveLeft = false;
            break;
          case "KeyS":
            moveBackward = false;
            break;
          case "KeyD":
            moveRight = false;
            break;
        }
      };

      document.addEventListener("keydown", onKeyDown);
      document.addEventListener("keyup", onKeyUp);

      window.addEventListener("resize", onWindowResize);
    }

    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    }

    function animate() {
      requestAnimationFrame(animate);

      const time = performance.now();
      const delta = (time - prevTime) / 1000;

      if (controls.isLocked === true) {
        // Basis snelheid
        let speed = 18.0;

        // Shift = sprint
        if (window.event && window.event.shiftKey) {
          speed = 30.0;
        }

        // Demping
        velocity.x -= velocity.x * 8.0 * delta;
        velocity.z -= velocity.z * 8.0 * delta;

        // zwaartekracht
        velocity.y -= 25.0 * delta;

        direction.z = Number(moveForward) - Number(moveBackward);
        direction.x = Number(moveRight) - Number(moveLeft);
        direction.normalize();

        if (moveForward || moveBackward) velocity.z -= direction.z * speed * delta;
        if (moveLeft || moveRight) velocity.x -= direction.x * speed * delta;

        controls.moveRight(-velocity.x * delta);
        controls.moveForward(-velocity.z * delta);

        controls.getObject().position.y += velocity.y * delta; // hoogte

        // op de grond blijven
        if (controls.getObject().position.y < 1.6) {
          velocity.y = 0;
          controls.getObject().position.y = 1.6;
          canJump = true;
        }
      }

      prevTime = time;

      renderer.render(scene, camera);
    }
  </script>
</body>
</html>
