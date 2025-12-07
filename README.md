<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>City Flight Escape</title>
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>

  <style>
    body {
      margin: 0;
      overflow: hidden;
      font-family: Arial, sans-serif;
      background: radial-gradient(circle at top, #8b0000, #000000 70%);
    }

    #hud {
      position: absolute;
      top: 20px;
      left: 20px;
      color: white;
      font-size: 16px;
      background: linear-gradient(135deg, #000000, #8b0000);
      padding: 12px 18px;
      border-radius: 12px;
      box-shadow: 0 0 15px red;
      display: flex;
      gap: 10px;
      align-items: center;
      z-index: 10;
    }

    button {
      background: black;
      color: red;
      border: 1px solid red;
      padding: 6px 12px;
      border-radius: 8px;
      cursor: pointer;
      font-weight: bold;
    }

    button:hover {
      background: red;
      color: black;
    }

    #endText {
      position: absolute;
      top: 45%;
      width: 100%;
      text-align: center;
      font-size: 50px;
      font-weight: bold;
      text-shadow: 0 0 25px black;
      z-index: 10;
    }
  </style>
</head>

<body>

  <div id="hud">
    <span id="timer">Time: 0</span>
    <button id="speedBtn">Speed: NORMAL</button>
    <button id="restartBtn">Restart</button>
  </div>

  <div id="endText"></div>

  <script>
    // SCENE
    const scene = new THREE.Scene();
    scene.fog = new THREE.Fog(0x110000, 10, 200);

    // FIRST PERSON CAMERA
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.set(0, 2, 15);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    // LIGHTS
    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(10, 20, 10);
    scene.add(light);

    const ambient = new THREE.AmbientLight(0xff2222, 0.6);
    scene.add(ambient);

    // GROUND
    const groundGeo = new THREE.PlaneGeometry(200, 500);
    const groundMat = new THREE.MeshStandardMaterial({ color: 0x070000 });
    const ground = new THREE.Mesh(groundGeo, groundMat);
    ground.rotation.x = -Math.PI / 2;
    ground.position.z = -200;
    scene.add(ground);

    // BUILDINGS
    const buildings = [];
    function generateCity() {
      buildings.length = 0;

      for (let i = 0; i < 260; i++) {
        const h = Math.random() * 18 + 5;
        const geo = new THREE.BoxGeometry(5, h, 5);
        const mat = new THREE.MeshStandardMaterial({
          color: 0x220000,
          emissive: 0x550000
        });

        const b = new THREE.Mesh(geo, mat);
        b.position.x = (Math.random() - 0.5) * 80;
        b.position.y = h / 2;
        b.position.z = -i * 8;
        buildings.push(b);
        scene.add(b);
      }
    }
    generateCity();

    // INPUT
    const keys = {};
    window.addEventListener("keydown", e => keys[e.key.toLowerCase()] = true);
    window.addEventListener("keyup", e => keys[e.key.toLowerCase()] = false);

    // GAME STATE
    let gameOver = false;
    let win = false;
    let startTime = Date.now();
    let speed = 0.4;
    let speedMode = 1;

    // SPEED BUTTON
    const speedBtn = document.getElementById("speedBtn");
    speedBtn.addEventListener("click", () => {
      speedMode++;
      if (speedMode > 3) speedMode = 1;

      if (speedMode === 1) {
        speed = 0.4;
        speedBtn.innerText = "Speed: NORMAL";
      }
      if (speedMode === 2) {
        speed = 0.7;
        speedBtn.innerText = "Speed: FAST";
      }
      if (speedMode === 3) {
        speed = 1.0;
        speedBtn.innerText = "Speed: INSANE";
      }
    });

    // RESTART BUTTON
    document.getElementById("restartBtn").addEventListener("click", () => {
      camera.position.set(0, 2, 15);
      gameOver = false;
      win = false;
      startTime = Date.now();
      document.getElementById("endText").innerHTML = "";
      buildings.forEach(b => scene.remove(b));
      generateCity();
    });

    // MAIN LOOP
    function animate() {
      requestAnimationFrame(animate);

      if (!gameOver && !win) {
        // Movement (First Person)
        if (keys["a"]) camera.position.x -= 0.5;
        if (keys["d"]) camera.position.x += 0.5;
        if (keys["w"]) camera.position.y += 0.2;
        if (keys["s"]) camera.position.y -= 0.2;

        camera.position.z -= speed;

        // Collision
        buildings.forEach(b => {
          if (camera.position.distanceTo(b.position) < 3.2) {
            endGame(false);
          }
        });

        // Timer
        let t = (Date.now() - startTime) / 1000;
        document.getElementById("timer").innerText = `Time: ${Math.floor(t)}`;

        if (t >= 30) endGame(true);
      }

      renderer.render(scene, camera);
    }
    animate();

    // END GAME
    function endGame(victory) {
      win = victory;
      gameOver = !victory;

      const end = document.getElementById("endText");
      if (victory) {
        end.innerHTML = "YOU ESCAPED!";
        end.style.color = "lime";
      } else {
        end.innerHTML = "YOU CRASHED!";
        end.style.color = "red";
      }
    }

    // RESIZE
    window.addEventListener("resize", () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>

</body>
</html>
