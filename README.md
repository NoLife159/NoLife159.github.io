<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chotchayut Biewbangkoed - 3D Portfolio</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body, html {
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #05050d;
            color: #ffffff;
        }

        #canvas-container {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 1;
        }

        /* Overlay UI Header */
        .ui-container {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 2;
            pointer-events: none;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            padding: 40px;
        }

        .header {
            pointer-events: auto;
        }

        .header h1 {
            font-size: 2.8rem;
            font-weight: 800;
            letter-spacing: 2px;
            background: linear-gradient(45deg, #00ffff, #8a2be2);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-transform: uppercase;
        }

        .header p {
            font-size: 1.2rem;
            color: #a0a0c0;
            margin-top: 5px;
        }

        .instructions {
            align-self: flex-start;
            background: rgba(255, 255, 255, 0.05);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            padding: 15px 25px;
            border-radius: 12px;
            pointer-events: auto;
        }

        .instructions p {
            font-size: 0.9rem;
            color: #d0d0e0;
            margin-bottom: 5px;
        }

        .instructions p strong {
            color: #00ffff;
        }

        /* Loading Screen */
        #loader {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: #05050d;
            z-index: 99;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            transition: opacity 0.8s ease;
        }

        .spinner {
            width: 50px;
            height: 50px;
            border: 3px solid rgba(0, 255, 255, 0.2);
            border-radius: 50%;
            border-top-color: #00ffff;
            animation: spin 1s ease-in-out infinite;
        }

        @keyframes spin {
            to { transform: rotate(360deg); }
        }

        #loader p {
            margin-top: 20px;
            font-size: 1.1rem;
            letter-spacing: 1px;
            color: #8a2be2;
        }
    </style>

    <!-- Import Three.js และโมดูลที่เกี่ยวข้องผ่าน CDN -->
    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
            }
        }
    </script>
