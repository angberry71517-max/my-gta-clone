<!DOCTYPE html>
<html>
<head>
    <title>GTA-ULTRA: Infinite 4K City</title>
    <style>
        body { margin: 0; background: #000; overflow: hidden; }
        #hud { 
            position: absolute; top: 20px; left: 20px; color: #0ff; 
            font-family: 'Courier New', monospace; text-transform: uppercase;
            letter-spacing: 2px; pointer-events: none;
        }
        .bar { width: 200px; height: 10px; border: 1px solid #0ff; margin-top: 5px; }
        #nitro-fill { width: 0%; height: 100%; background: #0ff; transition: 0.1s; }
    </style>
</head>
<body>
    <div id="hud">
        <div id="speed">000 KM/H</div>
        <div class="bar"><div id="nitro-fill"></div></div>
        <div style="font-size: 10px; margin-top: 10px;">CORE_ENGINE: ACTIVE | RENDER_SCALE: 4K</div>
    </div>

    <script type="importmap">
        { "imports": { "three": "https://unpkg.com/three@0.160.0/build/three.module.js" } }
    </script>

    <script type="module">
        import * as THREE from 'three';

        // --- ENGINE CONFIG ---
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x00050a, 0.015); // Dark city atmosphere
        
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 3000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.toneMapping = THREE.ReinhardToneMapping;
        document.body.appendChild(renderer.domElement);

        // --- WORLD GEN (The "Infinite" Logic) ---
        const buildings = [];
        const buildingMat = new THREE.MeshPhongMaterial({ color: 0x111111, emissive: 0x001122 });

        function createBuilding(x, z) {
            const h = 20 + Math.random() * 100;
            const geo = new THREE.BoxGeometry(15, h, 15);
            const mesh = new THREE.Mesh(geo, buildingMat);
            mesh.position.set(x, h/2, z);
            
            // Add Window Lights (The "Real Life" look)
            const lightGeo = new THREE.BoxGeometry(15.2, 1, 15.2);
            const lightMat = new THREE.MeshBasicMaterial({ color: Math.random() > 0.5 ? 0xffaa00 : 0x00ffff });
            const windows = new THREE.Mesh(lightGeo, lightMat);
            windows.position.y = Math.random() * (h/2);
            mesh.add(windows);
            
            scene.add(mesh);
            buildings.push({ mesh, x, z });
        }

        // Generate initial city grid
        for(let x = -5; x < 5; x++) {
            for(let z = -5; z < 5; z++) {
                if(Math.abs(x) > 1 || Math.abs(z) > 1) createBuilding(x * 60, z * 60);
            }
        }

        // --- THE PLAYER CAR ---
        const car = new THREE.Group();
        const chassis = new THREE.Mesh(
            new THREE.BoxGeometry(2.5, 0.8, 5),
            new THREE.MeshStandardMaterial({ color: 0x000000, metalness: 1, roughness: 0.1 })
        );
        car.add(chassis);
        
        // Headlights
        const headLight = new THREE.SpotLight(0xffffff, 50, 100, Math.PI/4);
        headLight.position.set(0, 0, 2.5);
        car.add(headLight);
        car.add(headLight.target);
        headLight.target.position.set(0,0,10);
        
        scene.add(car);

        // --- CONTROLS & FLUID PHYSICS ---
        let velocity = 0;
        let nitro = 0;
        const keys = {};
        window.onkeydown = (e) => keys[e.code] = true;
        window.onkeyup = (e) => keys[e.code] = false;

        function updateCity() {
            // Recycling buildings to save memory (Optimization)
            buildings.forEach(b => {
                if (b.mesh.position.distanceTo(car.position) > 400) {
                    b.mesh.position.z += (car.position.z > b.mesh.position.z) ? 800 : -800;
                }
            });
        }

        function animate() {
            requestAnimationFrame(animate);

            // Physics
            if(keys['KeyW']) velocity += 0.01;
            if(keys['KeyS']) velocity -= 0.01;
            if(keys['ShiftLeft'] && nitro < 100) { velocity *= 1.05; nitro += 0.5; } 
            else { nitro *= 0.95; }

            velocity *= 0.98;
            car.translateZ(velocity);

            if(keys['KeyA']) car.rotation.y += 0.04;
            if(keys['KeyD']) car.rotation.y -= 0.04;

            // Update UI
            document.getElementById('speed').innerText = Math.floor(Math.abs(velocity) * 400) + " KM/H";
            document.getElementById('nitro-fill').style.width = nitro + "%";

            // Camera follow
            const camOffset = new THREE.Vector3(0, 5, -15).applyQuaternion(car.quaternion).add(car.position);
            camera.position.lerp(camOffset, 0.1);
            camera.lookAt(car.position);

            updateCity();
            renderer.render(scene, camera);
        }
        animate();
    </script>
</body>
</html>
