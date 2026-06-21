<!DOCTYPE html>  
<html lang="th">  
<head>  
    <meta charset="UTF-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">  
      
    <meta name="apple-mobile-web-app-capable" content="yes">  
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">  
    <meta name="apple-mobile-web-app-title" content="AR Math">  
    <link rel="apple-touch-icon" href="https://cdn-icons-png.flaticon.com/512/3070/3070385.png"> <title>คณิตคิดเลขเร็ว AR - ครูศิรินันท์</title>  
      
    <link rel="preconnect" href="https://fonts.googleapis.com">  
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>  
    <link href="https://fonts.googleapis.com/css2?family=Mali:wght@400;600;700;900&display=swap" rel="stylesheet">  
      
    <script src="https://cdn.tailwindcss.com"></script>  
    <script>  
        tailwind.config = {  
            theme: {  
                extend: {  
                    fontFamily: {  
                        'mali': ['Mali', 'cursive']  
                    },  
                    animation: {  
                        'pop': 'pop 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275)',  
                        'shake': 'shake 0.4s ease-in-out',  
                        'bounce-slow': 'bounce 2s infinite'  
                    },  
                    keyframes: {  
                        pop: {  
                            '0%, 100%': { transform: 'scale(1)' },  
                            '50%': { transform: 'scale(1.15)', backgroundColor: '#4ade80', borderColor: '#ffffff' }  
                        },  
                        shake: {  
                            '0%, 100%': { transform: 'translateX(0)' },  
                            '25%': { transform: 'translateX(-15px) rotate(-2deg)' },  
                            '75%': { transform: 'translateX(15px) rotate(2deg)' }  
                        }  
                    }  
                }  
            }  
        }  
    </script>  
  
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>  
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js" crossorigin="anonymous"></script>  
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js" crossorigin="anonymous"></script>  
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js" crossorigin="anonymous"></script>  
  
    <style>  
        /* Base Setup - No Overflow Guarantee for iPad */  
        body {  
            margin: 0;  
            overflow: hidden;  
            touch-action: none;  
            user-select: none;  
            -webkit-user-select: none;  
            -webkit-touch-callout: none;  
            background-color: #111827;  
        }  
  
        /* Video/Canvas Background mirroring */  
        #output_canvas {  
            width: 100vw;  
            height: 100dvh;  
            object-fit: cover;  
            transform: scaleX(-1);  
        }  
  
        /* Soft Jelly Button Base Shadows */  
        .jelly-shadow-yellow { box-shadow: 0 8px 0 #ca8a04; }  
        .jelly-shadow-blue { box-shadow: 0 8px 0 #2563eb; }  
        .jelly-shadow-pink { box-shadow: 0 8px 0 #db2777; }  
        .jelly-shadow-purple { box-shadow: 0 8px 0 #9333ea; }  
        .jelly-shadow-green { box-shadow: 0 8px 0 #16a34a; }  
          
        .jelly-btn:active {  
            transform: translateY(6px) scale(0.98);  
            box-shadow: 0 2px 0 rgba(0,0,0,0.2) !important;  
        }  
  
        /* iPad Safe Area Padding */  
        .safe-area-pt { padding-top: max(1rem, env(safe-area-inset-top)); }  
        .safe-area-pb { padding-bottom: max(1rem, env(safe-area-inset-bottom)); }  
    </style>  
</head>  
<body class="font-mali text-gray-800">  
  
    <div class="fixed inset-0 w-full h-[100dvh] z-0 pointer-events-none bg-gray-900">  
        <video id="input_video" class="hidden" autoplay playsinline></video>  
        <canvas id="output_canvas"></canvas>  
    </div>  
  
    <div class="fixed inset-0 w-full h-[100dvh] z-20 flex flex-col justify-center items-center pointer-events-none safe-area-pt safe-area-pb">  
          
        <div id="screen-loading" class="absolute inset-0 flex flex-col items-center justify-center bg-blue-50/90 pointer-events-auto backdrop-blur-sm transition-opacity duration-500 z-50">  
            <div class="animate-spin text-7xl md:text-8xl mb-8">⏳</div>  
            <h1 class="text-4xl md:text-5xl font-bold text-blue-600 mb-4 text-center px-4">กำลังเตรียมกล้องและ AI...</h1>  
            <p class="text-2xl text-gray-600 bg-white/80 px-6 py-3 rounded-full shadow-sm">กรุณากดอนุญาตการเข้าถึงกล้อง (Allow Camera)</p>  
        </div>  
  
        <div id="screen-start" class="hidden absolute inset-0 flex flex-col items-center justify-center bg-white/50 pointer-events-auto backdrop-blur-md z-40">  
            <div class="bg-white/95 px-8 md:px-14 py-12 md:py-16 rounded-[50px] shadow-[0_15px_0_#e5e7eb] border-8 border-white text-center max-w-2xl w-[90%]">  
                <h1 class="text-6xl md:text-7xl font-black text-pink-500 mb-2 drop-shadow-md">คณิตคิดเลขเร็ว</h1>  
                <h2 class="text-4xl md:text-5xl font-bold text-blue-500 mb-4">AR Kids Edition ✨</h2>  
                <p class="text-xl md:text-2xl font-semibold text-gray-500 mb-8">ห้องเรียนครูศิรินันท์ จือเหลียง<br>โรงเรียนวัดบางแคกลาง - ไพลประชานุกูล</p>  
                  
                <div class="bg-blue-50 rounded-3xl p-6 mb-10 border-4 border-blue-200 shadow-inner">  
                    <p class="text-2xl font-bold text-gray-700 mb-2">☝️ วิธีเล่นบน iPad:</p>  
                    <p class="text-xl text-gray-600">ตั้ง iPad ไว้ ชูนิ้วชี้ แล้วเล็งค้างไว้ที่ปุ่มเพื่อเลือกคำตอบ</p>  
                </div>  
                <button class="hand-interact jelly-btn bg-green-400 jelly-shadow-green border-4 border-white rounded-[40px] font-black text-white text-4xl md:text-5xl px-12 py-6 w-full transition-transform" onclick="initAndStartGame()">  
                    เริ่มเกมเลย! 🚀  
                </button>  
            </div>  
        </div>  
  
        <div id="screen-game" class="hidden absolute inset-0 flex flex-col items-center justify-between py-6 px-4 md:px-8 h-[100dvh] pointer-events-auto z-30 safe-area-pt safe-area-pb">  
            <div class="flex-none flex justify-between items-center w-full max-w-5xl px-8 py-4 bg-white/95 border-4 border-white shadow-[0_8px_0_#e5e7eb] rounded-[30px]">  
                <span id="question-count" class="text-3xl md:text-4xl font-bold text-blue-600">ข้อที่ 1/5</span>  
                <span id="score-display" class="text-3xl md:text-4xl font-bold text-pink-600">คะแนน: 0</span>  
            </div>  
  
            <div class="flex-1 flex items-center justify-center w-full max-w-4xl my-6">  
                <div class="bg-white/95 rounded-[50px] border-8 border-white shadow-[0_15px_0_#d1d5db] w-full py-14 md:py-20 text-center transform transition-transform">  
                    <h1 id="question-text" class="text-8xl md:text-9xl font-black text-gray-800 tracking-wider drop-shadow-sm"></h1>  
                </div>  
            </div>  
  
            <div id="answers-grid" class="flex-none grid grid-cols-2 gap-6 w-full max-w-5xl h-[35vh] min-h-[260px] pb-4">  
                </div>  
        </div>  
  
        <div id="screen-result" class="hidden absolute inset-0 flex flex-col items-center justify-center bg-white/70 pointer-events-auto backdrop-blur-lg z-40">  
            <div class="bg-white/95 px-10 py-16 md:px-16 md:py-20 rounded-[50px] shadow-[0_15px_0_#e5e7eb] border-8 border-white text-center max-w-2xl w-[90%]">  
                <h1 class="text-6xl md:text-7xl font-black text-purple-500 mb-4">จบเกมแล้ว! 🎉</h1>  
                <p class="text-3xl font-bold text-gray-600 mb-8">คะแนนของหนูคือ</p>  
                <div class="text-9xl font-black text-yellow-400 drop-shadow-xl mb-12" id="final-score">5/5</div>  
                <button class="hand-interact jelly-btn bg-blue-400 jelly-shadow-blue border-4 border-white rounded-[40px] font-black text-white text-4xl px-12 py-6 w-full transition-transform" onclick="startGame()">  
                    เล่นอีกครั้ง 🔄  
                </button>  
            </div>  
        </div>  
  
    </div>  
  
    <div id="ar-cursor" class="hidden fixed z-50 w-24 h-24 pointer-events-none transform -translate-x-1/2 -translate-y-1/2 flex items-center justify-center">  
        <div class="absolute text-6xl drop-shadow-lg">✨</div>  
        <svg class="absolute inset-0 w-full h-full transform -rotate-90">  
            <circle cx="48" cy="48" r="42" stroke="rgba(255,255,255,0.8)" stroke-width="8" fill="none" />  
            <circle id="cursor-ring" cx="48" cy="48" r="42" stroke="#f472b6" stroke-width="8" fill="none" stroke-dasharray="264" stroke-dashoffset="264" stroke-linecap="round" />  
        </svg>  
    </div>  
  
    <script>  
        /* ================= Audio Engine (Tone.js) ================= */  
        let gameSynth;  
          
        async function initAudio() {  
            try {  
                await Tone.start();  
                if (!gameSynth) {  
                    gameSynth = new Tone.PolySynth(Tone.Synth).toDestination();  
                    gameSynth.volume.value = -2; // ปรับเสียงให้ดังขึ้นสำหรับ iPad  
                }  
            } catch (e) {  
                console.error("Audio Engine Error:", e);  
            }  
        }  
  
        function playSound(type) {  
            try {  
                if (!gameSynth) return;  
                let now = Tone.now();  
                switch(type) {  
                    case 'hover': gameSynth.triggerAttackRelease("G4", "32n", now); break;  
                    case 'click': gameSynth.triggerAttackRelease("C5", "16n", now); break;  
                    case 'correct': gameSynth.triggerAttackRelease(["C5", "E5", "G5", "C6"], "8n", now); break;  
                    case 'wrong': gameSynth.triggerAttackRelease(["C3", "D#3"], "8n", now); break;  
                    case 'win':   
                        gameSynth.triggerAttackRelease("C4", "8n", now);  
                        gameSynth.triggerAttackRelease("E4", "8n", now + 0.15);  
                        gameSynth.triggerAttackRelease("G4", "8n", now + 0.3);  
                        gameSynth.triggerAttackRelease("C5", "4n", now + 0.45);  
                        break;  
                }  
            } catch (e) {}  
        }  
  
        /* ================= Game Logic & State ================= */  
        let questionPool = [];  
        let currentQuestions = [];  
        let currentIndex = 0;  
        let score = 0;  
        let isProcessing = false;  
  
        const buttonStyles = [  
            { bg: 'bg-yellow-400', shadow: 'jelly-shadow-yellow' },  
            { bg: 'bg-blue-400', shadow: 'jelly-shadow-blue' },  
            { bg: 'bg-pink-400', shadow: 'jelly-shadow-pink' },  
            { bg: 'bg-purple-400', shadow: 'jelly-shadow-purple' }  
        ];  
  
        function generateQuestions() {  
            let pool = [];  
            while(pool.length < 50) {  
                let op = ['+', '-', 'x', '÷'][Math.floor(Math.random() * 4)];  
                let a, b, ans;  
                if(op === '+') {  
                    a = Math.floor(Math.random() * 20) + 1;  
                    b = Math.floor(Math.random() * 20) + 1;  
                    ans = a + b;  
                } else if(op === '-') {  
                    a = Math.floor(Math.random() * 20) + 10;   
                    b = Math.floor(Math.random() * (a - 1)) + 1;  
                    ans = a - b;  
                } else if(op === 'x') {  
                    a = Math.floor(Math.random() * 9) + 2;  
                    b = Math.floor(Math.random() * 9) + 2;  
                    ans = a * b;  
                } else if(op === '÷') {  
                    b = Math.floor(Math.random() * 9) + 2;  
                    ans = Math.floor(Math.random() * 9) + 2;  
                    a = b * ans;  
                }  
                  
                let qStr = `${a} ${op} ${b}`;  
                if(!pool.some(q => q.text === qStr)) {  
                    let options = [ans];  
                    while(options.length < 4) {  
                        let offset = Math.floor(Math.random() * 11) - 5;  
                        if(offset === 0) offset = 6;  
                        let opt = ans + offset;  
                        if(opt >= 0 && !options.includes(opt)) options.push(opt);  
                    }  
                    options.sort(() => Math.random() - 0.5);  
                    pool.push({ text: qStr, answer: ans, options: options });  
                }  
            }  
            return pool;  
        }  
  
        function showScreen(id) {  
            ['screen-loading', 'screen-start', 'screen-game', 'screen-result'].forEach(scr => {  
                document.getElementById(scr).classList.add('hidden');  
            });  
            document.getElementById(id).classList.remove('hidden');  
        }  
  
        async function initAndStartGame() {  
            await initAudio();  
            startGame();  
        }  
  
        function startGame() {  
            playSound('click');  
            if (questionPool.length === 0) questionPool = generateQuestions();  
              
            questionPool.sort(() => Math.random() - 0.5);  
            currentQuestions = questionPool.slice(0, 5);  
              
            currentIndex = 0;  
            score = 0;  
            updateScoreUI();  
            showScreen('screen-game');  
            loadQuestion();  
        }  
  
        function loadQuestion() {  
            isProcessing = false;  
            let q = currentQuestions[currentIndex];  
            document.getElementById('question-count').innerText = `ข้อที่ ${currentIndex + 1}/5`;  
            document.getElementById('question-text').innerText = `${q.text} = ?`;  
  
            const btnContainer = document.getElementById('answers-grid');  
            btnContainer.innerHTML = '';  
  
            let shuffledStyles = [...buttonStyles].sort(() => Math.random() - 0.5);  
  
            q.options.forEach((opt, idx) => {  
                let style = shuffledStyles[idx];  
                let btn = document.createElement('button');  
                btn.className = `hand-interact jelly-btn w-full h-full ${style.bg} ${style.shadow} border-4 border-white rounded-[40px] font-black text-white text-6xl md:text-7xl flex items-center justify-center transition-all duration-200`;  
                btn.innerText = opt;  
                btn.onclick = () => checkAnswer(btn, opt, q.answer);  
                btnContainer.appendChild(btn);  
            });  
        }  
  
        function checkAnswer(btn, selected, correct) {  
            if (isProcessing) return;  
            isProcessing = true;  
  
            if (selected === correct) {  
                playSound('correct');  
                btn.classList.add('animate-pop');  
                score++;  
                updateScoreUI();  
                spawnSparkles(btn);  
            } else {  
                playSound('wrong');  
                btn.classList.add('animate-shake');  
            }  
  
            setTimeout(() => {  
                currentIndex++;  
                if (currentIndex < 5) loadQuestion();  
                else endGame();  
            }, 1300);  
        }  
  
        function updateScoreUI() {  
            document.getElementById('score-display').innerText = `คะแนน: ${score}`;  
        }  
  
        function endGame() {  
            playSound('win');  
            showScreen('screen-result');  
            document.getElementById('final-score').innerText = `${score} / 5`;  
            spawnConfetti();  
        }  
  
        /* ================= Particle Effects ================= */  
        function spawnSparkles(element) {  
            const rect = element.getBoundingClientRect();  
            for(let i=0; i<10; i++) {  
                let spark = document.createElement('div');  
                spark.innerText = '✨';  
                spark.className = 'fixed text-4xl z-50 pointer-events-none transition-all duration-500 ease-out';  
                spark.style.left = (rect.left + rect.width/2 - 20) + 'px';  
                spark.style.top = (rect.top + rect.height/2 - 20) + 'px';  
                document.body.appendChild(spark);  
  
                let angle = (i / 10) * Math.PI * 2;  
                let dist = 120;  
                setTimeout(() => {  
                    spark.style.transform = `translate(${Math.cos(angle)*dist}px, ${Math.sin(angle)*dist}px) scale(1.5)`;  
                    spark.style.opacity = '0';  
                }, 10);  
                setTimeout(() => spark.remove(), 550);  
            }  
        }  
  
        function spawnConfetti() {  
            const emojis = ['⭐', '🎉', '🎈', '🍬', '✨', '🌈'];  
            for(let i=0; i<80; i++) {  
                let conf = document.createElement('div');  
                conf.innerText = emojis[Math.floor(Math.random()*emojis.length)];  
                conf.className = 'fixed text-5xl z-50 pointer-events-none transition-all duration-[3000ms] ease-out';  
                conf.style.left = Math.random() * 100 + 'vw';  
                conf.style.top = '-10vh';  
                conf.style.transform = `rotate(${Math.random()*360}deg)`;  
                document.body.appendChild(conf);  
  
                setTimeout(() => {  
                    conf.style.top = '110vh';  
                    conf.style.transform = `rotate(${Math.random()*360 + 360}deg)`;  
                }, 50);  
                setTimeout(() => conf.remove(), 3100);  
            }  
        }  
  
        /* ================= AR & MediaPipe System ================= */  
        const videoElement = document.getElementById('input_video');  
        const canvasElement = document.getElementById('output_canvas');  
        const canvasCtx = canvasElement.getContext('2d');  
        let isCameraReady = false;  
  
        let hoverTarget = null;  
        let hoverStartTime = 0;  
        const HOVER_DURATION = 1200;   
  
        function getScreenCoords(normX, normY) {  
            const rect = canvasElement.getBoundingClientRect();  
            const canvasAspect = rect.width / rect.height;  
            const videoAspect = canvasElement.width / canvasElement.height;  
  
            let renderedWidth, renderedHeight, offsetX = 0, offsetY = 0;  
  
            if (canvasAspect > videoAspect) {  
                renderedWidth = rect.width;  
                renderedHeight = renderedWidth / videoAspect;  
                offsetY = (rect.height - renderedHeight) / 2;  
            } else {  
                renderedHeight = rect.height;  
                renderedWidth = renderedHeight * videoAspect;  
                offsetX = (rect.width - renderedWidth) / 2;  
            }  
  
            let screenX = rect.width - (offsetX + normX * renderedWidth);  
            let screenY = offsetY + normY * renderedHeight;  
            return { x: screenX, y: screenY };  
        }  
  
        function updateHoverLogic(screenX, screenY) {  
            const cursor = document.getElementById('ar-cursor');  
            const ring = document.getElementById('cursor-ring');  
              
            cursor.classList.remove('hidden');  
            cursor.style.left = `${screenX}px`;  
            cursor.style.top = `${screenY}px`;  
  
            const elementUnder = document.elementFromPoint(screenX, screenY);  
            const target = elementUnder ? elementUnder.closest('.hand-interact') : null;  
  
            if (target && !isProcessing) {  
                if (target === hoverTarget) {  
                    let elapsed = Date.now() - hoverStartTime;  
                    let progress = Math.min(elapsed / HOVER_DURATION, 1);  
                    ring.style.strokeDashoffset = 264 - (progress * 264);  
  
                    if (elapsed >= HOVER_DURATION) {  
                        playSound('click');  
                        target.click();  
                        hoverTarget = null;  
                        ring.style.strokeDashoffset = 264;  
                    }  
                } else {  
                    hoverTarget = target;  
                    hoverStartTime = Date.now();  
                    ring.style.strokeDashoffset = 264;  
                }  
            } else {  
                hoverTarget = null;  
                ring.style.strokeDashoffset = 264;  
            }  
        }  
  
        const hands = new Hands({locateFile: (file) => {  
            return `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`;  
        }});  
          
        hands.setOptions({  
            maxNumHands: 1,  
            modelComplexity: 1, /* ใช้โมเดลระดับ 1 เพื่อให้ iPad ทำงานได้รวดเร็ว */  
            minDetectionConfidence: 0.7,  
            minTrackingConfidence: 0.7  
        });  
  
        hands.onResults((results) => {  
            if (!isCameraReady) {  
                isCameraReady = true;  
                showScreen('screen-start');  
            }  
  
            canvasCtx.save();  
            canvasCtx.clearRect(0, 0, canvasElement.width, canvasElement.height);  
            canvasCtx.drawImage(results.image, 0, 0, canvasElement.width, canvasElement.height);  
              
            if (results.multiHandLandmarks && results.multiHandLandmarks.length > 0) {  
                const landmarks = results.multiHandLandmarks[0];  
                  
                drawConnectors(canvasCtx, landmarks, HAND_CONNECTIONS, {color: 'rgba(255,255,255,0.8)', lineWidth: 7});  
                drawLandmarks(canvasCtx, landmarks, {color: '#f472b6', lineWidth: 3, radius: 7});  
  
                const indexTip = landmarks[8];  
                const screenCoords = getScreenCoords(indexTip.x, indexTip.y);  
                updateHoverLogic(screenCoords.x, screenCoords.y);  
            } else {  
                document.getElementById('ar-cursor').classList.add('hidden');  
                hoverTarget = null;  
                document.getElementById('cursor-ring').style.strokeDashoffset = 264;  
            }  
            canvasCtx.restore();  
        });  
  
        const camera = new Camera(videoElement, {  
            onFrame: async () => {  
                if (canvasElement.width !== videoElement.videoWidth) {  
                    canvasElement.width = videoElement.videoWidth;  
                    canvasElement.height = videoElement.videoHeight;  
                }  
                await hands.send({image: videoElement});  
            },  
            width: 640,  
            height: 480  
        });  
          
        camera.start();  
    </script>  
</body>  
</html>  
