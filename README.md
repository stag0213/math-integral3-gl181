<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D立体可視化: x²+y²≤1, 0≤z≤x²</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Three.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <!-- OrbitControls -->
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <!-- FontAwesome icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- MathJax for rendering math expressions -->
    <script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
    <script id="mathjax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #0f172a;
            color: #f8fafc;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        #canvas-container {
            width: 100vw;
            height: 100vh;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
        }
        .glass-panel {
            background: rgba(30, 41, 59, 0.75);
            backdrop-filter: blur(12px);
            -webkit-backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.125);
        }
    </style>
</head>
<body class="bg-slate-900 text-slate-100 select-none">

    <div id="canvas-container"></div>

    <!-- Top Left Title and Formula Panel -->
    <div class="absolute top-4 left-4 z-10 glass-panel p-5 rounded-2xl max-w-sm shadow-2xl transition-all duration-300">
        <h1 class="text-xl font-bold text-cyan-400 mb-2 flex items-center gap-2">
            <i class="fa-solid fa-cube"></i> 立体の3D可視化
        </h1>
        <div class="text-sm space-y-2 text-slate-300 border-t border-slate-700/60 pt-3">
            <p class="font-medium text-slate-200">【領域の条件】</p>
            <div class="bg-slate-800/80 p-3 rounded-lg border border-slate-700 font-mono text-center text-cyan-300">
                \( x^2 + y^2 \le 1 \)<br>
                \( 0 \le z \le x^2 \)
            </div>
            <p class="text-xs text-slate-400 leading-relaxed pt-1">
                底面は円 \(x^2+y^2 \le 1\)、下面は \(z=0\)、上面は放物柱面 \(z=x^2\) で囲まれた立体です。
            </p>
        </div>
    </div>

    <!-- Top Right Features & Properties -->
    <div class="absolute top-4 right-4 z-10 glass-panel p-4 rounded-2xl max-w-xs shadow-2xl text-xs space-y-3 hidden sm:block">
        <h2 class="font-semibold text-slate-200 border-b border-slate-700 pb-2 flex items-center justify-between">
            <span><i class="fa-solid fa-calculator text-cyan-400"></i> 数学的性質</span>
        </h2>
        <div class="space-y-1.5 text-slate-300">
            <div class="flex justify-between items-center bg-slate-800/50 p-2 rounded">
                <span class="text-slate-400">体積 \(V\):</span>
                <span class="font-mono text-cyan-300 font-bold">\(\frac{\pi}{4} \approx 0.785\)</span>
            </div>
            <div class="text-slate-400 text-[11px] leading-snug">
                \(V = \iint_{x^2+y^2\le 1} x^2 \, dxdy = \frac{\pi}{4}\)
            </div>
        </div>
    </div>

    <!-- Bottom Center Interactive Controls Panel -->
    <div class="absolute bottom-6 left-1/2 transform -translate-x-1/2 z-10 glass-panel px-6 py-4 rounded-3xl shadow-2xl flex flex-wrap items-center justify-center gap-6 max-w-[95vw] sm:max-w-none">
        <!-- Display Toggles -->
        <div class="flex items-center gap-4 border-r border-slate-700/80 pr-4">
            <label class="flex items-center gap-2 text-xs cursor-pointer hover:text-cyan-300 transition">
                <input type="checkbox" id="toggle-solid" checked class="rounded accent-cyan-500 w-4 h-4">
                <span>立体本体</span>
            </label>
            <label class="flex items-center gap-2 text-xs cursor-pointer hover:text-cyan-300 transition">
                <input type="checkbox" id="toggle-parabola" checked class="rounded accent-purple-500 w-4 h-4">
                <span>放物柱 \(z=x^2\)</span>
            </label>
            <label class="flex items-center gap-2 text-xs cursor-pointer hover:text-cyan-300 transition">
                <input type="checkbox" id="toggle-cylinder" checked class="rounded accent-blue-500 w-4 h-4">
                <span>円柱ガイド</span>
            </label>
        </div>

        <!-- Opacity Slider -->
        <div class="flex items-center gap-3 text-xs border-r border-slate-700/80 pr-4">
            <span class="text-slate-400">不透明度:</span>
            <input type="range" id="opacity-slider" min="0.2" max="1.0" step="0.05" value="0.85" class="w-24 accent-cyan-400 cursor-pointer">
        </div>

        <!-- Slice / Animation Control -->
        <div class="flex items-center gap-3 text-xs">
            <span class="text-slate-400">断面切断:</span>
            <input type="range" id="slice-slider" min="-1" max="1" step="0.02" value="1" class="w-24 accent-pink-500 cursor-pointer">
            <button id="btn-animate" class="px-3 py-1.5 rounded-lg bg-cyan-600 hover:bg-cyan-500 text-white font-medium text-xs transition flex items-center gap-1 shadow-md">
                <i class="fa-solid fa-play" id="play-icon"></i>
                <span id="play-text">アニメーション</span>
            </button>
            <button id="btn-reset" class="px-2.5 py-1.5 rounded-lg bg-slate-700 hover:bg-slate-600 text-slate-200 transition text-xs" title="カメラ位置をリセット">
                <i class="fa-solid fa-rotate-right"></i>
            </button>
        </div>
    </div>

    <!-- Navigation / Help tooltip -->
    <div class="absolute bottom-4 right-4 z-10 text-[11px] text-slate-500 glass-panel px-3 py-1.5 rounded-lg hidden md:block">
        ドラッグ: 回転 | 右ドラッグ: 移動 | ホイール: 拡大縮小
    </div>

    <script>
        let scene, camera, renderer, controls;
        let solidMesh, parabolaMesh, cylinderMesh, planeCutMesh, edgesMesh;
        let isAnimating = false;
        let animationId = null;

        // Colors
        const COLOR_SOLID = 0x06b6d4;      // Cyan
        const COLOR_PARABOLA = 0xa855f7;   // Purple
        const COLOR_CYLINDER = 0x3b82f6;   // Blue
        const COLOR_EDGES = 0xffffff;      // White wireframe
        const COLOR_SLICE = 0xec4899;      // Pink

        // Local clipping plane definition
        const localClippingPlane = new THREE.Plane(new THREE.Vector3(0, 0, 1), 1);

        function init() {
            const container = document.getElementById('canvas-container');

            // Scene setup
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x0f172a);

            // Camera setup
            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 100);
            camera.position.set(2.5, 2.0, 3.0);

            // Renderer setup
            renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            renderer.localClippingEnabled = true;
            container.appendChild(renderer.domElement);

            // Controls setup
            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.enableDamping = true;
            controls.dampingFactor = 0.05;
            controls.target.set(0, 0.4, 0);

            // Lighting setup
            setupLighting();

            // Coordinate System Setup (Axes & Grid)
            setupAxesAndGrid();

            // Generate Geometries for the Specific Region
            createSolidMesh();
            createParabolaGuide();
            createCylinderGuide();

            // Event Listeners
            window.addEventListener('resize', onWindowResize);
            setupUIEventListeners();

            // Initial render call
            animate();
        }

        function setupLighting() {
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
            scene.add(ambientLight);

            const dirLight1 = new THREE.DirectionalLight(0xffffff, 0.8);
            dirLight1.position.set(5, 10, 7);
            dirLight1.castShadow = true;
            scene.add(dirLight1);

            const dirLight2 = new THREE.DirectionalLight(0x38bdf8, 0.4);
            dirLight2.position.set(-5, -2, -5);
            scene.add(dirLight2);

            const pointLight = new THREE.PointLight(0xffffff, 0.5, 10);
            pointLight.position.set(0, 3, 0);
            scene.add(pointLight);
        }

        function setupAxesAndGrid() {
            // Main Grid on XY plane / XZ plane
            const gridHelper = new THREE.GridHelper(4, 20, 0x475569, 0x1e293b);
            gridHelper.position.y = 0;
            scene.add(gridHelper);

            // Axes
            const axesGroup = new THREE.Group();

            // Custom colored axis lines with labels
            const axisLength = 1.8;

            // X Axis (Red)
            const xMat = new THREE.LineBasicMaterial({ color: 0xef4444, linewidth: 2 });
            const xGeo = new THREE.BufferGeometry().setFromPoints([new THREE.Vector3(0,0,0), new THREE.Vector3(axisLength,0,0)]);
            axesGroup.add(new THREE.Line(xGeo, xMat));

            // Y Axis (Green) - Note: In math, Z is usually height. Here Three.js Y is UP, so we map:
            // Three.js X = Math X
            // Three.js Z = Math Y (negated for right hand)
            // Three.js Y = Math Z (height)
            const zMat = new THREE.LineBasicMaterial({ color: 0x22c55e, linewidth: 2 }); // Math Y
            const zGeo = new THREE.BufferGeometry().setFromPoints([new THREE.Vector3(0,0,0), new THREE.Vector3(0,0,-axisLength)]);
            axesGroup.add(new THREE.Line(zGeo, zMat));

            // Z Axis (Height) (Blue)
            const yMat = new THREE.LineBasicMaterial({ color: 0x3b82f6, linewidth: 2 }); // Math Z
            const yGeo = new THREE.BufferGeometry().setFromPoints([new THREE.Vector3(0,0,0), new THREE.Vector3(0,axisLength,0)]);
            axesGroup.add(new THREE.Line(yGeo, yMat));

            scene.add(axesGroup);

            // Canvas Text Labels for Axes
            createAxisLabel('X', axisLength + 0.1, 0, 0, '#ef4444');
            createAxisLabel('Y', 0, 0, -(axisLength + 0.1), '#22c55e');
            createAxisLabel('Z', 0, axisLength + 0.1, 0, '#3b82f6');
        }

        function createAxisLabel(text, x, y, z, color) {
            const canvas = document.createElement('canvas');
            canvas.width = 64;
            canvas.height = 64;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = color;
            ctx.font = 'Bold 48px sans-serif';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText(text, 32, 32);

            const texture = new THREE.CanvasTexture(canvas);
            const spriteMaterial = new THREE.SpriteMaterial({ map: texture });
            const sprite = new THREE.Sprite(spriteMaterial);
            sprite.position.set(x, y, z);
            sprite.scale.set(0.3, 0.3, 0.3);
            scene.add(sprite);
        }

        function createSolidMesh() {
            // We construct a customParametric/BufferGeometry for the solid region
            // Region: x^2 + y^2 <= 1, 0 <= z <= x^2
            // Math coordinates: (x, y, z) -> Three.js coordinates: (x, z, -y)

            const N_THETA = 80;
            const N_R = 30;

            const vertices = [];
            const indices = [];
            const uvs = [];

            // Generate vertices for Top Surface (z = x^2), Bottom Surface (z = 0), and Side Wall (x^2+y^2=1)
            // 1. Top Surface (z = x^2)
            // 2. Bottom Surface (z = 0)
            // 3. Side Surface (r = 1, 0 <= z <= x^2)

            const topOffset = 0;
            const bottomOffset = (N_THETA + 1) * (N_R + 1);

            // Helper to get math coordinates
            function getPoint(r, theta) {
                const x = r * Math.cos(theta);
                const y = r * Math.sin(theta);
                const z = x * x; // Surface z = x^2
                return { x, y, z };
            }

            // --- Top Surface Mesh ---
            for (let i = 0; i <= N_THETA; i++) {
                const theta = (i / N_THETA) * Math.PI * 2;
                for (let j = 0; j <= N_R; j++) {
                    const r = j / N_R;
                    const p = getPoint(r, theta);
                    // Map to Three.js: (x, z, -y)
                    vertices.push(p.x, p.z, -p.y);
                    uvs.push(i / N_THETA, j / N_R);
                }
            }

            for (let i = 0; i < N_THETA; i++) {
                for (let j = 0; j < N_R; j++) {
                    const first = i * (N_R + 1) + j;
                    const second = first + N_R + 1;

                    indices.push(first, first + 1, second);
                    indices.push(second, first + 1, second + 1);
                }
            }

            // --- Bottom Surface Mesh (z = 0) ---
            for (let i = 0; i <= N_THETA; i++) {
                const theta = (i / N_THETA) * Math.PI * 2;
                for (let j = 0; j <= N_R; j++) {
                    const r = j / N_R;
                    const x = r * Math.cos(theta);
                    const y = r * Math.sin(theta);
                    vertices.push(x, 0, -y);
                    uvs.push(i / N_THETA, j / N_R);
                }
            }

            for (let i = 0; i < N_THETA; i++) {
                for (let j = 0; j < N_R; j++) {
                    const first = bottomOffset + i * (N_R + 1) + j;
                    const second = first + N_R + 1;

                    // Reverse winding order for bottom face normals
                    indices.push(first, second, first + 1);
                    indices.push(second, second + 1, first + 1);
                }
            }

            // --- Side Wall Surface (r = 1, 0 <= z <= x^2) ---
            const sideOffset = bottomOffset + (N_THETA + 1) * (N_R + 1);
            const N_Z = 20;

            for (let i = 0; i <= N_THETA; i++) {
                const theta = (i / N_THETA) * Math.PI * 2;
                const x = Math.cos(theta);
                const y = Math.sin(theta);
                const zMax = x * x;

                for (let k = 0; k <= N_Z; k++) {
                    const z = (k / N_Z) * zMax;
                    vertices.push(x, z, -y);
                    uvs.push(i / N_THETA, k / N_Z);
                }
            }

            for (let i = 0; i < N_THETA; i++) {
                for (let k = 0; k < N_Z; k++) {
                    const first = sideOffset + i * (N_Z + 1) + k;
                    const second = first + N_Z + 1;

                    indices.push(first, first + 1, second);
                    indices.push(second, first + 1, second + 1);
                }
            }

            const geometry = new THREE.BufferGeometry();
            geometry.setAttribute('position', new THREE.Float32BufferAttribute(vertices, 3));
            geometry.setAttribute('uv', new THREE.Float32BufferAttribute(uvs, 2));
            geometry.setIndex(indices);
            geometry.computeVertexNormals();

            // Material
            const material = new THREE.MeshPhongMaterial({
                color: COLOR_SOLID,
                transparent: true,
                opacity: 0.85,
                side: THREE.DoubleSide,
                shininess: 90,
                specular: 0x444444,
                clippingPlanes: [localClippingPlane],
                clipShadows: true
            });

            solidMesh = new THREE.Mesh(geometry, material);
            scene.add(solidMesh);

            // Wireframe / Edges for clear boundary rendering
            const edges = new THREE.EdgesGeometry(geometry, 15);
            const lineMat = new THREE.LineBasicMaterial({ color: COLOR_EDGES, transparent: true, opacity: 0.5 });
            edgesMesh = new THREE.LineSegments(edges, lineMat);
            solidMesh.add(edgesMesh);
        }

        function createParabolaGuide() {
            // Extended surface for z = x^2 (-1.2 <= x, y <= 1.2)
            const geometry = new THREE.PlaneGeometry(2.4, 2.4, 40, 40);
            const pos = geometry.attributes.position;

            for (let i = 0; i < pos.count; i++) {
                const x = pos.getX(i);
                const y = pos.getY(i);
                // Grid plane is XY in geometry. We transform z = x^2
                // Geometry Z attribute becomes Math Z = x^2
                pos.setZ(i, x * x);
            }
            geometry.computeVertexNormals();

            // Re-orient to Three.js coordinate system (Math z = Three Y, Math y = -Three Z)
            geometry.rotateX(-Math.PI / 2);

            const material = new THREE.MeshPhongMaterial({
                color: COLOR_PARABOLA,
                transparent: true,
                opacity: 0.25,
                side: THREE.DoubleSide,
                wireframe: true
            });

            parabolaMesh = new THREE.Mesh(geometry, material);
            scene.add(parabolaMesh);
        }

        function createCylinderGuide() {
            // Full cylinder x^2 + y^2 = 1, z from 0 to 1
            const geometry = new THREE.CylinderGeometry(1, 1, 1.2, 48, 1, true);
            // Center geometry so bottom is at z=0
            geometry.translate(0, 0.6, 0);

            const material = new THREE.MeshPhongMaterial({
                color: COLOR_CYLINDER,
                transparent: true,
                opacity: 0.15,
                side: THREE.DoubleSide,
                wireframe: true
            });

            cylinderMesh = new THREE.Mesh(geometry, material);
            scene.add(cylinderMesh);
        }

        function setupUIEventListeners() {
            // Toggle Solid
            document.getElementById('toggle-solid').addEventListener('change', (e) => {
                solidMesh.visible = e.target.checked;
            });

            // Toggle Parabola
            document.getElementById('toggle-parabola').addEventListener('change', (e) => {
                parabolaMesh.visible = e.target.checked;
            });

            // Toggle Cylinder
            document.getElementById('toggle-cylinder').addEventListener('change', (e) => {
                cylinderMesh.visible = e.target.checked;
            });

            // Opacity Slider
            document.getElementById('opacity-slider').addEventListener('input', (e) => {
                const val = parseFloat(e.target.value);
                solidMesh.material.opacity = val;
            });

            // Slice Slider (Clipping)
            const sliceSlider = document.getElementById('slice-slider');
            sliceSlider.addEventListener('input', (e) => {
                updateClipping(parseFloat(e.target.value));
            });

            // Animation Toggle Button
            const btnAnimate = document.getElementById('btn-animate');
            btnAnimate.addEventListener('click', () => {
                isAnimating = !isAnimating;
                const icon = document.getElementById('play-icon');
                const text = document.getElementById('play-text');

                if (isAnimating) {
                    icon.className = "fa-solid fa-pause";
                    text.textContent = "停止";
                } else {
                    icon.className = "fa-solid fa-play";
                    text.textContent = "アニメーション";
                }
            });

            // Reset Camera Button
            document.getElementById('btn-reset').addEventListener('click', () => {
                camera.position.set(2.5, 2.0, 3.0);
                controls.target.set(0, 0.4, 0);
                controls.update();
            });
        }

        function updateClipping(val) {
            // Slice along Math Y axis (Three.js Z axis)
            localClippingPlane.constant = val;
        }

        let animClock = 0;

        function animate() {
            requestAnimationFrame(animate);

            if (isAnimating) {
                animClock += 0.015;
                const sliceVal = Math.sin(animClock);
                document.getElementById('slice-slider').value = sliceVal;
                updateClipping(sliceVal);
            }

            controls.update();
            renderer.render(scene, camera);
        }

        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        // Initialize when window loads
        window.onload = function() {
            init();
        };
    </script>
</body>
</html>
