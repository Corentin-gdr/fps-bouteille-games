git branch -m main <BRANCHE>
git récupérer l'origine
git branch -u origin/<BRANCHE>  <BRANCHE> 
git remote set-head origin -a
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <title>FPS Bouteille Cartoon</title>
  <style>
    body { margin: 0; overflow: hidden; }
    canvas { display: block; }
    #instructions {
      position: absolute;
      top: 20px;
      left: 20px;
      color: white;
      font-family: sans-serif;
      z-index: 10;
    }
  </style>
</head>
<body>
  <div id="instructions">
    Clique pour jouer<br />
    ZQSD pour bouger — Souris pour viser — Clic gauche pour tirer
  </div>

  <script type="module">
    import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.128/build/three.module.js';
    import { PointerLockControls } from 'https://cdn.jsdelivr.net/npm/three@0.128/examples/jsm/controls/PointerLockControls.js';

    let camera, scene, renderer, controls;
    let moveForward = false, moveBackward = false, moveLeft = false, moveRight = false;
    let velocity = new THREE.Vector3();
    let prevTime = performance.now();

    const objects = [];

    init();
    animate();

    function init() {
      scene = new THREE.Scene();
      scene.background = new THREE.Color(0x87ceeb);

      camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 1, 1000);

      renderer = new THREE.WebGLRenderer();
      renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(renderer.domElement);

      controls = new PointerLockControls(camera, document.body);
      document.body.addEventListener('click', () => controls.lock());
      scene.add(controls.getObject());

      // Sol
      const floorGeometry = new THREE.PlaneGeometry(1000, 1000);
      const floorMaterial = new THREE.MeshBasicMaterial({ color: 0x228b22 });
      const floor = new THREE.Mesh(floorGeometry, floorMaterial);
      floor.rotation.x = -Math.PI / 2;
      scene.add(floor);

      // Cibles
      for (let i = 0; i < 10; i++) {
        const boxGeometry = new THREE.BoxGeometry(1, 1, 1);
        const boxMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000 });
        const box = new THREE.Mesh(boxGeometry, boxMaterial);
        box.position.set(Math.random() * 50 - 25, 0.5, Math.random() * 50 - 25);
        scene.add(box);
        objects.push(box);
      }

      // Arme (cube tenu par la caméra)
      const weaponGeometry = new THREE.BoxGeometry(0.1, 0.1, 0.5);
      const weaponMaterial = new THREE.MeshBasicMaterial({ color: 0x333333 });
      const weapon = new THREE.Mesh(weaponGeometry, weaponMaterial);
      weapon.position.set(0.3, -0.3, -0.5);
      camera.add(weapon);

      // Déplacements
      const onKeyDown = (e) => {
        switch (e.code) {
          case 'KeyZ': moveForward = true; break;
          case 'KeyS': moveBackward = true; break;
          case 'KeyQ': moveLeft = true; break;
          case 'KeyD': moveRight = true; break;
        }
      };
      const onKeyUp = (e) => {
        switch (e.code) {
          case 'KeyZ': moveForward = false; break;
          case 'KeyS': moveBackward = false; break;
          case 'KeyQ': moveLeft = false; break;
          case 'KeyD': moveRight = false; break;
        }
      };
      document.addEventListener('keydown', onKeyDown);
      document.addEventListener('keyup', onKeyUp);

      // Tir
      document.addEventListener('mousedown', (e) => {
        if (e.button === 0) {
          shoot();
        }
      });

      window.addEventListener('resize', () => {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
      });
    }

    function shoot() {
      const raycaster = new THREE.Raycaster();
      raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
      const hits = raycaster.intersectObjects(objects);
      if (hits.length > 0) {
        const target = hits[0].object;
        scene.remove(target);
        objects.splice(objects.indexOf(target), 1);
        console.log("Cible touchée !");
      }
    }

    function animate() {
      requestAnimationFrame(animate);
      const time = performance.now();
      const delta = (time - prevTime) / 1000;

      velocity.x -= velocity.x * 10.0 * delta;
      velocity.z -= velocity.z * 10.0 * delta;

      const speed = 100.0;

      if (moveForward) velocity.z -= speed * delta;
      if (moveBackward) velocity.z += speed * delta;
      if (moveLeft) velocity.x -= speed * delta;
      if (moveRight) velocity.x += speed * delta;

      controls.moveRight(-velocity.x * delta);
      controls.moveForward(-velocity.z * delta);

      prevTime = time;

      renderer.render(scene, camera);
    }
  </script>
</body>
</html>
