<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>신나는 사과 받기 게임!</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Jua&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Jua', sans-serif;
            user-select: none;
            -webkit-user-select: none;
        }
        /* 캔버스 터치 동작 최적화 (모바일 스크롤 방지) */
        canvas {
            touch-action: none;
        }
    </style>
</head>
<body class="bg-gradient-to-b from-blue-100 to-green-100 min-h-screen flex flex-col items-center justify-center p-4">

    <div class="relative bg-white p-4 rounded-3xl shadow-2xl border-4 border-green-400 w-full max-w-lg overflow-hidden">
        
        <div class="flex justify-between items-center mb-3">
            <h1 class="text-3xl text-red-500 tracking-wider">🍎 사과 받기!</h1>
            <div class="flex space-x-2">
                <button id="soundToggleBtn" class="bg-yellow-400 hover:bg-yellow-500 text-white font-bold py-1 px-3 rounded-full text-sm transition-transform active:scale-95">
                    🔊 소리 켜짐
                </button>
            </div>
        </div>

        <div class="flex justify-between items-center bg-green-50 p-3 rounded-2xl border border-green-200 mb-4 text-lg">
            <div class="flex items-center space-x-2">
                <span class="text-gray-600">점수:</span>
                <span id="scoreText" class="text-2xl text-green-600 font-bold">0</span>
            </div>
            <div class="flex items-center space-x-2">
                <span class="text-gray-600">최고:</span>
                <span id="highScoreText" class="text-2xl text-blue-600 font-bold">0</span>
            </div>
            <div id="livesContainer" class="flex space-x-1 text-2xl">
                ❤️❤️❤️
            </div>
        </div>

        <div class="relative w-full aspect-[3/4] bg-sky-200 rounded-2xl overflow-hidden border-2 border-sky-300">
            <canvas id="gameCanvas" class="w-full h-full block bg-gradient-to-b from-sky-200 via-sky-100 to-amber-50"></canvas>

            <div id="startScreen" class="absolute inset-0 bg-black/60 flex flex-col items-center justify-center text-white p-6 text-center transition-opacity duration-300">
                <span class="text-6xl mb-4 animate-bounce">🍎</span>
                <h2 class="text-4xl mb-2 text-yellow-300">사과 수확 대작전!</h2>
                <p class="text-sm text-gray-300 mb-6">하늘에서 떨어지는 사과를 바구니로 받아보세요.<br>폭탄과 벌레 먹은 사과는 피해야 합니다!</p>
                
                <div class="mb-6 w-full max-w-xs">
                    <span class="block text-sm text-gray-300 mb-2">난이도 선택</span>
                    <div class="grid grid-cols-3 gap-2">
                        <button onclick="setDifficulty('easy')" id="diff-easy" class="diff-btn py-2 px-3 rounded-xl bg-gray-700 hover:bg-gray-600 text-sm font-semibold border-2 border-transparent">쉬움</button>
                        <button onclick="setDifficulty('medium')" id="diff-medium" class="diff-btn py-2 px-3 rounded-xl bg-green-500 text-white text-sm font-semibold border-2 border-green-300">보통</button>
                        <button onclick="setDifficulty('hard')" id="diff-hard" class="diff-btn py-2 px-3 rounded-xl bg-gray-700 hover:bg-gray-600 text-sm font-semibold border-2 border-transparent">어려움</button>
                    </div>
                </div>

                <button id="startBtn" class="bg-red-500 hover:bg-red-600 text-white text-2xl font-bold py-3 px-8 rounded-2xl shadow-lg transition-transform hover:scale-105 active:scale-95">
                    게임 시작!
                </button>
            </div>

            <div id="gameOverScreen" class="absolute inset-0 bg-black/85 flex flex-col items-center justify-center text-white p-6 text-center hidden">
                <span class="text-6xl mb-4" id="gameOverEmoji">😭</span>
                <h2 class="text-4xl mb-1 text-red-500">게임 오버!</h2>
                <p id="finalScoreMsg" class="text-2xl text-yellow-300 mb-6">최종 점수: 0점</p>
                
                <div class="flex space-x-4">
                    <button id="restartBtn" class="bg-green-500 hover:bg-green-600 text-white text-xl font-bold py-3 px-6 rounded-2xl shadow-lg transition-transform hover:scale-105 active:scale-95">
                        다시 도전!
                    </button>
                    <button id="homeBtn" class="bg-gray-600 hover:bg-gray-500 text-white text-xl font-bold py-3 px-6 rounded-2xl shadow-lg transition-transform hover:scale-105 active:scale-95">
                        메뉴로
                    </button>
                </div>
            </div>
        </div>

        <div class="mt-4 text-center">
            <p class="text-gray-500 text-xs md:text-sm">
                ⌨️ 키보드 <span class="bg-gray-200 px-1.5 py-0.5 rounded border">◀</span> <span class="bg-gray-200 px-1.5 py-0.5 rounded border">▶</span> 방향키 또는 <br class="sm:hidden"> 📱화면 터치/드래그로 바구니를 움직이세요!
            </p>
        </div>
    </div>

    <script>
        // --- 사운드 효과 (Web Audio API) ---
        class SoundEffects {
            constructor() {
                this.ctx = null;
                this.enabled = true;
            }

            init() {
                if (!this.ctx) {
                    this.ctx = new (window.AudioContext || window.webkitAudioContext)();
                }
                if (this.ctx && this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            }

            playCatch() {
                if (!this.enabled) return;
                this.init();
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.connect(gain);
                gain.connect(this.ctx.destination);

                osc.type = 'sine';
                osc.frequency.setValueAtTime(300, this.ctx.currentTime);
                osc.frequency.exponentialRampToValueAtTime(600, this.ctx.currentTime + 0.15);

                gain.gain.setValueAtTime(0.15, this.ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.15);

                osc.start();
                osc.stop(this.ctx.currentTime + 0.15);
            }

            playGoldCatch() {
                if (!this.enabled) return;
                this.init();
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.connect(gain);
                gain.connect(this.ctx.destination);

                osc.type = 'triangle';
                osc.frequency.setValueAtTime(400, this.ctx.currentTime);
                osc.frequency.setValueAtTime(600, this.ctx.currentTime + 0.08);
                osc.frequency.exponentialRampToValueAtTime(900, this.ctx.currentTime + 0.25);

                gain.gain.setValueAtTime(0.2, this.ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.25);

                osc.start();
                osc.stop(this.ctx.currentTime + 0.25);
            }

            playHurt() {
                if (!this.enabled) return;
                this.init();
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.connect(gain);
                gain.connect(this.ctx.destination);

                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(150, this.ctx.currentTime);
                osc.frequency.linearRampToValueAtTime(50, this.ctx.currentTime + 0.2);

                gain.gain
