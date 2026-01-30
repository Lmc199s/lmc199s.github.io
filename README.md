<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Modelo 3D de Hibridación del Carbono - VR Ready</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/utils/BufferGeometryUtils.min.js"></script>
    
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        /* (Se mantiene todo tu estilo CSS original) */
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Segoe UI', sans-serif; }
        body { background: linear-gradient(135deg, #e3f2fd 0%, #f3e5f5 100%); color: #333; min-height: 100vh; overflow-x: hidden; }
        .container { max-width: 1800px; margin: 0 auto; padding: 20px; }
        header { text-align: center; margin-bottom: 25px; padding: 30px; background: rgba(255, 255, 255, 0.9); border-radius: 20px; box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1); border: 1px solid rgba(255, 255, 255, 0.8); }
        h1 { font-size: 3.2rem; margin-bottom: 15px; background: linear-gradient(90deg, #2196F3, #9C27B0, #FF9800); -webkit-background-clip: text; background-clip: text; color: transparent; }
        .subtitle { font-size: 1.3rem; color: #666; max-width: 900px; margin: 0 auto; line-height: 1.7; }
        .main-content { display: grid; grid-template-columns: 1fr 1.8fr; gap: 30px; margin-bottom: 30px; }
        @media (max-width: 1200px) { .main-content { grid-template-columns: 1fr; } }
        .control-panel { background: rgba(255, 255, 255, 0.95); border-radius: 20px; padding: 25px; box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1); display: flex; flex-direction: column; gap: 25px; max-height: 750px; overflow-y: auto; }
        .visualization-area { background: rgba(255, 255, 255, 0.95); border-radius: 20px; overflow: hidden; box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1); position: relative; min-height: 700px; }
        #canvas-container { width: 100%; height: 100%; }
        .section-title { font-size: 1.6rem; margin-bottom: 20px; color: #2196F3; display: flex; align-items: center; gap: 12px; padding-bottom: 10px; border-bottom: 2px solid rgba(33, 150, 243, 0.2); }
        .hybridization-selector { display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 15px; }
        .hybrid-card { background: white; border-radius: 15px; padding: 20px; cursor: pointer; transition: 0.3s; border: 2px solid #e0e0e0; text-align: center; }
        .hybrid-card.active { background: rgba(33, 150, 243, 0.1); border-color: #2196F3; }
        .control-group { background: rgba(255, 255, 255, 0.8); border-radius: 15px; padding: 20px; }
        .control-item { display: flex; align-items: center; justify-content: space-between; margin-bottom: 18px; }
        .toggle-switch { position: relative; width: 70px; height: 34px; }
        .toggle-switch input { opacity: 0; width: 0; height: 0; }
        .toggle-slider { position: absolute; cursor: pointer; top: 0; left: 0; right: 0; bottom: 0; background-color: #e0e0e0; border-radius: 34px; transition: .4s; }
        .toggle-slider:before { position: absolute; content: ""; height: 26px; width: 26px; left: 4px; bottom: 4px; background-color: white; border-radius: 50%; transition: .4s; }
        input:checked + .toggle-slider { background-color: #2196F3; }
        input:checked + .toggle-slider:before { transform: translateX(36px); }
        .slider-container { display: flex; align-items: center; gap: 15px; width: 100%; }
        .info-display { background: rgba(255, 255, 255, 0.9); border-radius: 15px; padding: 25px; margin-top: 10px; }
        .visualization-controls { position: absolute; bottom: 25px; right: 25px; display: flex; flex-direction: column; gap: 15px; z-index: 100; }
        .vis-control-btn { width: 50px; height: 50px; border-radius: 50%; background: white; border: 2px solid #e0e0e0; cursor: pointer; display: flex; align-items: center; justify-content: center; transition: 0.3s; }
        .instructions { position: absolute; bottom: 25px; left: 25px; background: rgba(255, 255, 255, 0.95); padding: 15px; border-radius: 15px; max-width: 350px; z-index: 100; border-left: 4px solid #2196F3; }
        .loading-screen { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: white; display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 200; }
        footer { text-align: center; padding: 25px; margin-top: 30px; border-top: 1px solid #ddd; }
        
        /* Estilo para el botón VR nativo de Three.js que inyectaremos */
        #VRButton {
            position: absolute !important;
            bottom: 25px !important;
            left: 50% !important;
            transform: translateX(-50%) !important;
            background: #2196F3 !important;
            border: none !important;
            padding: 12px 24px !important;
            font-weight: bold !important;
            text-transform: uppercase !important;
        }
    </style>
</head>
<body>
    <div class="loading-screen" id="loading-screen">
        <div style="width: 60px; height: 60px; border: 5px solid #eee; border-top-color: #2196F3; border-radius: 50%; animation: spin 1s linear infinite;"></div>
        <p style="margin-top: 20px; color: #2196F3;">Cargando experiencia VR...</p>
    </div>
    
    <div class="container">
        <header>
            <h1><i class="fas fa-atom"></i> Modelo 3D Hibridación del Carbono</h1>
            <p class="subtitle">Modo VR compatible con lentes de celular. Haz clic en "ENTER VR" para comenzar.</p>
        </header>
        
        <div class="main-content">
            <div class="control-panel">
                <h2 class="section-title"><i class="fas fa-sliders-h"></i> Hibridación</h2>
                <div class="hybridization-selector">
                    <div class="hybrid-card sp3 active" data-type="sp3"><b>sp³</b><br><small>Tetraédrica</small></div>
                    <div class="hybrid-card sp2" data-type="sp2"><b>sp²</b><br><small>Trigonal</small></div>
                    <div class="hybrid-card sp" data-type="sp"><b>sp</b><br><small>Lineal</small></div>
                </div>

                <div class="control-group">
                    <div class="control-item">
                        <span>Rotación automática</span>
                        <div class="toggle-switch"><input type="checkbox" id="rotate-toggle" checked><label class="toggle-slider"></label></div>
                    </div>
                </div>

                <div class="info-display">
                    <h3 id="info-title">Hibridación sp³</h3>
                    <div id="info-content">Cargando datos...</div>
                </div>
            </div>
            
            <div class="visualization-area">
                <div id="canvas-container"></div>
                <div class="instructions">
                    <p><i class="fas fa-vr-cardboard"></i> <b>Modo VR:</b> Activa el botón inferior y coloca el móvil en tus lentes.</p>
                </div>
            </div>
        </div>
    </div>

    <script type="module">
        // Importamos el botón VR oficial de Three.js desde un CDN compatible con módulos
        import { VRButton } from 'https://cdn.jsdelivr.net/npm/three@0.128.0/examples/jsm/webxr/VRButton.js';

        let scene, camera, renderer, controls, nucleus;
        let hybridOrbitals = [], pOrbitals = [], electrons = [];
        let currentHybridization = 'sp3';
        let config = { autoRotate: true, rotationSpeed: 0.002, orbitalDistance: 1.0, electronSpeed: 1.0 };
        let colors = { nucleus: 0xE91E63, sp3: 0x2196F3, sp2: 0x9C27B0, sp: 0xFF9800, pOrbital: 0xFF4081, electron: 0x00BCD4, backgroundColor: 0x111111 };

        const hybridizationData = {
            sp3: { title: "sp³", positions: [{x:1,y:1,z:1},{x:-1,y:-1,z:1},{x:-1,y:1,z:-1},{x:1,y:-1,z:-1}], pPos: [] },
            sp2: { title: "sp²", positions: [{x:1,y:0,z:0},{x:-0.5,y:0.86,z:0},{x:-0.5,y:-0.86,z:0}], pPos: [{axis:'z'}] },
            sp: { title: "sp", positions: [{x:1,y:0,z:0},{x:-1,y:0,z:0}], pPos: [{axis:'y'},{axis:'z'}] }
        };

        function init() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x050505); // Fondo oscuro para mejor efecto VR

            const container = document.getElementById('canvas-container');
            camera = new THREE.PerspectiveCamera(70, container.clientWidth / container.clientHeight, 0.1, 1000);
            camera.position.set(0, 1.6, 3); // Altura promedio de ojos en VR (1.6m)

            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setPixelRatio(window.devicePixelRatio);
            renderer.setSize(container.clientWidth, container.clientHeight);
            
            // --- CONFIGURACIÓN VR ---
            renderer.xr.enabled = true; 
            document.querySelector('.visualization-area').appendChild(VRButton.createButton(renderer));
            // ------------------------

            container.appendChild(renderer.domElement);

            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.target.set(0, 1.6, 0); // Mirar al átomo a la altura de los ojos

            const light = new THREE.HemisphereLight(0xffffff, 0x444444, 1);
            scene.add(light);

            createCarbonAtom();
            showHybridization('sp3');

            // Cambio de bucle para VR
            renderer.setAnimationLoop(animate);
            
            document.getElementById('loading-screen').style.display = 'none';
        }

        function createCarbonAtom() {
            const geo = new THREE.SphereGeometry(0.2, 32, 32);
            const mat = new THREE.MeshPhongMaterial({ color: colors.nucleus });
            nucleus = new THREE.Mesh(geo, mat);
            nucleus.position.y = 1.6; // Posicionar el átomo frente al usuario en VR
            scene.add(nucleus);
        }

        function showHybridization(type) {
            // Limpieza básica
            hybridOrbitals.forEach(o => scene.remove(o));
            pOrbitals.forEach(o => scene.remove(o));
            hybridOrbitals = []; pOrbitals = [];

            const data = hybridizationData[type];
            const color = colors[type];

            data.positions.forEach(pos => {
                const geo = new THREE.SphereGeometry(0.15, 16, 16);
                const mat = new THREE.MeshPhongMaterial({ color: color, transparent: true, opacity: 0.7 });
                const orb = new THREE.Mesh(geo, mat);
                // Escalar posición
                orb.position.set(pos.x * 0.8, (pos.y * 0.8) + 1.6, pos.z * 0.8);
                scene.add(orb);
                hybridOrbitals.push(orb);
            });
            
            document.getElementById('info-title').innerText = "Hibridación " + type;
        }

        function animate() {
            if (config.autoRotate) {
                scene.rotation.y += config.rotationSpeed;
            }
            controls.update();
            renderer.render(scene, camera);
        }

        // Eventos de botones
        document.querySelectorAll('.hybrid-card').forEach(card => {
            card.addEventListener('click', () => {
                document.querySelectorAll('.hybrid-card').forEach(c => c.classList.remove('active'));
                card.classList.add('active');
                showHybridization(card.dataset.type);
            });
        });

        document.getElementById('rotate-toggle').addEventListener('change', (e) => {
            config.autoRotate = e.target.checked;
        });

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        init();
    </script>
</body>
</html>
