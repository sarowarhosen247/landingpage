<!DOCTYPE html>
<html lang="en" class="h-full">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Minimalist 3D Interactive Story</title>
    <!-- Tailwind CSS for clean layout and UI styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Three.js for lightweight 3D WebGL rendering -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <!-- Google Fonts for calm, elegant typography -->
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500&family=Newsreader:ital,opsz,wght@0,6..72,300;1,6..72,300&display=swap" rel="stylesheet">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['"Plus Jakarta Sans"', 'sans-serif'],
                        serif: ['"Newsreader"', 'serif'],
                    },
                    colors: {
                        canvas: '#f7f6f2',
                        darkcanvas: '#121212',
                        mutedstone: '#e3e1da',
                        accentwarm: '#c29b61',
                    }
                }
            }
        }
    </script>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #0c0c0e;
            font-family: 'Plus Jakarta Sans', sans-serif;
            user-select: none;
            -webkit-user-select: none;
        }
        #webgl-container {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            z-index: 1;
        }
        .ui-layer {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            z-index: 10;
            pointer-events: none;
        }
        .interactive {
            pointer-events: auto;
        }
        /* Custom subtle fade & slide transitions */
        .story-fade {
            transition: opacity 1.2s cubic-bezier(0.16, 1, 0.3, 1), transform 1.2s cubic-bezier(0.16, 1, 0.3, 1);
        }
        @keyframes pulse-subtle {
            0%, 100% { opacity: 0.4; transform: scale(1); }
            50% { opacity: 0.9; transform: scale(1.05); }
        }
        .animate-pulse-subtle {
            animation: pulse-subtle 4s ease-in-out infinite;
        }
        /* Custom scrollbar for readme modal */
        ::-webkit-scrollbar {
            width: 4px;
        }
        ::-webkit-scrollbar-track {
            background: rgba(255, 255, 255, 0.05);
        }
        ::-webkit-scrollbar-thumb {
            background: rgba(255, 255, 255, 0.2);
            border-radius: 2px;
        }
    </style>
