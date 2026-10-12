<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Modelo Bioclimático 3D</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #111;
        }

        /* Botón para desplegar/ocultar el menú */
        #toggle-menu {
            position: absolute;
            top: 15px;
            left: 15px;
            background: rgba(255, 255, 255, 0.95);
            border: 2px solid #2c3e50;
            border-radius: 8px;
            padding: 10px 15px;
            font-size: 1rem;
            font-weight: bold;
            color: #2c3e50;
            cursor: pointer;
            z-index: 1000;
            box-shadow: 0 4px 10px rgba(0,0,0,0.3);
            -webkit-tap-highlight-color: transparent;
            transition: background 0.3s ease;
        }

        #toggle-menu:hover {
            background: #f0f0f0;
        }

        /* Interfaz de Usuario (Panel) */
        #ui-container {
            position: absolute;
            top: 70px; /* Debajo del botón toggle */
            left: 15px;
            width: 300px;
            max-width: 85vw;
            background: rgba(255, 255, 255, 0.95);
            padding: 15px;
            border-radius: 12px;
            box-shadow: 0 8px 25px rgba(0,0,0,0.4);
            z-index: 999;
            pointer-events: auto;
            /* Transición para el efecto de deslizar */
            transform: translateX(0);
            transition: transform 0.3s cubic-bezier(0.4, 0.0, 0.2, 1), opacity 0.3s ease;
            opacity: 1;
        }

        /* Clase para ocultar el panel desplazándolo hacia la izquierda */
        #ui-container.hidden {
            transform: translateX(-150%);
            opacity: 0;
            pointer-events: none;
        }

        h2 { margin: 0 0 10px 0; font-size: 1.1rem; color: #222; }
        p { font-size: 0.8rem; color: #555; margin-bottom: 15px; line-height: 1.3; }

        .control-group { margin-bottom: 12px; }

        button.action-btn {
            width: 100%;
            padding: 12px;
            margin-bottom: 8px;
            border: 2px solid #2c3e50;
            border-radius: 8px;
            background: white;
            color: #2c3e50;
            font-weight: 700;
            font-size: 0.9rem;
            cursor: pointer;
            transition: all 0.2s ease;
            -webkit-tap-highlight-color: transparent;
            touch-action: manipulation;
        }

        button.action-btn.active {
            background: #2c3e50;
            color: white;
        }

        .slider-container { display: flex; flex-direction: column; gap: 5px; margin-bottom: 15px; }
        .slider-container label { font-size: 0.85rem; font-weight: bold; color: #333; }
        input[type="range"] { 
            width: 100%; 
            height: 30px; 
            cursor: pointer; 
            touch-action: none; 
        }

        #instructions {
            position: absolute;
            bottom: 15px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0,0,0,0.7);
            color: white;
            padding: 8px 16px;
            border-radius: 20px;
            font-size: 0.75rem;
            pointer-events: none;
            text-align: center;
            width: max-content;
            z-index: 10;
            backdrop-filter: blur(4px);
        }

        @media (max-width: 600px) {
            #toggle-menu { top: 10px; left: 10px; }
            #ui-container { top: 60px; left: 10px; width: calc(100% - 20px); box-sizing: border-box; }
        }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
</head>
<body>

    <!-- Botón para Mostrar/Ocultar Menú -->
    <button id="toggle-menu">☰ Controles</button>

    <div id="ui-container">
        <h2>Simulación Bioclimática</h2>
        <p>Ajusta las variables para analizar el comportamiento pasivo del proyecto.</p>

        <div class="slider-container">
            <label for="sunSlider">☀️ Trayectoria Solar (Hora)</label>
            <input type="range" id="sunSlider" min="0" max="100" value="30">
        </div>

        <div class="control-group">
            <button id="btn-wind" class="action-btn">🌬️ Activar Flujo de Aire</button>
            <button id="btn-temp" class="action-btn">🌡️ Analizar Temperatura</button>
        </div>
    </div>

    <div id="instructions">👆 1 dedo: Orbitar | ✌️ 2 dedos: Zoom</div>

    <script>
        // --- LÓGICA DEL MENÚ DESPLEGABLE ---
        const toggleBtn = document.getElementById('toggle-menu');
        const uiContainer = document.getElementById('ui-container');

        toggleBtn.addEventListener('click', () => {
            uiContainer.classList.toggle('hidden');
            if (uiContainer.classList.contains('hidden')) {
                toggleBtn.innerText = '☰ Mostrar Controles';
            } else {
                toggleBtn.innerText = '☰ Ocultar Controles';
            }
        });

        // --- 1. ESCENA Y CÁMARA ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x7ec0ee); 

        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(16, 12, 20);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        renderer.domElement.style.touchAction = 'none';
        document.body.appendChild(renderer.domElement);

        const controls = new THREE.OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;
        controls.maxPolarAngle = Math.PI / 2 - 0.05; 

        // --- 2. ILUMINACIÓN ---
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
        scene.add(ambientLight);

        const sunLight = new THREE.DirectionalLight(0xfff5b6, 1.2);
        sunLight.castShadow = true;
        sunLight.shadow.mapSize.width = 2048;
        sunLight.shadow.mapSize.height = 2048;
        sunLight.shadow.camera.near = 0.5;
        sunLight.shadow.camera.far = 50;
        sunLight.shadow.camera.left = -15;
        sunLight.shadow.camera.right = 15;
        sunLight.shadow.camera.top = 15;
        sunLight.shadow.camera.bottom = -15;
        scene.add(sunLight);

        const sunGeo = new THREE.SphereGeometry(1.2, 32, 32);
        const sunMat = new THREE.MeshBasicMaterial({ color: 0xffdd00 });
        const sunMesh = new THREE.Mesh(sunGeo, sunMat);
        scene.add(sunMesh);

        // --- 3. MATERIALES ---
        const mat = {
            ground: new THREE.MeshStandardMaterial({ color: 0x6e8b3d, roughness: 1 }),
            floor: new THREE.MeshStandardMaterial({ color: 0x909090, roughness: 0.9 }),
            wall: new THREE.MeshStandardMaterial({ color: 0xfafafa, roughness: 0.8 }), 
            frame: new THREE.MeshStandardMaterial({ color: 0x222222, roughness: 0.3, metalness: 0.6 }), // Aluminio oscuro
            glass: new THREE.MeshPhysicalMaterial({  // Material de cristal realista
                color: 0xadd8e6,
                metalness: 0.1,
                roughness: 0.1,
                transmission: 0.8, // Transparencia tipo cristal (necesita iluminación)
                transparent: true,
                opacity: 0.4
            }),
            roofFront: new THREE.MeshStandardMaterial({ color: 0x555555 }),
            grass: new THREE.MeshStandardMaterial({ color: 0x4CAF50, roughness: 0.8 })
        };

        const houseGroup = new THREE.Group();
        scene.add(houseGroup);

        // Suelo y Base
        const ground = new THREE.Mesh(new THREE.PlaneGeometry(60, 60), mat.ground);
        ground.rotation.x = -Math.PI / 2;
        ground.receiveShadow = true;
        scene.add(ground);

        const base = new THREE.Mesh(new THREE.BoxGeometry(11, 0.4, 9), mat.floor);
        base.position.y = 0.2;
        base.receiveShadow = true;
        base.castShadow = true;
        houseGroup.add(base);

        function createBox(w, h, d, x, y, z, material, castShadow = true) {
            const mesh = new THREE.Mesh(new THREE.BoxGeometry(w, h, d), material);
            mesh.position.set(x, y, z);
            mesh.castShadow = castShadow;
            mesh.receiveShadow = true;
            houseGroup.add(mesh);
            return mesh;
        }

        // MUROS Y VENTANAS MEJORADAS
        // Muro Trasero
        createBox(10.6, 4, 0.4, 0, 2.4, -4.3, mat.wall);
        
        // Muro Izquierdo (Entrada)
        createBox(0.4, 1, 9, -5.3, 0.9, 0, mat.wall); // Antepecho
        createBox(0.4, 1.5, 9, -5.3, 3.65, 0, mat.wall); // Dintel superior
        createBox(0.4, 1.5, 3, -5.3, 2.15, 3, mat.wall); // Lado frontal
        createBox(0.4, 1.5, 3, -5.3, 2.15, -3, mat.wall); // Lado trasero
        
        // Marco y Cristal Ventana Izquierda
        createBox(0.45, 1.6, 3.2, -5.3, 2.15, 0, mat.frame); // Marco principal
        createBox(0.5, 1.5, 0.1, -5.3, 2.15, 0, mat.frame);  // Parteluz (división central)
        createBox(0.35, 1.4, 3.0, -5.3, 2.15, 0, mat.glass, false); // Cristal

        // Muro Derecho (Salida térmica)
        createBox(0.4, 2.5, 9, 5.3, 1.65, 0, mat.wall); // Antepecho más alto
        createBox(0.4, 0.5, 9, 5.3, 4.15, 0, mat.wall); // Dintel superior
        createBox(0.4, 1, 3, 5.3, 3.4, 3, mat.wall); // Lado frontal
        createBox(0.4, 1, 3, 5.3, 3.4, -3, mat.wall); // Lado trasero
        
        // Marco y Cristal Ventana Derecha
        createBox(0.45, 1.1, 3.2, 5.3, 3.4, 0, mat.frame); // Marco
        createBox(0.5, 1.0, 0.1, 5.3, 3.4, 0, mat.frame);  // Parteluz
        createBox(0.35, 0.9, 3.0, 5.3, 3.4, 0, mat.glass, false); // Cristal

        // Cubierta y Vegetación
        createBox(13, 0.3, 11, 0, 4.55, 0, mat.roofFront);
        createBox(12.6, 0.2, 10.6, 0, 4.8, 0, mat.grass);

        for(let i = 0; i < 60; i++) {
            const isCone = Math.random() > 0.5;
            const geo = isCone ? new THREE.ConeGeometry(0.2, 0.5, 5) : new THREE.SphereGeometry(0.25, 5, 5);
            const plant = new THREE.Mesh(geo, mat.grass);
            plant.position.set((Math.random() - 0.5) * 11, 5.0, (Math.random() - 0.5) * 9);
            plant.castShadow = true;
            houseGroup.add(plant);
        }

        // --- 4. SISTEMAS FÍSICOS (Viento y Calor) ---
        const windGroup = new THREE.Group();
        windGroup.visible = false;
        scene.add(windGroup);
        const windGeo = new THREE.SphereGeometry(0.12, 8, 8);
        const windMat = new THREE.MeshBasicMaterial({ color: 0x00ffff, transparent: true, opacity: 0.7 });
        const winds = [];
        for(let i = 0; i < 120; i++) {
            const p = new THREE.Mesh(windGeo, windMat);
            resetWind(p);
            p.position.x = (Math.random() - 0.5) * 15;
            windGroup.add(p);
            winds.push(p);
        }
        function resetWind(p) {
            p.position.set(-8 - Math.random() * 2, 1.2 + Math.random() * 1.5, (Math.random() - 0.5) * 2);
            p.userData.speedX = 0.08 + Math.random() * 0.04;
            p.userData.offsetY = Math.random() * Math.PI * 2;
        }

        const heatGroup = new THREE.Group();
        heatGroup.visible = false;
        scene.add(heatGroup);
        const hotGeo = new THREE.SphereGeometry(0.15, 6, 6);
        const hotMat = new THREE.MeshBasicMaterial({ color: 0xff3300, transparent: true, opacity: 0.6 });
        const coolMat = new THREE.MeshBasicMaterial({ color: 0x00aaff, transparent: true, opacity: 0.6 });
        const heatParticles = [];
        
        for(let i = 0; i < 80; i++) {
            const p = new THREE.Mesh(hotGeo, hotMat);
            p.position.set((Math.random() - 0.5) * 14, Math.random() * 6, (Math.random() - 0.5) * 14);
            p.userData.isOutside = true;
            heatGroup.add(p);
            heatParticles.push(p);
        }
        for(let i = 0; i < 60; i++) {
            const p = new THREE.Mesh(hotGeo, coolMat);
            p.position.set((Math.random() - 0.5) * 9, 0.5 + Math.random() * 3, (Math.random() - 0.5) * 7);
            p.userData.isOutside = false;
            heatGroup.add(p);
            heatParticles.push(p);
        }

        // --- 5. INTERACCIONES UI ---
        const sunSlider = document.getElementById('sunSlider');
        function updateSun() {
            const time = sunSlider.value / 100; 
            const angle = Math.PI * time; 
            sunLight.position.x = Math.cos(angle) * 20;
            sunLight.position.y = Math.sin(angle) * 18;
            sunLight.position.z = 8; 
            sunMesh.position.copy(sunLight.position);

            if (!document.getElementById('btn-temp').classList.contains('active')) {
                if (time < 0.2 || time > 0.8) {
                    scene.background.setHex(0xffaa66);
                    sunLight.color.setHex(0xffaa66);
                    sunLight.intensity = 0.7;
                } else {
                    scene.background.setHex(0x7ec0ee);
                    sunLight.color.setHex(0xffffff);
                    sunLight.intensity = 1.3;
                }
            }
        }
        sunSlider.addEventListener('input', updateSun);
        sunSlider.addEventListener('change', updateSun);
        updateSun();

        let isWind = false;
        document.getElementById('btn-wind').addEventListener('click', function() {
            isWind = !isWind;
            windGroup.visible = isWind;
            this.classList.toggle('active');
            
            // Efecto visual: abrir las ventanas (hacer desaparecer los cristales momentáneamente)
            mat.glass.opacity = isWind ? 0.1 : 0.4; 
        });

        let isTemp = false;
        document.getElementById('btn-temp').addEventListener('click', function() {
            isTemp = !isTemp;
            heatGroup.visible = isTemp;
            this.classList.toggle('active');
            
            if (isTemp) {
                mat.wall.color.setHex(0xff4422); 
                mat.ground.color.setHex(0x553311);
                scene.background.setHex(0x3a1005); 
                sunLight.intensity = 0.5;
            } else {
                mat.wall.color.setHex(0xfafafa); 
                mat.ground.color.setHex(0x6e8b3d);
                updateSun(); 
            }
        });

        // --- 6. ANIMACIÓN ---
        function animate() {
            requestAnimationFrame(animate);
            const time = Date.now() * 0.002;

            if (isWind) {
                winds.forEach(p => {
                    p.position.x += p.userData.speedX;
                    if (p.position.x > -4.5 && p.position.x < 4.5) p.position.y += 0.015; 
                    p.position.z += Math.sin(time * 2 + p.userData.offsetY) * 0.01;
                    if (p.position.x > 9) resetWind(p);
                });
            }

            if (isTemp) {
                heatParticles.forEach(p => {
                    if (p.userData.isOutside) {
                        p.position.y += 0.02; 
                        if (p.position.y > 7) p.position.y = 0;
                    } else {
                        p.position.y += Math.sin(time + p.position.x) * 0.005; 
                    }
                });
            }

            controls.update();
            renderer.render(scene, camera);
        }

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        animate();
    </script>
</body>
</html>
