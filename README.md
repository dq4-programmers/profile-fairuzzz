<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>🦖 Dino HD · Grafik Warna Detail</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            user-select: none;
        }
        body {
            background: linear-gradient(145deg, #0a1f2e 0%, #1b3b4f 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            font-family: 'Segoe UI', 'Poppins', system-ui, sans-serif;
        }
        .game-wrapper {
            background: #0f2a3a;
            padding: 20px 25px 30px 25px;
            border-radius: 48px 48px 32px 32px;
            box-shadow: 0 25px 40px rgba(0,0,0,0.7), inset 0 0 0 2px #3c6e8c, inset 0 0 20px #1f4b63;
            border-bottom: 8px solid #1d4b5e;
        }
        canvas {
            display: block;
            width: 1000px;
            height: 400px;
            border-radius: 24px;
            box-shadow: 0 0 0 4px #6e9eb6, 0 20px 30px rgba(0,0,0,0.6);
            background: radial-gradient(circle at 20% 20%, #9ed9ff, #5aa3cc);
            cursor: pointer;
            transition: filter 0.1s;
        }
        canvas:active {
            filter: brightness(0.96);
        }
        .info-panel {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin: 16px 8px 0 8px;
            color: #e6f7ff;
            text-shadow: 0 3px 5px #0a1c26;
            font-weight: 600;
            letter-spacing: 1px;
        }
        .score-board {
            background: #1b3e4e;
            padding: 10px 32px;
            border-radius: 60px;
            box-shadow: inset 0 2px 8px #0a1c26, 0 8px 0 #0a1c26;
            font-size: 28px;
            color: #fff7b0;
            text-shadow: 0 3px 0 #7f5f1a, 0 0 12px #ffd966;
            display: flex;
            align-items: center;
            gap: 12px;
        }
        .score-board span {
            background: #2f5e74;
            padding: 2px 18px;
            border-radius: 40px;
            color: #ffec9e;
        }
        .hint {
            background: #1e4c5e;
            padding: 10px 30px;
            border-radius: 40px;
            font-size: 18px;
            box-shadow: inset 0 2px 6px #0b222e, 0 6px 0 #0a1c26;
            display: flex;
            align-items: center;
            gap: 8px;
            color: #c3e6ff;
        }
        .hint kbd {
            background: #2d6d89;
            padding: 4px 12px;
            border-radius: 30px;
            font-weight: 700;
            color: #ffffff;
            box-shadow: 0 2px 0 #0b2b38;
            margin: 0 3px;
        }
    </style>
</head>
<body>
<div class="game-wrapper">
    <canvas id="gameCanvas" width="1000" height="400"></canvas>
    <div class="info-panel">
        <div class="score-board">
            🏆 SKOR <span id="scoreDisplay">0</span>
        </div>
        <div class="hint">
            ␣ <kbd>SPASI</kbd> / <kbd>↑</kbd> untuk lompat
        </div>
    </div>
</div>

<script>
    (function(){
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreSpan = document.getElementById('scoreDisplay');

        // ========== KONFIGURASI & VARIABEL ==========
        const GROUND_Y = 340;          // permukaan tanah
        const DINO_WIDTH = 58;
        const DINO_HEIGHT = 68;
        const DINO_X = 130;            // posisi kiri dino

        // Fisika
        let gravity = 0.7;
        let jumpPower = -13.5;
        let velocityY = 0;
        let dinoY = GROUND_Y - DINO_HEIGHT;  // posisi Y (kaki di ground)
        let isJumping = false;

        // Rintangan (kaktus)
        let obstacles = [];
        let obstacleSpawnTimer = 0;
        const MIN_OBS_SPAWN = 65;      // frame antar spawn (semakin kecil semakin sering)
        const OBS_WIDTH = 28;
        const OBS_HEIGHT = 48;

        // Skor & game state
        let score = 0;
        let gameOver = false;
        let frameCount = 0;

        // Kecepatan dunia (pixel per frame)
        let worldSpeed = 7.8;
        const MAX_SPEED = 16.5;

        // Efek tanah bergulir (untuk ilusi)
        let groundOffset = 0;

        // Partikel debu (untuk detail)
        let dustParticles = [];

        // ========== GAMBAR DETAIL ==========
        // Semua digambar secara manual dengan Canvas API untuk kontrol penuh

        // ----- AWAN HD (bergerak lambat) -----
        let clouds = [
            { x: 200, y: 60, size: 1.2, speed: 0.2 },
            { x: 700, y: 100, size: 1.0, speed: 0.15 },
            { x: 1100, y: 40, size: 1.5, speed: 0.25 },
            { x: 500, y: 140, size: 0.9, speed: 0.1 },
            { x: 900, y: 80, size: 1.3, speed: 0.18 }
        ];

        // ----- BURUNG / PTERODAKTIL (latar belakang) -----
        let birds = [
            { x: 300, y: 150, speed: 0.5, wingPhase: 0 },
            { x: 800, y: 110, speed: 0.7, wingPhase: 2.0 },
            { x: 1200, y: 180, speed: 0.4, wingPhase: 4.2 }
        ];

        // ========== FUNGSI GAMBAR DETAIL ==========

        // ----- DINO HD (dengan bayangan, gradien, mata, sisik) -----
        function drawDino(x, y, width, height) {
            // Bayangan dinosaurus (gelap transparan)
            ctx.save();
            ctx.shadowColor = 'rgba(0,0,0,0.5)';
            ctx.shadowBlur = 22;
            ctx.shadowOffsetY = 6;
            ctx.shadowOffsetX = 4;

            // ---- BADAN UTAMA (gradien vertikal) ----
            const bodyGrad = ctx.createLinearGradient(x, y, x + width*0.8, y + height);
            bodyGrad.addColorStop(0, '#7ec850');
            bodyGrad.addColorStop(0.45, '#4e9e2f');
            bodyGrad.addColorStop(0.8, '#2c6e1a');
            bodyGrad.addColorStop(1, '#1b4a0e');
            ctx.fillStyle = bodyGrad;
            ctx.beginPath();
            // Badan dinosaurus (bentuk agak lonjong, ekor di kiri)
            ctx.ellipse(x + width*0.62, y + height*0.55, width*0.45, height*0.4, 0, 0, Math.PI*2);
            ctx.fill();

            // ---- KEPALA ----
            const headGrad = ctx.createRadialGradient(
                x + width*0.2, y + height*0.2, 5,
                x + width*0.2, y + height*0.25, 35
            );
            headGrad.addColorStop(0, '#a3e06b');
            headGrad.addColorStop(0.6, '#5fa832');
            headGrad.addColorStop(1, '#2d6b14');
            ctx.fillStyle = headGrad;
            ctx.beginPath();
            ctx.ellipse(x + width*0.22, y + height*0.25, width*0.28, height*0.3, 0, 0, Math.PI*2);
            ctx.fill();

            // ---- LEHER (menyatu) ----
            ctx.fillStyle = '#4e9e2f';
            ctx.beginPath();
            ctx.ellipse(x + width*0.32, y + height*0.33, width*0.18, height*0.18, 0.1, 0, Math.PI*2);
            ctx.fill();

            // ---- MATA BESAR (dengan highlight & pupil) ----
            // Mata putih
            ctx.shadowBlur = 10;
            ctx.shadowColor = 'rgba(255,255,200,0.6)';
            ctx.fillStyle = '#f9f9f9';
            ctx.beginPath();
            ctx.ellipse(x + width*0.15, y + height*0.22, 11, 13, 0, 0, Math.PI*2);
            ctx.fill();

            // Pupil hitam dengan gradien
            const pupilGrad = ctx.createRadialGradient(
                x + width*0.13, y + height*0.2, 2,
                x + width*0.15, y + height*0.22, 9
            );
            pupilGrad.addColorStop(0, '#1f1f1f');
            pupilGrad.addColorStop(0.8, '#0a0a0a');
            ctx.fillStyle = pupilGrad;
            ctx.beginPath();
            ctx.arc(x + width*0.15, y + height*0.22, 7, 0, Math.PI*2);
            ctx.fill();

            // Highlight mata (kilau)
            ctx.shadowBlur = 18;
            ctx.shadowColor = 'white';
            ctx.fillStyle = 'rgba(255,255,255,0.95)';
            ctx.beginPath();
            ctx.arc(x + width*0.11, y + height*0.18, 3.2, 0, Math.PI*2);
            ctx.fill();
            ctx.beginPath();
            ctx.arc(x + width*0.18, y + height*0.25, 2, 0, Math.PI*2);
            ctx.fillStyle = 'rgba(255,255,240,0.8)';
            ctx.fill();

            // ---- MULUT / SENYUM (detail) ----
            ctx.shadowBlur = 8;
            ctx.shadowColor = '#2a2a2a';
            ctx.strokeStyle = '#2f1f0c';
            ctx.lineWidth = 2.8;
            ctx.beginPath();
            ctx.arc(x + width*0.08, y + height*0.35, 8, 0.1, 1.2);
            ctx.stroke();

            // ---- SISIK / TEKSTUR di punggung (detail titik) ----
            ctx.shadowBlur = 4;
            ctx.shadowColor = '#2d4a1a';
            for (let i = 0; i < 12; i++) {
                let sx = x + width*0.45 + i * 5.5;
                let sy = y + height*0.28 + Math.sin(i * 0.9) * 7;
                ctx.fillStyle = i % 2 === 0 ? '#3f8a2a' : '#5fb83a';
                ctx.beginPath();
                ctx.arc(sx, sy, 4 + (i % 3) * 0.6, 0, Math.PI*2);
                ctx.fill();
                // highlight kecil
                ctx.fillStyle = '#b1e68b';
                ctx.beginPath();
                ctx.arc(sx-1.2, sy-1.2, 1.5, 0, Math.PI*2);
                ctx.fill();
            }

            // ---- EKOR (dengan gradien dan ujung runcing) ----
            ctx.shadowBlur = 18;
            ctx.shadowColor = '#1f3f0e';
            const tailGrad = ctx.createLinearGradient(x-10, y+height*0.5, x+30, y+height*0.8);
            tailGrad.addColorStop(0, '#3f8a2a');
            tailGrad.addColorStop(1, '#1d4a0a');
            ctx.fillStyle = tailGrad;
            ctx.beginPath();
            ctx.moveTo(x + width*0.3, y + height*0.45);
            ctx.quadraticCurveTo(x - 20, y + height*0.5, x - 40, y + height*0.65);
            ctx.quadraticCurveTo(x - 12, y + height*0.72, x + width*0.3, y + height*0.62);
            ctx.fill();

            // ---- KAKI (dengan detail) ----
            // kaki belakang
            ctx.fillStyle = '#2e6e18';
            ctx.beginPath();
            ctx.ellipse(x + width*0.55, y + height*0.87, 14, 11, 0, 0, Math.PI*2);
            ctx.fill();
            // kaki depan
            ctx.fillStyle = '#3f8a2a';
            ctx.beginPath();
            ctx.ellipse(x + width*0.25, y + height*0.87, 12, 10, 0, 0, Math.PI*2);
            ctx.fill();

            // kuku / jari
            ctx.shadowBlur = 6;
            ctx.fillStyle = '#d6b56e';
            ctx.beginPath();
            ctx.ellipse(x + width*0.17, y + height*0.92, 4, 3.5, 0, 0, Math.PI*2);
            ctx.fill();
            ctx.beginPath();
            ctx.ellipse(x + width*0.29, y + height*0.92, 4, 3.5, 0, 0, Math.PI*2);
            ctx.fill();
            ctx.beginPath();
            ctx.ellipse(x + width*0.48, y + height*0.92, 4, 3.5, 0, 0, Math.PI*2);
            ctx.fill();

            // ---- DURI PUNGGUNG (warna cerah) ----
            ctx.shadowBlur = 12;
            ctx.shadowColor = '#ffd966';
            ctx.fillStyle = '#ffb347';
            for (let i = 0; i < 6; i++) {
                let spikeX = x + width*0.38 + i * 7.5;
                let spikeY = y + height*0.12 + i * 2.2;
                ctx.beginPath();
                ctx.moveTo(spikeX, spikeY);
                ctx.lineTo(spikeX + 8, spikeY - 13);
                ctx.lineTo(spikeX + 14, spikeY - 4);
                ctx.closePath();
                ctx.fillStyle = i % 2 === 0 ? '#f7b731' : '#f0932b';
                ctx.fill();
            }

            // ---- GRADASI CAHAYA (efek glossy) ----
            ctx.shadowBlur = 25;
            ctx.shadowColor = 'rgba(255,255,200,0.3)';
            ctx.globalAlpha = 0.18;
            ctx.fillStyle = 'white';
            ctx.beginPath();
            ctx.ellipse(x + width*0.4, y + height*0.3, 30, 18, -0.1, 0, Math.PI*2);
            ctx.fill();
            ctx.globalAlpha = 1.0;
            ctx.restore();
        }

        // ----- KAKTUS HD (dengan gradien, duri, bayangan) -----
        function drawCactus(x, y, width, height) {
            ctx.save();
            ctx.shadowColor = 'rgba(0,0,0,0.5)';
            ctx.shadowBlur = 16;
            ctx.shadowOffsetY = 5;
            ctx.shadowOffsetX = 3;

            // Batang utama
            const grad = ctx.createLinearGradient(x, y, x + width, y + height);
            grad.addColorStop(0, '#3b8c3b');
            grad.addColorStop(0.5, '#1e6b1e');
            grad.addColorStop(1, '#0e4a0e');
            ctx.fillStyle = grad;
            ctx.beginPath();
            ctx.roundRect(x, y, width, height, 12);
            ctx.fill();

            // Lengan kiri
            ctx.fillStyle = '#2b7a2b';
            ctx.beginPath();
            ctx.roundRect(x - 12, y + height*0.3, 14, height*0.45, 8);
            ctx.fill();
            // Lengan kanan
            ctx.fillStyle = '#1f6b1f';
            ctx.beginPath();
            ctx.roundRect(x + width - 2, y + height*0.2, 14, height*0.5, 8);
            ctx.fill();

            // Duri-duri (detail)
            ctx.shadowBlur = 6;
            ctx.shadowColor = '#b6d7a8';
            ctx.fillStyle = '#d4e8b0';
            for (let i = 0; i < 8; i++) {
                let dy = y + 8 + i * (height / 9);
                ctx.beginPath();
                ctx.arc(x - 3, dy, 2.8, 0, Math.PI * 2);
                ctx.fill();
                ctx.beginPath();
                ctx.arc(x + width + 3, dy + 6, 2.8, 0, Math.PI * 2);
                ctx.fill();
            }

            // Bunga kaktus (warna cerah)
            ctx.shadowBlur = 18;
            ctx.shadowColor = '#ffb3b3';
            ctx.fillStyle = '#ff6b6b';
            ctx.beginPath();
            ctx.arc(x + width*0.5, y - 6, 10, 0, Math.PI*2);
            ctx.fill();
            ctx.fillStyle = '#ffb347';
            ctx.beginPath();
            ctx.arc(x + width*0.35, y - 10, 6, 0, Math.PI*2);
            ctx.fill();
            ctx.fillStyle = '#ffe066';
            ctx.beginPath();
            ctx.arc(x + width*0.65, y - 12, 7, 0, Math.PI*2);
            ctx.fill();

            // Highlight
            ctx.shadowBlur = 12;
            ctx.globalAlpha = 0.3;
            ctx.fillStyle = 'white';
            ctx.beginPath();
            ctx.ellipse(x + 6, y + 12, 6, 20, 0.1, 0, Math.PI*2);
            ctx.fill();
            ctx.globalAlpha = 1.0;
            ctx.restore();
        }

        // ----- TANAH & RUMPUT DETAIL -----
        function drawGround() {
            // Tanah dengan gradien
            const groundGrad = ctx.createLinearGradient(0, GROUND_Y, 0, canvas.height);
            groundGrad.addColorStop(0, '#8b6b4d');
            groundGrad.addColorStop(0.4, '#6b4f36');
            groundGrad.addColorStop(1, '#3f2e1e');
            ctx.fillStyle = groundGrad;
            ctx.fillRect(0, GROUND_Y, canvas.width, canvas.height - GROUND_Y);

            // Lapisan rumput bergelombang
            ctx.shadowBlur = 14;
            ctx.shadowColor = '#264d1a';
            ctx.fillStyle = '#3f9e2f';
            for (let i = 0; i < canvas.width + 40; i += 20) {
                let offset = (i + groundOffset * 1.8) % 40;
                let x = i - offset;
                ctx.beginPath();
                ctx.moveTo(x, GROUND_Y);
                ctx.lineTo(x + 10, GROUND_Y - 12 - Math.sin(i * 0.5) * 4);
                ctx.lineTo(x + 20, GROUND_Y);
                ctx.fill();
            }

            // Aksen batu / kerikil di tanah
            ctx.shadowBlur = 8;
            ctx.shadowColor = '#2b1e10';
            ctx.fillStyle = '#a58a6f';
            for (let i = 0; i < 18; i++) {
                let stoneX = (i * 73 + groundOffset * 2) % 1100 - 50;
                let stoneY = GROUND_Y + 12 + (i % 5) * 8;
                ctx.beginPath();
                ctx.ellipse(stoneX, stoneY, 5 + (i % 4), 4, 0, 0, Math.PI * 2);
                ctx.fill();
            }
            // Bunga kecil
            ctx.shadowBlur = 16;
            ctx.shadowColor = '#ffb3d9';
            ctx.fillStyle = '#ff9fdb';
            for (let i = 0; i < 8; i++) {
                let fx = (i * 137 + groundOffset * 1.2) % 1100 - 40;
                let fy = GROUND_Y - 5;
                ctx.beginPath();
                ctx.arc(fx, fy, 5, 0, Math.PI * 2);
                ctx.fillStyle = i % 2 ? '#ff80b3' : '#ffb3d9';
                ctx.fill();
                ctx.fillStyle = '#ffe066';
                ctx.beginPath();
                ctx.arc(fx-1, fy-1, 2, 0, Math.PI*2);
                ctx.fill();
            }
        }

        // ----- LATAR BELAKANG (gunung, matahari, awan) -----
        function drawBackground() {
            // Langit gradasi (sudah dari canvas background, tapi kita tambah matahari)
            // Matahari dengan glow
            ctx.save();
            ctx.shadowBlur = 70;
            ctx.shadowColor = '#ffcf6e';
            ctx.beginPath();
            ctx.arc(870, 70, 48, 0, Math.PI * 2);
            ctx.fillStyle = '#fff3b0';
            ctx.fill();
            ctx.shadowBlur = 110;
            ctx.beginPath();
            ctx.arc(870, 70, 40, 0, Math.PI * 2);
            ctx.fillStyle = '#ffe066';
            ctx.fill();
            ctx.restore();

            // Gunung berlapis (detail)
            ctx.save();
            // gunung belakang
            ctx.fillStyle = '#5b7e9b';
            ctx.shadowBlur = 18;
            ctx.shadowColor = '#1f3c4a';
            ctx.beginPath();
            ctx.moveTo(0, GROUND_Y - 20);
            ctx.lineTo(150, 160);
            ctx.lineTo(320, GROUND_Y - 20);
            ctx.fill();
            // gunung tengah
            ctx.fillStyle = '#416d82';
            ctx.beginPath();
            ctx.moveTo(180, GROUND_Y - 20);
            ctx.lineTo(380, 120);
            ctx.lineTo(600, GROUND_Y - 20);
            ctx.fill();
            // gunung depan
            ctx.fillStyle = '#2f5d70';
            ctx.beginPath();
            ctx.moveTo(500, GROUND_Y - 20);
            ctx.lineTo(730, 100);
            ctx.lineTo(950, GROUND_Y - 20);
            ctx.fill();
            ctx.restore();

            // Salju di puncak (detail)
            ctx.fillStyle = '#e9f2f9';
            ctx.shadowBlur = 14;
            ctx.shadowColor = '#b0d4ff';
            ctx.beginPath();
            ctx.moveTo(150, 160);
            ctx.lineTo(180, 130);
            ctx.lineTo(210, 160);
            ctx.fill();
            ctx.beginPath();
            ctx.moveTo(380, 120);
            ctx.lineTo(420, 85);
            ctx.lineTo(460, 120);
            ctx.fill();
            ctx.beginPath();
            ctx.moveTo(730, 100);
            ctx.lineTo(770, 65);
            ctx.lineTo(810, 100);
            ctx.fill();

            // Awan bergerak
            ctx.shadowBlur = 28;
            ctx.shadowColor = 'rgba(255,255,255,0.7)';
            clouds.forEach(c => {
                ctx.fillStyle = 'rgba(255,255,255,0.9)';
                ctx.beginPath();
                ctx.ellipse(c.x, c.y, 55 * c.size, 28 * c.size, 0, 0, Math.PI*2);
                ctx.fill();
                ctx.fillStyle = 'rgba(255,255,255,0.98)';
                ctx.beginPath();
                ctx.ellipse(c.x - 25 * c.size, c.y + 8, 35 * c.size, 20 * c.size, 0, 0, Math.PI*2);
                ctx.fill();
                ctx.beginPath();
                ctx.ellipse(c.x + 30 * c.size, c.y - 5, 40 * c.size, 22 * c.size, 0, 0, Math.PI*2);
                ctx.fill();
            });

            // Burung terbang (latar)
            birds.forEach(b => {
                ctx.save();
                ctx.translate(b.x, b.y);
                let wing = Math.sin(Date.now() * 0.01 + b.wingPhase) * 12;
                ctx.fillStyle = '#2c3e50';
                ctx.shadowBlur = 12;
                ctx.shadowColor = '#1a2633';
                ctx.beginPath();
                ctx.moveTo(-12, 0);
                ctx.lineTo(0, wing);
                ctx.lineTo(12, 0);
                ctx.fillStyle = '#34495e';
                ctx.fill();
                ctx.restore();
            });
        }

        // ----- PARTIKEL DEBU (untuk lompat & lari) -----
        function updateDust() {
            // Spawn debu ketika berlari atau mendarat
            if (!isJumping && !gameOver && frameCount % 4 === 0) {
                dustParticles.push({
                    x: DINO_X + 20 + Math.random() * 20,
                    y: GROUND_Y - 6 + Math.random() * 10,
                    vx: -2 - Math.random() * 3,
                    vy: -1 + Math.random() * 2,
                    life: 1.0,
                    size: 4 + Math.random() * 6,
                    color: `rgba(180, 150, 120, ${0.5 + Math.random()*0.3})`
                });
            }
            // Update partikel
            for (let i = dustParticles.length - 1; i >= 0; i--) {
                let p = dustParticles[i];
                p.x += p.vx;
                p.y += p.vy;
                p.vy += 0.06;
                p.life -= 0.018;
                p.size *= 0.99;
                if (p.life <= 0 || p.x < -20 || p.y > GROUND_Y + 20) {
                    dustParticles.splice(i, 1);
                }
            }
        }

        function drawDust() {
            ctx.save();
            ctx.shadowBlur = 12;
            ctx.shadowColor = '#6b5a4a';
            dustParticles.forEach(p => {
                ctx.globalAlpha = p.life * 0.7;
                ctx.fillStyle = p.color || '#ab8e6b';
                ctx.beginPath();
                ctx.arc(p.x, p.y, p.size, 0, Math.PI*2);
                ctx.fill();
            });
            ctx.restore();
            ctx.globalAlpha = 1.0;
        }

        // ========== GAME LOGIC ==========
        function resetGame() {
            gameOver = false;
            score = 0;
            velocityY = 0;
            dinoY = GROUND_Y - DINO_HEIGHT;
            isJumping = false;
            obstacles = [];
            obstacleSpawnTimer = 0;
            worldSpeed = 7.8;
            dustParticles = [];
            frameCount = 0;
            scoreSpan.textContent = '0';
        }

        function jump() {
            if (gameOver) {
                resetGame();
                return;
            }
            if (!isJumping) {
                velocityY = jumpPower;
                isJumping = true;
                // efek debu
                for (let i = 0; i < 8; i++) {
                    dustParticles.push({
                        x: DINO_X + 30 + Math.random() * 20,
                        y: GROUND_Y - 10 + Math.random() * 10,
                        vx: -3 - Math.random() * 4,
                        vy: -2 + Math.random() * 4,
                        life: 0.9,
                        size: 6 + Math.random() * 8,
                        color: `rgba(160, 130, 100, 0.8)`
                    });
                }
            }
        }

        function spawnObstacle() {
            let x = canvas.width;
            let y = GROUND_Y - OBS_HEIGHT;
            obstacles.push({
                x: x,
                y: y,
                width: OBS_WIDTH,
                height: OBS_HEIGHT,
                active: true
            });
        }

        function updateGame() {
            if (gameOver) return;

            // tingkat kesulitan naik perlahan
            if (worldSpeed < MAX_SPEED && frameCount % 180 === 0) {
                worldSpeed += 0.35;
            }

            // update skor
            if (frameCount % 6 === 0) {
                score += Math.floor(worldSpeed * 0.9);
                scoreSpan.textContent = score;
            }

            // fisika lompat
            if (isJumping) {
                velocityY += gravity;
                dinoY += velocityY;
                if (dinoY >= GROUND_Y - DINO_HEIGHT) {
                    dinoY = GROUND_Y - DINO_HEIGHT;
                    isJumping = false;
                    velocityY = 0;
                    // debu mendarat
                    for (let i = 0; i < 10; i++) {
                        dustParticles.push({
                            x: DINO_X + 20 + Math.random() * 40,
                            y: GROUND_Y - 8,
                            vx: -2 - Math.random() * 4,
                            vy: -1 - Math.random() * 3,
                            life: 0.9,
                            size: 7 + Math.random() * 8,
                            color: `rgba(150, 120, 90, 0.9)`
                        });
                    }
                }
            }

            // spawn obstacle
            obstacleSpawnTimer--;
            if (obstacleSpawnTimer <= 0) {
                let spawnDelay = Math.max(42, Math.floor(MIN_OBS_SPAWN - worldSpeed * 1.5));
                obstacleSpawnTimer = spawnDelay + Math.floor(Math.random() * 18);
                spawnObstacle();
            }

            // gerakkan obstacle dan cek tabrakan
            for (let i = obstacles.length - 1; i >= 0; i--) {
                let obs = obstacles[i];
                obs.x -= worldSpeed;

                // Hapus jika keluar layar
                if (obs.x + obs.width < -50) {
                    obstacles.splice(i, 1);
                    continue;
                }

                // Deteksi tabrakan (dino vs kaktus)
                const dinoRect = {
                    x: DINO_X + 6,           // sedikit toleransi
                    y: dinoY + 6,
                    w: DINO_WIDTH - 16,
                    h: DINO_HEIGHT - 12
                };
                const obsRect = {
                    x: obs.x + 4,
                    y: obs.y + 4,
                    w: obs.width - 8,
                    h: obs.height - 8
                };

                if (dinoRect.x < obsRect.x + obsRect.w &&
                    dinoRect.x + dinoRect.w > obsRect.x &&
                    dinoRect.y < obsRect.y + obsRect.h &&
                    dinoRect.y + dinoRect.h > obsRect.y) {
                    gameOver = true;
                    // Efek ledakan debu
                    for (let j = 0; j < 20; j++) {
                        dustParticles.push({
                            x: DINO_X + 20 + Math.random() * 50,
                            y: dinoY + 20 + Math.random() * 40,
                            vx: -6 + Math.random() * 12,
                            vy: -8 + Math.random() * 14,
                            life: 1.3,
                            size: 10 + Math.random() * 15,
                            color: `rgba(200, 80, 60, 0.9)`
                        });
                    }
                }
            }

            // update tanah bergulir (ilusi)
            groundOffset = (groundOffset + worldSpeed * 0.9) % 40;

            // update awan
            clouds.forEach(c => {
                c.x -= c.speed;
                if (c.x < -150) c.x = canvas.width + 120;
            });

            // update burung
            birds.forEach(b => {
                b.x -= b.speed;
                if (b.x < -80) b.x = canvas.width + 80;
                b.wingPhase += 0.05;
            });

            // update partikel debu
            updateDust();

            frameCount++;
        }

        // ========== RENDER ==========
        function draw() {
            // Bersihkan canvas (langit dasar)
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Latar belakang HD
            drawBackground();

            // Gambar tanah
            drawGround();

            // Gambar obstacle (kaktus)
            obstacles.forEach(obs => {
                drawCactus(obs.x, obs.y, obs.width, obs.height);
            });

            // Gambar Dino dengan posisi Y yang benar
            drawDino(DINO_X, dinoY, DINO_WIDTH, DINO_HEIGHT);

            // Partikel debu (di atas tanah, di bawah dino? sebenarnya di atas dino sedikit)
            drawDust();

            // Overlay game over
            if (gameOver) {
                ctx.save();
                ctx.shadowBlur = 30;
                ctx.shadowColor = '#ff4444';
                ctx.fillStyle = 'rgba(20, 10,

