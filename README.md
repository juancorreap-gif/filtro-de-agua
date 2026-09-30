<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Prototipo Digital 3D - Filtro Multicapas</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #1a1a1a;
            color: #fff;
        }
        #canvas-container {
            width: 100vw;
            height: 100vw;
            max-height: 70vh;
        }
        #ui-panel {
            position: absolute;
            bottom: 0;
            left: 0;
            width: 100%;
            box-sizing: border-box;
            background: rgba(30, 30, 30, 0.85);
            backdrop-filter: blur(10px);
            padding: 20px;
            border-top: 2px solid #333;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        h1 {
            margin: 0 0 10px 0;
            font-size: 1.4rem;
            color: #4fc3f7;
            text-align: center;
        }
        .controls {
            display: flex;
            gap: 15px;
            margin-bottom: 15px;
        }
        button {
            background-color: #0288d1;
            color: white;
            border: none;
            padding: 10px 20px;
            font-size: 1rem;
            border-radius: 5px;
            cursor: pointer;
            transition: background 0.3s;
            font-weight: bold;
        }
        button:hover {
            background-color: #03a9f4;
        }
        #info-text {
            font-size: 0.95rem;
            text-align: center;
            max-width: 600px;
            color: #ccc;
            line-height: 1.4;
        }
        #instructions {
            position: absolute;
            top: 15px;
            left: 15px;
            background: rgba(0,0,0,0.6);
            padding: 10px;
            border-radius: 5px;
            font-size: 0.85rem;
            pointer-events: none;
        }
    </style>
    <!-- Cargamos Three.js y OrbitControls para poder rotar la cámara -->
    <script src="https://cloudflare.com"></script>
    <script src="https://jsdelivr.net"></script>
