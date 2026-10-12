<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <!-- Meta tag actualizado para móviles: evita el zoom accidental de la página -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Simulador 3D - Vivienda Bioclimática</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #222;
            /* Evita gestos de recarga del navegador en móviles al deslizar */
            touch-action: none; 
        }

        /* Interfaz de Usuario (UI) */
        #ui-container {
            position: absolute;
            top: 20px;
            left: 20px;
            width: 320px;
            background: rgba(255, 255, 255, 0.95);
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.3);
            pointer-events: auto;
            z-index: 10;
            box-sizing: border-box;
        }

        h2 { margin-top: 0; font-size: 1.2rem; color: #333; }
        p { font-size: 0.9rem; color: #555; margin-bottom: 20px; line-height: 1.4; }

        .control-group {
            margin-bottom: 15px;
            padding-bottom: 15px;
            border-bottom: 1px solid #ddd;
        }

        .control-group:last-child { border-bottom: none; margin-bottom: 0; padding-bottom: 0; }

        button {
            width: 100%;
            padding: 12px 10px; /* Botones un poco más grandes para los dedos */
            margin-bottom: 10px;
            border: 2px solid #333;
            border-radius: 8px;
            background: white;
            color: #333;
            font-weight: bold;
            font-size: 1rem;
            cursor: pointer;
            transition: all 0.3s ease;
            /* Elimina el resaltado azul en móviles al tocar */
            -webkit-tap-highlight-color: transparent; 
        }

        button.active {
            background: #333;
            color: white;
            border-color: #333;
        }

        button:hover { background: #e0e0e0; }
        button.active:hover { background: #555; }

        /* Control deslizante para el Sol */
        .slider-container {
            display: flex;
            flex-direction: column;
            gap: 8px;
        }
        .slider-container label { font-size: 0.9rem; font-weight: bold; color: #444; }
        input[type="range"] { 
            width: 100%; 
            cursor: pointer; 
            height: 25px; /* Más fácil de agarrar en celular */
        }

        #instructions {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0,0,0,0.7);
            color: white;
            padding: 10px 20px;
            border-radius: 20px;
            font-size: 0.85rem;
            pointer-events: none;
            text-align: center;
            width: 85%;
            max-width: 400px;
            box-sizing: border-box;
            z-index: 10;
        }

        /* ==========================================
           DISEÑO RESPONSIVO PARA CELULARES
           ========================================== */
        @media (max-width: 600px) {
            #ui-container {
                top: 10px;
                left: 5%;
                width: 90%;
                padding: 15px;
            }
            h2 { font-size: 1.1rem; margin-bottom: 5px; }
            p { margin-bottom: 15px; font-size: 0.85rem; }
            button { font-size: 0.9rem; padding: 10px; }
            #instructions { font-size: 0.75rem; bottom: 10px; padding: 8px 12px; }
        }
    </style>
    <!-- Importar Three.js y OrbitControls desde CDN -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
</head>
<body>

    <div id="ui-container">
        <h2>Análisis Bioclimático 3D</h2>
        <p>Gira el modelo con el ratón o tus dedos. Usa los controles para simular los fenómenos.</p>

        <div class="control-group slider-container">
            <label for="sunSlider">☀️ Asoleamiento (Hora del día)</label>
            <input type="range" id="sunSlider" min="0" max="100" value="30">
        </div>

        <div class="control-group">
            <button id="btn-wind">🌬️ Activar Ventilación Cruzada</button>
        </div>

        <div class="control-group">
            <button id="btn-temp">🌡️ Análisis Térmico (Calor)</button>
        </div>
    </div>

    <!-- Instrucciones actualizadas para incluir gestos móviles -->
    <div id="instructions">
        👆 1 dedo: Orbitar | ✌️ 2 dedos: Zoom/Mover<br>
        (🖱️ PC: Clic izq para rotar | Rueda para zoom)
    </div>

    <script>
        // 1. CONFIGURACIÓN BÁSICA DE THREE.JS
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x87CEEB); 

        const camera = new THREE.PerspectiveCamera(50, window.innerWidth / window.innerHeight, 0.1, 1000);
        
        // Ajustamos la cámara inicial un poco más lejos para móviles
        if (window.innerWidth < 600) {
            camera.position.set(15, 12, 22);
        } else {
            camera.position.set(12, 10, 18);
        }

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        document.body.appendChild(renderer.domElement);

        const controls = new THREE.OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;
        controls.maxPolarAngle = Math.PI / 2 - 0.05;
        
        // Habilitar controles táctiles explícitamente (activados por defecto, pero lo aseguramos)
        controls.enableTouch = true; 

        // 2. ILUMINACIÓN Y SOL
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.4);
        scene.add(ambientLight);

        const sunLight = new THREE.DirectionalLight(0xfff5b6, 1.2);
        sunLight.castShadow = true;
        sunLight.shadow.mapSize.width = 2048;
        sunLight.shadow.mapSize.height = 2048;
        sunLight.shadow.camera.near = 0.5;
        sunLight.shadow.camera.far = 50;
        const shadowSize = 12;
        sunLight.shadow.camera.left = -shadowSize;
        sunLight.shadow.camera.right = shadowSize;
        sunLight.shadow.camera.top = shadowSize;
        sunLight.shadow.camera.bottom = -shadowSize;
        scene.add(sunLight);

        const sunGeo = new THREE.SphereGeometry(1, 32, 32);
        const sunMat = new THREE.MeshBasicMaterial({ color: 0xffd700 });
        const sunMesh = new THREE.Mesh(sunGeo, sunMat);
        scene.add(sunMesh);

        // 3. CONSTRUCCIÓN DE LA VIVIENDA
        const materials = {
            base: new THREE.MeshStandardMaterial({ color: 0xcccccc }),
            wall: new THREE.MeshStandardMaterial({ color: 0xeaeaea, roughness: 0.8 }),
            roof: new THREE.MeshStandardMaterial({ color: 0x444444 }),
            grass: new THREE.MeshStandardMaterial({ color: 0x4CAF50, roughness: 1 })
        };

        const houseGroup = new THREE.Group();
        scene.add(houseGroup);

        // Suelo Exterior
        const groundGeo = new THREE.PlaneGeometry(50, 50);
        const groundMat = new THREE.MeshStandardMaterial({ color: 0x556B2F });
        const ground = new THREE.Mesh(groundGeo, groundMat);
        ground.rotation.x = -Math.PI / 2;
        ground.receiveShadow = true;
        scene.add(ground);

        // Base
        const base = new THREE.Mesh(new THREE.BoxGeometry(10, 0.5, 8), materials.base);
        base.position.y = 0.25;
        base.receiveShadow = true;
        base.castShadow = true;
        houseGroup.add(base);

        function createWall(w, h, d, x, y, z) {
            const wall = new THREE.Mesh(new THREE.BoxGeometry(w, h, d), materials.wall);
            wall.position.set(x, y, z);
            wall.castShadow = true;
            wall.receiveShadow = true;
            houseGroup.add(wall);
            return wall;
        }

        const walls = [];
        walls.push(createWall(9.6, 4, 0.4, 0, 2.5, -3.8)); // Muro Trasero
        
        // Fachada de entrada (Ventana baja)
        walls.push(createWall(0.4, 1, 8, -4.8, 1, 0));
        walls.push(createWall(0.4, 1.5, 8, -4.8, 3.75, 0));
        walls.push(createWall(0.4, 1.5, 2.5, -4.8, 2.25, 2.75));
        walls.push(createWall(0.4, 1.5, 2.5, -4.8, 2.25, -2.75));

        // Fachada de salida (Ventana alta)
        walls.push(createWall(0.4, 2.5, 8, 4.8, 1.75, 0)); 
        walls.push(createWall(0.4, 0.5, 8, 4.8, 4.25, 0)); 
        walls.push(createWall(0.4, 1.5, 2.5, 4.8, 3.25, 2.75));
        walls.push(createWall(0.4, 1.5, 2.5, 4.8, 3.25, -2.75));

        // Cubierta
        const roof = new THREE.Mesh(new THREE.BoxGeometry(10.4, 0.4, 8.4), materials.roof);
        roof.position.set(0, 4.7, 0);
        roof.castShadow = true;
        roof.receiveShadow = true;
        houseGroup.add(roof);

        // Cubierta Verde
        const greenRoof = new THREE.Mesh(new THREE.BoxGeometry(10, 0.2, 8), materials.grass);
        greenRoof.position.set(0, 5.0, 0);
        greenRoof.receiveShadow = true;
        houseGroup.add(greenRoof);

        for(let i = 0; i < 40; i++) {
            const plant = new THREE.Mesh(new THREE.ConeGeometry(0.15, 0.4, 4), materials.grass);
            plant.position.set((Math.random() - 0.5) * 9, 5.3, (Math.random() - 0.5) * 7);
            plant.castShadow = true;
            houseGroup.add(plant);
        }

        // Volumen de Aire Interior
        const interiorAirGeo = new THREE.BoxGeometry(9.2, 4, 7.6);
        const interiorAirMat = new THREE.MeshBasicMaterial({ color: 0x00ffff, transparent: true, opacity: 0, depthWrite: false });
        const interiorAir = new THREE.Mesh(interiorAirGeo, interiorAirMat);
        interiorAir.position.set(0, 2.5, 0);
        houseGroup.add(interiorAir);

        // 4. SISTEMA DE PARTÍCULAS
        const windParticles = new THREE.Group();
        windParticles.visible = false;
        scene.add(windParticles);

        const particleGeo = new THREE.SphereGeometry(0.1, 8, 8);
        const particleMat = new THREE.MeshBasicMaterial({ color: 0x00aaff, transparent: true, opacity: 0.6 });
        
        const particles = [];
        for(let i = 0; i < 150; i++) {
            const p = new THREE.Mesh(particleGeo, particleMat);
            resetParticle(p);
            p.position.x = (Math.random() - 0.5) * 15;
            windParticles.add(p);
            particles.push(p);
        }

        function resetParticle(p) {
            p.position.set(-8 - Math.random() * 2, 1.5 + Math.random() * 1.5, (Math.random() - 0.5) * 2);
            p.userData.speedX = 0.05 + Math.random() * 0.05;
            p.userData.offsetY = Math.random() * Math.PI * 2;
        }

        // 5. INTERACCIÓN Y UI
        const sunSlider = document.getElementById('sunSlider');
        function updateSunPosition() {
            const time = sunSlider.value / 100; 
            const angle = Math.PI * time; 
            const radius = 15;
            
            sunLight.position.x = Math.cos(angle) * radius;
            sunLight.position.y = Math.sin(angle) * radius;
            sunLight.position.z = 5; 
            sunMesh.position.copy(sunLight.position);

            if (time < 0.2 || time > 0.8) {
                scene.background.setHex(0xffaa66); 
                sunLight.color.setHex(0xffaa66);
                sunLight.intensity = 0.8;
            } else {
                scene.background.setHex(0x87CEEB); 
                sunLight.color.setHex(0xffffff);
                sunLight.intensity = 1.2;
            }
        }
        
        // En móviles, 'input' funciona bien pero agregamos 'change' por compatibilidad
        sunSlider.addEventListener('input', updateSunPosition);
        sunSlider.addEventListener('change', updateSunPosition);
        updateSunPosition();

        let isWindActive = false;
        const btnWind = document.getElementById('btn-wind');
        btnWind.addEventListener('click', () => {
            isWindActive = !isWindActive;
            windParticles.visible = isWindActive;
            btnWind.classList.toggle('active');
        });

        let isTempActive = false;
        const btnTemp = document.getElementById('btn-temp');
        btnTemp.addEventListener('click', () => {
            isTempActive = !isTempActive;
            btnTemp.classList.toggle('active');
            
            if (isTempActive) {
                materials.wall.color.setHex(0xff4422);
                materials.wall.emissive.setHex(0x330000);
                interiorAir.material.opacity = 0.35;
                scene.background.setHex(0xff7744); 
            } else {
                materials.wall.color.setHex(0xeaeaea);
                materials.wall.emissive.setHex(0x000000);
                interiorAir.material.opacity = 0;
                updateSunPosition();
            }
        });

        // 6. RENDERIZADO
        function animate() {
            requestAnimationFrame(animate);

            if (isWindActive) {
                const time = Date.now() * 0.002;
                particles.forEach(p => {
                    p.position.x += p.userData.speedX;
                    if (p.position.x > -4 && p.position.x < 4) {
                        p.position.y += 0.01; // Efecto ascendente
                    }
                    p.position.z += Math.sin(time + p.userData.offsetY) * 0.01;
                    if (p.position.x > 8) resetParticle(p);
                });
            }

            controls.update();
            renderer.render(scene, camera);
        }

        // Manejo del cambio de pantalla (Rotar el celular)
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        animate();
    </script>
</body>
</html>