</head>
<body class="h-full text-neutral-200 antialiased selection:bg-neutral-800 selection:text-neutral-200">

    <div id="webgl-container"></div>

    <div class="ui-layer flex flex-col justify-between p-6 md:p-12 box-border">
        
        <!-- Top Header: Title & README trigger -->
        <header class="flex justify-between items-center interactive">
            <div class="flex items-center space-x-3">
                <span class="inline-block w-2 h-2 rounded-full bg-amber-200 animate-pulse-subtle"></span>
                <h1 class="text-xs uppercase tracking-[0.3em] font-medium text-neutral-400">Chronicles of Silence</h1>
            </div>
            <div class="flex items-center space-x-4">
                <button id="audio-toggle" class="text-xs uppercase tracking-widest text-neutral-400 hover:text-white transition-colors py-2 px-3 rounded-full border border-neutral-800 bg-neutral-900/40 backdrop-blur-md">
                    Sound: Off
                </button>
                <button id="readme-btn" class="text-xs uppercase tracking-widest text-neutral-400 hover:text-white transition-colors py-2 px-3 rounded-full border border-neutral-800 bg-neutral-900/40 backdrop-blur-md">
                    About / Specs
                </button>
            </div>
        </header>

        <!-- Center / Bottom Narrative Section -->
        <main class="my-auto max-w-xl mx-auto w-full text-center px-4 interactive">
            <div id="narrative-box" class="story-fade opacity-100 transform translate-y-0">
                <span id="chapter-indicator" class="text-xs uppercase tracking-[0.25em] text-amber-200/70 mb-3 block">Chapter I</span>
                <h2 id="chapter-title" class="font-serif italic text-3xl md:text-5xl font-light text-neutral-100 mb-4 leading-tight">The Awakening Stone</h2>
                <p id="chapter-desc" class="text-neutral-400 font-light text-sm md:text-base leading-relaxed mb-8 max-w-md mx-auto">
                    In the quiet expanse of digital mist, ancient monolithic forms drift in eternal stillness. Click upon the central monolith to awaken the resonance.
                </p>
                <div class="flex justify-center items-center space-x-3">
                    <button id="action-btn" class="group relative px-6 py-3 rounded-full bg-neutral-100 text-neutral-900 text-xs uppercase tracking-widest font-medium hover:bg-amber-100 transition-all duration-300 shadow-lg shadow-black/20">
                        <span class="relative z-10 flex items-center space-x-2">
                            <span>Interact with Monolith</span>
                            <svg class="w-3.5 h-3.5 transform group-hover:translate-x-1 transition-transform" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 5l7 7m0 0l-7 7m7-7H3"/></svg>
                        </span>
                    </button>
                </div>
            </div>
        </main>

        <!-- Footer Navigation & Progress Dots -->
        <footer class="flex flex-col md:flex-row justify-between items-center interactive space-y-4 md:space-y-0">
            <div class="text-xs text-neutral-500 tracking-wider font-light">
                <span id="interaction-hint">Tip: Drag to rotate view • Click elements to progress</span>
            </div>

            <!-- Chapter Navigation Dots -->
            <div class="flex items-center space-x-3 bg-neutral-900/50 border border-neutral-800/80 px-4 py-2 rounded-full backdrop-blur-md">
                <button class="chapter-dot w-2.5 h-2.5 rounded-full bg-neutral-100 transition-all duration-300" data-chapter="0" title="Chapter 1"></button>
                <button class="chapter-dot w-2.5 h-2.5 rounded-full bg-neutral-700 hover:bg-neutral-400 transition-all duration-300" data-chapter="1" title="Chapter 2"></button>
                <button class="chapter-dot w-2.5 h-2.5 rounded-full bg-neutral-700 hover:bg-neutral-400 transition-all duration-300" data-chapter="2" title="Chapter 3"></button>
                <button class="chapter-dot w-2.5 h-2.5 rounded-full bg-neutral-700 hover:bg-neutral-400 transition-all duration-300" data-chapter="3" title="Chapter 4"></button>
            </div>

            <!-- Next / Prev Controls -->
            <div class="flex items-center space-x-2">
                <button id="prev-btn" class="p-2.5 rounded-full border border-neutral-800 bg-neutral-900/40 text-neutral-400 hover:text-white hover:border-neutral-700 transition-all disabled:opacity-30 disabled:cursor-not-allowed" disabled>
                    <svg class="w-4 h-4 transform rotate-180" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/></svg>
                </button>
                <button id="next-btn" class="p-2.5 rounded-full border border-neutral-800 bg-neutral-900/40 text-neutral-400 hover:text-white hover:border-neutral-700 transition-all">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/></svg>
                </button>
            </div>
        </footer>

    </div>

    <div id="readme-modal" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/80 backdrop-blur-md hidden opacity-0 transition-opacity duration-300 interactive">
        <div class="bg-neutral-900 border border-neutral-800 max-w-2xl w-full p-8 rounded-2xl shadow-2xl relative max-h-[85vh] overflow-y-auto">
            <button id="readme-close" class="absolute top-6 right-6 text-neutral-400 hover:text-white">
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/></svg>
            </button>
            <h3 class="font-serif italic text-2xl text-neutral-100 mb-2">Technical Specification & Readme</h3>
            <p class="text-xs uppercase tracking-widest text-amber-200/70 mb-6">Minimalist WebGL Storytelling Architecture</p>
            
            <div class="space-y-4 text-sm text-neutral-300 font-light leading-relaxed">
                <p><strong>Overview:</strong> This single-file WebGL storytelling experience is built with Three.js and Tailwind CSS, adhering to strict minimalist design principles with generous negative space, neutral earth tones, and fluid interactive chapters.</p>
                
                <div>
                    <h4 class="text-xs uppercase tracking-widest text-neutral-400 font-medium mb-1">1. Asset Organization & Structure</h4>
                    <p class="text-xs text-neutral-400">All geometry is procedurally generated via Three.js primitives and custom buffer geometries to guarantee zero external asset loading latency and instant 60fps performance across desktop and mobile browsers.</p>
                </div>

                <div>
                    <h4 class="text-xs uppercase tracking-widest text-neutral-400 font-medium mb-1">2. Key Scripts & Shaders</h4>
                    <p class="text-xs text-neutral-400">The scene graph updates dynamically through a unified render loop. Custom lighting setups feature soft ambient tones, directional highlights, and atmospheric particle dust fields for immersive depth.</p>
                </div>

                <div>
                    <h4 class="text-xs uppercase tracking-widest text-neutral-400 font-medium mb-1">3. Adding New Chapters</h4>
                    <p class="text-xs text-neutral-400">Chapters are defined in the <code>storyChapters</code> JavaScript configuration array. Each entry specifies camera coordinates, lighting intensity, custom 3D group meshes, narrative metadata, and interactive click callbacks.</p>
                </div>
            </div>
            
            <div class="mt-8 pt-6 border-t border-neutral-800 flex justify-end">
                <button id="readme-close-btn" class="px-5 py-2 rounded-full bg-neutral-100 text-neutral-900 text-xs uppercase tracking-widest font-medium hover:bg-neutral-200 transition-colors">
                    Return to Story
                </button>
            </div>
        </div>
    </div>

    <script>
        let scene, camera, renderer, raycaster, mouse;
        let currentChapter = 0;
        let chapterGroups = [];
        let globalTime = 0;
        let isTransitioning = false;
        let particleSystem;
        let audioCtx = null;
        let isAudioEnabled = false;

        const storyData = [
            {
                id: 0,
                indicator: "Chapter I",
                title: "The Awakening Stone",
                desc: "In the quiet expanse of digital mist, ancient monolithic forms drift in eternal stillness. Click upon the central monolith to awaken the resonance.",
                btnText: "Interact with Monolith",
                cameraPos: { x: 0, y: 0, z: 12 },
                targetPos: { x: 0, y: 0, z: 0 },
                bgColor: 0x0c0c0e
            },
            {
                id: 1,
                indicator: "Chapter II",
                title: "The Crystalline Core",
                desc: "As resonance builds, geometric geometry unfurls into intricate crystalline matrices. Touch or click the core to accelerate its harmonic pulse.",
                btnText: "Energize Core",
                cameraPos: { x: 8, y: 4, z: 10 },
                targetPos: { x: 0, y: 0, z: 0 },
                bgColor: 0x0f1115
            },
            {
                id: 2,
                indicator: "Chapter III",
                title: "The Celestial Rings",
                desc: "Ascending beyond terrestrial limits, orbital rings trace celestial pathways through boundless void. Witness the gentle mechanics of time.",
                btnText: "Align Orbits",
                cameraPos: { x: 0, y: 10, z: 8 },
                targetPos: { x: 0, y: 0, z: 0 },
                bgColor: 0x0a0c10
            },
            {
                id: 3,
                indicator: "Chapter IV",
                title: "Infinite Horizon",
                desc: "The journey converges into a serene horizon of endless light and dust. You have arrived at the threshold of silence.",
                btnText: "Replay Journey",
                cameraPos: { x: 0, y: 2, z: 15 },
                targetPos: { x: 0, y: 0, z: 0 },
                bgColor: 0x121014
            }
        ];

        function initThree() {
            const container = document.getElementById('webgl-container');
            
            scene = new THREE.Scene();
            scene.background = new THREE.Color(storyData[0].bgColor);
            scene.fog = new THREE.FogExp2(storyData[0].bgColor, 0.035);

            camera = new THREE.PerspectiveCamera(55, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(storyData[0].cameraPos.x, storyData[0].cameraPos.y, storyData[0].cameraPos.z);

            renderer = new THREE.WebGLRenderer({ antialias: true, alpha: false, powerPreference: "high-performance" });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            container.appendChild(renderer.domElement);

            raycaster = new THREE.Raycaster();
            mouse = new THREE.Vector2();

            // Lighting setup
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
            scene.add(ambientLight);

            const dirLight = new THREE.DirectionalLight(0xfff5e6, 1.2);
            dirLight.position.set(10, 20, 15);
            dirLight.castShadow = true;
            scene.add(dirLight);

            const pointLight = new THREE.PointLight(0xd4af37, 2, 25);
            pointLight.position.set(0, 0, 0);
            scene.add(pointLight);

            buildChapters();
            buildParticles();

            window.addEventListener('resize', onWindowResize);
            window.addEventListener('mousemove', onMouseMove);
            window.addEventListener('click', onClick);
            window.addEventListener('touchstart', onTouchStart, { passive: true });

            animate();
        }

        function buildChapters() {
            // Chapter 0: Monolith Group
            const group0 = new THREE.Group();
            const monoGeo = new THREE.BoxGeometry(2.2, 5, 0.8);
            const monoMat = new THREE.MeshStandardMaterial({
                color: 0xe3e1da,
                roughness: 0.3,
                metalness: 0.2
            });
            const monolith = new THREE.Mesh(monoGeo, monoMat);
            monolith.castShadow = true;
            monolith.receiveShadow = true;
            group0.add(monolith);

            // Surrounding floating shards
            for (let i = 0; i < 5; i++) {
                const shardGeo = new THREE.TetrahedronGeometry(0.5 + Math.random() * 0.4);
                const shardMat = new THREE.MeshStandardMaterial({
                    color: i % 2 === 0 ? 0xc29b61 : 0x8c8980,
                    roughness: 0.4,
                    metalness: 0.3
                });
                const shard = new THREE.Mesh(shardGeo, shardMat);
                const angle = (i / 5) * Math.PI * 2;
                shard.position.set(Math.cos(angle) * 3.5, (Math.random() - 0.5) * 3, Math.sin(angle) * 3.5);
                group0.add(shard);
            }
            scene.add(group0);
            chapterGroups.push(group0);

            // Chapter 1: Crystalline Core Group
            const group1 = new THREE.Group();
            const coreGeo = new THREE.OctahedronGeometry(2.2, 0);
            const coreMat = new THREE.MeshPhysicalMaterial({
                color: 0xf3eee2,
                roughness: 0.1,
                metalness: 0.1,
                transmission: 0.9,
                thickness: 1.2,
                transparent: true,
                opacity: 0.9
            });
            const core = new THREE.Mesh(coreGeo, coreMat);
            group1.add(core);

            // Inner glowing core
            const innerGeo = new THREE.SphereGeometry(1, 32, 32);
            const innerMat = new THREE.MeshBasicMaterial({ color: 0xc29b61 });
            const innerSphere = new THREE.Mesh(innerGeo, innerMat);
            group1.add(innerSphere);

            group1.visible = false;
            scene.add(group1);
            chapterGroups.push(group1);

            // Chapter 2: Celestial Rings Group
            const group2 = new THREE.Group();
            const ringMat = new THREE.MeshStandardMaterial({
                color: 0xd4af37,
                roughness: 0.2,
                metalness: 0.8,
                wireframe: true
            });
            for (let i = 0; i < 3; i++) {
                const ringGeo = new THREE.TorusGeometry(3 + i * 1.5, 0.05, 16, 100);
                const ring = new THREE.Mesh(ringGeo, ringMat);
                ring.rotation.x = Math.PI / (2 + i);
                group2.add(ring);
            }
            const centerOrbGeo = new THREE.SphereGeometry(1.2, 32, 32);
            const centerOrbMat = new THREE.MeshStandardMaterial({ color: 0xefede6, roughness: 0.5 });
            const centerOrb = new THREE.Mesh(centerOrbGeo, centerOrbMat);
            group2.add(centerOrb);

            group2.visible = false;
            scene.add(group2);
            chapterGroups.push(group2);

            // Chapter 3: Infinite Horizon Group
            const group3 = new THREE.Group();
            const archGeo = new THREE.TorusGeometry(5, 0.08, 16, 100, Math.PI);
            const archMat = new THREE.MeshStandardMaterial({ color: 0xe3e1da, roughness: 0.3 });
            const arch = new THREE.Mesh(archGeo, archMat);
            arch.rotation.x = Math.PI;
            group3.add(arch);

            const pillarGeo = new THREE.CylinderGeometry(0.4, 0.6, 6, 32);
            const pillarMat = new THREE.MeshStandardMaterial({ color: 0x5a5853, roughness: 0.6 });
            const leftPillar = new THREE.Mesh(pillarGeo, pillarMat);
            leftPillar.position.set(-5, -3, 0);
            const rightPillar = new THREE.Mesh(pillarGeo, pillarMat);
            rightPillar.position.set(5, -3, 0);
            group3.add(leftPillar);
            group3.add(rightPillar);

            group3.visible = false;
            scene.add(group3);
            chapterGroups.push(group3);
        }

        function buildParticles() {
            const particleCount = 600;
            const geometry = new THREE.BufferGeometry();
            const positions = new Float32Array(particleCount * 3);

            for (let i = 0; i < particleCount * 3; i += 3) {
                positions[i] = (Math.random() - 0.5) * 40;
                positions[i + 1] = (Math.random() - 0.5) * 40;
                positions[i + 2] = (Math.random() - 0.5) * 40;
            }

            geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));

            const material = new THREE.PointsMaterial({
                color: 0xc29b61,
                size: 0.08,
                transparent: true,
                opacity: 0.6
            });

            particleSystem = new THREE.Points(geometry, material);
            scene.add(particleSystem);
        }

        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        function onMouseMove(event) {
            mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
        }

        function onTouchStart(event) {
            if (event.touches.length > 0) {
                mouse.x = (event.touches[0].clientX / window.innerWidth) * 2 - 1;
                mouse.y = -(event.touches[0].clientY / window.innerHeight) * 2 + 1;
                triggerInteraction();
            }
        }

        function onClick() {
            triggerInteraction();
        }

        function triggerInteraction() {
            playSoundEffect();
            // Subtle visual feedback pulse on current chapter group
            const activeGroup = chapterGroups[currentChapter];
            if (activeGroup) {
                let scaleTween = 1.15;
                let duration = 400;
                let startTime = performance.now();

                function pulseAnim(currentTime) {
                    let elapsed = currentTime - startTime;
                    let progress = Math.min(elapsed / duration, 1);
                    let factor = 1 + (scaleTween - 1) * Math.sin(progress * Math.PI);
                    activeGroup.scale.set(factor, factor, factor);
                    if (progress < 1) {
                        requestAnimationFrame(pulseAnim);
                    }
                }
                requestAnimationFrame(pulseAnim);
            }
        }

        function playSoundEffect() {
            if (!isAudioEnabled) return;
            try {
                if (!audioCtx) {
                    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                }
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                
                const frequencies = [329.63, 392.00, 440.00, 523.25, 659.25]; // E minor pentatonic
                osc.frequency.setValueAtTime(frequencies[currentChapter % frequencies.length], audioCtx.currentTime);
                osc.type = 'sine';

                gain.gain.setValueAtTime(0.05, audioCtx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 1.5);

                osc.connect(gain);
                gain.connect(audioCtx.destination);

                osc.start();
                osc.stop(audioCtx.currentTime + 1.5);
            } catch (e) {
                console.log("AudioContext blocked or unavailable.");
            }
        }

        function setChapter(index) {
            if (isTransitioning || index === currentChapter) return;
            isTransitioning = true;

            // Fade out UI narrative box
            const box = document.getElementById('narrative-box');
            box.style.opacity = '0';
            box.style.transform = 'translateY(15px)';

            setTimeout(() => {
                currentChapter = index;
                const data = storyData[currentChapter];

                document.getElementById('chapter-indicator').innerText = data.indicator;
                document.getElementById('chapter-title').innerText = data.title;
                document.getElementById('chapter-desc').innerText = data.desc;
                document.getElementById('action-btn').querySelector('span span').innerText = data.btnText;

                // Toggle 3D groups visibility
                chapterGroups.forEach((g, idx) => {
                    g.visible = (idx === currentChapter);
                });

                // Update navigation dots
                document.querySelectorAll('.chapter-dot').forEach((dot, idx) => {
                    if (idx === currentChapter) {
                        dot.classList.remove('bg-neutral-700', 'hover:bg-neutral-400');
                        dot.classList.add('bg-neutral-100');
                    } else {
                        dot.classList.remove('bg-neutral-100');
                        dot.classList.add('bg-neutral-700', 'hover:bg-neutral-400');
                    }
                });

                // Update prev/next buttons
                document.getElementById('prev-btn').disabled = (currentChapter === 0);
                document.getElementById('next-btn').disabled = (currentChapter === storyData.length - 1);

                // Update background fog & color
                scene.background.setHex(data.bgColor);
                scene.fog.color.setHex(data.bgColor);

                // Fade in UI narrative box
                box.style.opacity = '1';
                box.style.transform = 'translateY(0)';
                isTransitioning = false;
            }, 600);
        }

        function animate() {
            requestAnimationFrame(animate);
            globalTime += 0.01;

            // Subtle rotation of active chapter geometry
            if (chapterGroups[currentChapter]) {
                const group = chapterGroups[currentChapter];
                group.rotation.y = globalTime * 0.3;
                group.rotation.x = Math.sin(globalTime * 0.2) * 0.1;
            }

            // Gentle particle drift
            if (particleSystem) {
                particleSystem.rotation.y = globalTime * 0.05;
            }

            // Smooth camera parallax based on mouse movement
            const targetCamX = storyData[currentChapter].cameraPos.x + mouse.x * 1.5;
            const targetCamY = storyData[currentChapter].cameraPos.y + mouse.y * 1.5;
            camera.position.x += (targetCamX - camera.position.x) * 0.05;
            camera.position.y += (targetCamY - camera.position.y) * 0.05;
            camera.lookAt(storyData[currentChapter].targetPos.x, storyData[currentChapter].targetPos.y, storyData[currentChapter].targetPos.z);

            renderer.render(scene, camera);
        }

        document.addEventListener('DOMContentLoaded', () => {
            initThree();

            document.getElementById('next-btn').addEventListener('click', () => {
                if (currentChapter < storyData.length - 1) {
                    setChapter(currentChapter + 1);
                }
            });

            document.getElementById('prev-btn').addEventListener('click', () => {
                if (currentChapter > 0) {
                    setChapter(currentChapter - 1);
                }
            });

            document.getElementById('action-btn').addEventListener('click', () => {
                triggerInteraction();
                if (currentChapter < storyData.length - 1) {
                    setChapter(currentChapter + 1);
                } else {
                    setChapter(0);
                }
            });

            document.querySelectorAll('.chapter-dot').forEach((dot) => {
                dot.addEventListener('click', (e) => {
                    const idx = parseInt(e.target.getAttribute('data-chapter'));
                    setChapter(idx);
                });
            });

            // Readme Modal Toggle
            const readmeModal = document.getElementById('readme-modal');
            document.getElementById('readme-btn').addEventListener('click', () => {
                readmeModal.classList.remove('hidden');
                setTimeout(() => readmeModal.classList.remove('opacity-0'), 10);
            });
            document.getElementById('readme-close').addEventListener('click', closeModal);
            document.getElementById('readme-close-btn').addEventListener('click', closeModal);

            function closeModal() {
                readmeModal.classList.add('opacity-0');
                setTimeout(() => readmeModal.classList.add('hidden'), 300);
            }

            // Audio Toggle
            const audioBtn = document.getElementById('audio-toggle');
            audioBtn.addEventListener('click', () => {
                isAudioEnabled = !isAudioEnabled;
                audioBtn.innerText = isAudioEnabled ? "Sound: On" : "Sound: Off";
                audioBtn.classList.toggle('border-amber-200/50', isAudioEnabled);
            });
        });
    </script>
</body>
</html>