</head>
<body>

    <div id="instructions">🖱️ Arrastra con el ratón para rotar el filtro en 3D</div>
    <div id="canvas-container"></div>

    <div id="ui-panel">
        <h1>Prototipo de Filtro Multicapas 3D</h1>
        <div class="controls">
            <button id="btn-filter">Simular Filtrado</button>
            <button id="btn-reset">Reiniciar</button>
        </div>
        <div id="info-text">Selecciona una acción para ver cómo interactúa el agua con los materiales porosos y de retención física.</div>
    </div>

    <script>
        // --- CONFIGURACIÓN ESCENA 3D ---
        const container = document.getElementById('canvas-container');
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x1a1a1a);

        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / (window.innerHeight * 0.7), 0.1, 1000);
        camera.position.set(0, 3, 10);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight * 0.7);
        renderer.setPixelRatio(window.devicePixelRatio);
        container.appendChild(renderer.domElement);

        const controls = new THREE.OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;
        controls.maxPolarAngle = Math.PI / 2 + 0.3; // Evita ir totalmente abajo

        // Iluminación
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
        scene.add(ambientLight);
        const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
        dirLight.position.set(5, 10, 7);
        scene.add(dirLight);

        // --- MODELADO DEL FILTRO ---
        const filterGroup = new THREE.Group();

        // 1. Botella de Plástico (Cuerpo transparente)
        const bottleGeom = new THREE.CylinderGeometry(1.5, 1.2, 6, 32, 1, true);
        const bottleMat = new THREE.MeshPhysicalMaterial({
            color: 0xffffff,
            transparent: true,
            opacity: 0.25,
            roughness: 0.1,
            transmission: 0.9, 
            ior: 1.5,
            side: THREE.DoubleSide
        });
        const bottle = new THREE.Mesh(bottleGeom, bottleMat);
        filterGroup.add(bottle);

        // Boquilla de la botella abajo (embudo)
        const neckGeom = new THREE.CylinderGeometry(1.2, 0.4, 1, 32, 1, true);
        const neck = new THREE.Mesh(neckGeom, bottleMat);
        neck.position.y = -3.5;
        filterGroup.add(neck);

        // 2. Capas de Materiales (De abajo hacia arriba)
        const layersData = [
            { name: "Algodón / Gasa", height: 0.4, color: 0xffffff, y: -3.2, info: "Algodón: Retiene partículas diminutas flotantes antes de la salida." },
            { name: "Piedras Grandes (2-3 cm)", height: 1.0, color: 0x757575, y: -2.5, info: "Piedras Grandes: Capa de soporte que evita obstrucciones en la boquilla." },
            { name: "Piedras Medianas / Gravilla", height: 0.8, color: 0x9e9e9e, y: -1.6, info: "Piedras Medianas: Atrapan sedimentos de tamaño intermedio." },
            { name: "Arena Gruesa", height: 0.8, color: 0xd7ccc8, y: -0.8, info: "Arena Gruesa: Retiene residuos pequeños y detiene el paso del carbón." },
            { name: "Arena Fina", height: 0.8, color: 0xfff9c4, y: 0.0, info: "Arena Fina: Filtra impurezas minúsculas y clarifica el agua." },
            { name: "Carbón Vegetal Triturado", height: 1.0, color: 0x212121, y: 0.9, info: "Carbón Vegetal: Absorbe compuestos químicos, quita olores y mejora el sabor." },
            { name: "Grava Superior de Protección", height: 0.4, color: 0xbcaa9b, y: 1.6, info: "Grava Superior: Amortigua la caída del agua para no desacomodar el carbón." }
        ];

        const layersMeshes = [];

        layersData.forEach((layer) => {
            // Radio dinámico adaptado a la forma del cono de la botella
            const rTop = 1.2 + ((layer.y + layer.height/2 + 3) / 6) * 0.3;
            const rBot = 1.2 + ((layer.y - layer.height/2 + 3) / 6) * 0.3;
            
            const geom = new THREE.CylinderGeometry(rTop, rBot, layer.height, 32);
            const mat = new THREE.MeshStandardMaterial({
                color: layer.color,
                roughness: 0.8,
                metalness: 0.1,
                transparent: true,
                opacity: 0.9
            });
            const mesh = new THREE.Mesh(geom, mat);
            mesh.position.y = layer.y;
            mesh.userData = { info: layer.info, name: layer.name };
            filterGroup.add(mesh);
            layersMeshes.push(mesh);
        });

        // 3. Simulación de Agua
        const waterGeom = new THREE.CylinderGeometry(1.48, 1.45, 0.8, 32);
        const waterMat = new THREE.MeshStandardMaterial({
            color: 0x4fc3f7,
            transparent: true,
            opacity: 0.0, // Inicia invisible
            roughness: 0.1
        });
        const water = new THREE.Mesh(waterGeom, waterMat);
        water.position.y = 2.2;
        filterGroup.add(water);

        scene.add(filterGroup);

        // --- ANIMACIÓN Y LÓGICA DE INTERACCIÓN ---
        let isFiltering = false;
        let waterProgress = 0;
        const infoText = document.getElementById('info-text');

        document.getElementById('btn-filter').addEventListener('click', () => {
            isFiltering = true;
            waterMat.opacity = 0.7;
            infoText.innerHTML = "💧 <b>Filtrando:</b> El agua turbia entra por arriba. La capa protectora estabiliza el flujo, mientras que el carbón activado remueve toxinas y malos olores.";
        });

        document.getElementById('btn-reset').addEventListener('click', () => {
            isFiltering = false;
            waterProgress = 0;
            water.position.y = 2.2;
            waterMat.color.setHex(0x4fc3f7);
            waterMat.opacity = 0.0;
            infoText.innerHTML = "Selecciona una acción para ver cómo interactúa el agua con los materiales porosos y de retención física.";
        });

        // Evento Raycaster para hacer clic en las capas y ver qué hacen
        const raycaster = new THREE.Raycaster();
        const mouse = new THREE.Vector2();

        window.addEventListener('click', (e) => {
            // Ajustar coordenadas de mouse según tamaño del canvas técnico
            const rect = renderer.domElement.getBoundingClientRect();
            mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
            mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;

            raycaster.setFromCamera(mouse, camera);
            const intersects = raycaster.intersectObjects(layersMeshes);

            if (intersects.length > 0) {
                const clickedLayer = intersects[0].object;
                infoText.innerHTML = `🔎 <b>${clickedLayer.userData.name}:</b> ${clickedLayer.userData.info}`;
            }
        });

        // Bucle de renderizado
        function animate() {
            requestAnimationFrame(animate);
            controls.update();

            // Rotación automática muy suave cuando no se interactúa
            if(!controls.state == -1) {
                filterGroup.rotation.y += 0.003;
            }

            // Simulación del agua bajando por gravedad
            if (isFiltering && waterProgress < 1) {
                waterProgress += 0.002;
                water.position.y -= 0.009;
                
                // El agua cambia de color gradualmente (se vuelve más limpia/clara) al pasar el carbón
                if(water.position.y < 0.5) {