</head>
<body>

    <div id="loader">
        <div class="spinner"></div>
        <p>กำลังโหลด 3D Assets & Shader...</p>
    </div>

    <div class="ui-container">
        <div class="header">
            <h1>Chotchayut Biewbangkoed</h1>
            <p>Interactive 3D Graphics & Shader Portfolio</p>
        </div>
        
        <div class="instructions">
            <p><strong>🖱️ Drag Mouse:</strong> หมุนมุมกล้องรอบโมเดล PBR (OrbitControls)</p>
            <p><strong>✨ Move Cursor:</strong> ปฏิสัมพันธ์กับ Shader ก้อนเมฆหมอกบนท้องฟ้า</p>
            <p><strong>🔍 Scroll:</strong> ขยาย / ย่อ มุมมอง</p>
        </div>
    </div>

    <div id="canvas-container"></div>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
        import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
        import { RGBELoader } from 'three/addons/loaders/RGBELoader.js';

        // --- 1. SET UP SCENE, CAMERA, RENDERER ---
        const container = document.getElementById('canvas-container');
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x05050d, 0.015);

        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0, 2, 8);

        const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        renderer.toneMapping = THREE.ACESFilmicToneMapping;
        renderer.toneMappingExposure = 1.2;
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        container.appendChild(renderer.domElement);

        // --- 2. CONTROLS ---
        const controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;
        controls.maxPolarAngle = Math.PI / 2 + 0.1; // จำกัดไม่ให้หมุนลงใต้พื้นดินมากเกินไป
        controls.minDistance = 3;
        controls.maxDistance = 20;

        // --- 3. LIGHTING (สำหรับ PBR Material) ---
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
        scene.add(ambientLight);

        const dirLight = new THREE.DirectionalLight(0x00ffff, 2.5);
        dirLight.position.set(5, 10, 7);
        dirLight.castShadow = true;
        dirLight.shadow.mapSize.width = 2048;
        dirLight.shadow.mapSize.height = 2048;
        scene.add(dirLight);

        const pointLight = new THREE.PointLight(0x8a2be2, 4, 15);
        pointLight.position.set(-5, 2, -3);
        scene.add(pointLight);

        // --- 4. ADVANCED SHADER (INTERACTIVE CLOUD SYSTEM) ---
        // GLSL Shader code สำหรับสร้างก้อนเมฆหมอกแบบ Volumetric 3D Noise ที่ตอบสนองกับ Mouse
        const cloudVertexShader = `
            varying vec2 vUv;
            varying vec3 vWorldPosition;
            
            void main() {
                vUv = uv;
                vec4 worldPosition = modelMatrix * vec4(position, 1.0);
                vWorldPosition = worldPosition.xyz;
                gl_Position = projectionMatrix * viewMatrix * worldPosition;
            }
        `;

        const cloudFragmentShader = `
            uniform float uTime;
            uniform vec2 uMouse;
            varying vec2 vUv;
            varying vec3 vWorldPosition;

            // Simplex / Perlin Noise Functions
            vec3 mod289(vec3 x) { return x - floor(x * (1.0 / 289.0)) * 289.0; }
            vec4 mod289(vec4 x) { return x - floor(x * (1.0 / 289.0)) * 289.0; }
            vec4 permute(vec4 x) { return mod289(((x*34.0)+1.0)*x); }
            vec4 taylorInvSqrt(vec4 r) { return 1.79284291400159 - 0.85373472095314 * r; }

            float snoise(vec3 v) {
                const vec2 C = vec2(1.0/6.0, 1.0/3.0);
                const vec4 D = vec4(0.0, 0.5, 1.0, 2.0);
                vec3 i  = floor(v + dot(v, C.yyy));
                vec3 x0 = v - i + dot(i, C.xxx);
                vec3 g = step(x0.yzx, x0.xyz);
                vec3 l = 1.0 - g;
                vec3 i1 = min(g.xyz, l.zxy);
                vec3 i2 = max(g.xyz, l.zxy);
                vec3 x1 = x0 - i1 + C.xxx;
                vec3 x2 = x0 - i2 + C.yyy;
                vec3 x3 = x0 - D.yyy;
                i = mod289(i);
                vec4 p = permute(permute(permute(
                            i.z + vec4(0.0, i1.z, i2.z, 1.0))
                        + i.y + vec4(0.0, i1.y, i2.y, 1.0))
                        + i.x + vec4(0.0, i1.x, i2.x, 1.0));
                float n_ = 0.142857142857;
                vec3 ns = n_ * D.wyz - D.xzx;
                vec4 j = p - 49.0 * floor(p * ns.z);
                vec4 x_ = floor(j * ns.z);
                vec4 y_ = floor(j - 7.0 * x_);
                vec4 x = x_ *ns.x + ns.yyyy;
                vec4 y = y_ *ns.x + ns.yyyy;
                vec4 h = 1.0 - abs(x) - abs(y);
                vec4 b0 = vec4(x.xy, y.xy);
                vec4 b1 = vec4(x.zw, y.zw);
                vec4 s0 = floor(b0)*2.0 + 1.0;
                vec4 s1 = floor(b1)*2.0 + 1.0;
                vec4 sh = -step(h, vec4(0.0));
                vec4 a0 = b0.xzyw + s0.xzyw*sh.xxyy;
                vec4 a1 = b1.xzyw + s1.xzyw*sh.zzww;
                vec3 p0 = vec3(a0.xy, h.x);
                vec3 p1 = vec3(a0.zw, h.y);
                vec3 p2 = vec3(a1.xy, h.z);
                vec3 p3 = vec3(a1.zw, h.w);
                vec4 norm = taylorInvSqrt(vec4(dot(p0,p0), dot(p1,p1), dot(p2, p2), dot(p3,p3)));
                p0 *= norm.x; p1 *= norm.y; p2 *= norm.z; p3 *= norm.w;
                vec4 m = max(0.6 - vec4(dot(x0,x0), dot(x1,x1), dot(x2,x2), dot(x3,x3)), 0.0);
                m = m * m;
                return 42.0 * dot(m*m, vec4(dot(p0,x0), dot(p1,x1), dot(p2,x2), dot(p3,x3)));
            }

            // Fractal Brownian Motion
            float fbm(vec3 p) {
                float value = 0.0;
                float amplitude = 0.5;
                for (int i = 0; i < 4; i++) {
                    value += amplitude * snoise(p);
                    p *= 2.0;
                    amplitude *= 0.5;
                }
                return value;
            }

            void main() {
                // คำนวณระยะห่างตำแหน่งของ Mouse กับ Vertex ของ Shader
                vec2 mouseWorld = uMouse * 10.0;
                float mouseDist = distance(vWorldPosition.xz, mouseWorld);
                
                // สร้างแรงผลักดัน/การรบกวน (Displacement) จากตำแหน่งของ Mouse
                float mouseEffect = smoothstep(4.0, 0.0, mouseDist);
                
                // เลื่อนพิกัด Noise ตามเวลาและการขยับของ Mouse
                vec3 q = vWorldPosition * 0.15;
                q.y += uTime * 0.05;
                q.x += mouseEffect * 0.5;

                // คำนวณความหนาแน่นของเมฆ (Cloud Density)
                float density = fbm(q + vec3(uTime * 0.02, 0.0, 0.0));
                density += mouseEffect * 0.3; // เมฆจะฟุ้งและสว่างขึ้นเมื่อเม้าส์เคลื่อนผ่าน

                // การผสมสี Shader (Gradient สีฟ้า Neon คละกับสีม่วง Cyberpunk)
                vec3 colorCloud = mix(vec3(0.05, 0.05, 0.2), vec3(0.0, 0.8, 1.0), density);
                colorCloud = mix(colorCloud, vec3(0.6, 0.1, 0.9), mouseEffect);

                float alpha = smoothstep(0.1, 0.7, density) * 0.6;
                gl_FragColor = vec4(colorCloud, alpha);
            }
        `;

        const cloudUniforms = {
            uTime: { value: 0 },
            uMouse: { value: new THREE.Vector2(0, 0) }
        };

        const cloudMaterial = new THREE.ShaderMaterial({
            vertexShader: cloudVertexShader,
            fragmentShader: cloudFragmentShader,
            uniforms: cloudUniforms,
            transparent: true,
            depthWrite: false,
            side: THREE.DoubleSide
        });

        // สร้างโดมก้อนเมฆ (Cloud Dome Sky)
        const cloudGeometry = new THREE.SphereGeometry(30, 64, 64);
        const cloudMesh = new THREE.Mesh(cloudGeometry, cloudMaterial);
        scene.add(cloudMesh);

        // --- 5. LOAD 3D GLB MODEL WITH HIGH PBR MATERIALS ---
        // ใช้ Public Model จาก Khronos Group CDN (Damaged Helmet PBR Model)
        const gltfLoader = new GLTFLoader();
        const rgbeLoader = new RGBELoader();

        // โหลด Environment Map (HDR) เพื่อความสมจริงของ PBR Reflection
        rgbeLoader.load('https://raw.githubusercontent.com/mrdoob/three.js/dev/examples/textures/equirectangular/venice_sunset_1k.hdr', (texture) => {
            texture.mapping = THREE.EquirectangularReflectionMapping;
            scene.environment = texture;

            // โหลดโมเดล GLB PBR
            gltfLoader.load(
                'https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/main/2.0/DamagedHelmet/glTF-Binary/DamagedHelmet.glb',
                (gltf) => {
                    const model = gltf.scene;
                    model.position.set(0, 0, 0);
                    model.scale.set(1.8, 1.8, 1.8);
                    
                    model.traverse((child) => {
                        if (child.isMesh) {
                            child.castShadow = true;
                            child.receiveShadow = true;
                        }
                    });

                    scene.add(model);

                    // ซ่อน Loading Screen เมื่อโหลดเสร็จ
                    const loaderEl = document.getElementById('loader');
                    loaderEl.style.opacity = '0';
                    setTimeout(() => loaderEl.style.display = 'none', 800);
                },
                (xhr) => {
                    // Progress (optional)
                },
                (error) => {
                    console.error('An error occurred loading the GLB model:', error);
                }
            );
        });

        // --- 6. MOUSE INTERACTION TRACKING ---
        const mouse = new THREE.Vector2();
        const targetMouse = new THREE.Vector2();

        window.addEventListener('mousemove', (event) => {
            //แปลงพิกัด Mouse ให้เป็น Normalised Device Coordinates (-1 ถึง +1)
            targetMouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            targetMouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
        });

        // --- 7. RESIZE HANDLING ---
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        // --- 8. ANIMATION LOOP ---
        const clock = new THREE.Clock();

        function animate() {
            requestAnimationFrame(animate);

            const elapsedTime = clock.getElapsedTime();

            // Lerp พิกัด Mouse เพื่อให้ Shader เคลื่อนไหวอย่างนุ่มนวล
            mouse.x += (targetMouse.x - mouse.x) * 0.05;
            mouse.y += (targetMouse.y - mouse.y) * 0.05;

            // อัปเดต Uniforms ให้แก่ Shader
            cloudUniforms.uTime.value = elapsedTime;
            cloudUniforms.uMouse.value.copy(mouse);

            // อัปเดต OrbitControls
            controls.update();

            // Render Scene
            renderer.render(scene, camera);
        }

        animate();
    </script>
</body>
</html>
