<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Super QR Bros - Mini-Jeu</title>
    <style>
        /* Importation d'une police pixel rétro */
        @import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');

        * {
            box-sizing: border-box;
            user-select: none;
            -webkit-user-select: none;
            touch-action: none;
        }

        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            background-color: #111827;
            font-family: 'Press Start 2P', monospace, sans-serif;
            overflow: hidden;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
        }

        /* Conteneur principal du jeu */
        #game-wrapper {
            position: relative;
            width: 100%;
            height: 100%;
            max-width: 900px;
            display: flex;
            flex-direction: column;
            background: #000;
            box-shadow: 0 0 30px rgba(0, 0, 0, 0.8);
        }

        /* Barre d'interface (HUD) en haut */
        #hud {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 12px 16px;
            background: #1f2937;
            border-bottom: 4px solid #374151;
            font-size: 11px;
            z-index: 10;
            flex-shrink: 0;
        }

        .hud-item {
            display: flex;
            align-items: center;
            gap: 6px;
        }

        #sound-btn {
            background: #374151;
            border: 2px solid #9ca3af;
            color: white;
            font-family: inherit;
            font-size: 10px;
            padding: 6px 10px;
            border-radius: 6px;
            cursor: pointer;
        }

        #sound-btn:active {
            transform: scale(0.95);
        }

        /* Zone du Canvas */
        #canvas-container {
            position: relative;
            flex-grow: 1;
            width: 100%;
            overflow: hidden;
            background: #5c94fc;
        }

        canvas {
            display: block;
            width: 100%;
            height: 100%;
            image-rendering: pixelated;
        }

        /* Écrans superposés (Accueil, Victoire, Game Over) */
        .overlay {
            position: absolute;
            inset: 0;
            background: rgba(17, 24, 39, 0.88);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            text-align: center;
            padding: 20px;
            z-index: 20;
            backdrop-filter: blur(3px);
        }

        .overlay.hidden {
            display: none;
        }

        .overlay h1 {
            font-size: clamp(18px, 4vw, 32px);
            color: #fbbf24;
            text-shadow: 3px 3px 0px #b45309;
            margin-bottom: 16px;
            line-height: 1.4;
        }
https://MaxWeb42.github.io/super-qr-bros/index%20mario.html
        .overlay p {
            font-size: clamp(10px, 2.2vw, 14px);
            color: #e5e7eb;
            line-height: 1.8;
            max-width: 500px;
            margin-bottom: 24px;
        }

        .retro-btn {
            background: linear-gradient(180deg, #ef4444 0%, #dc2626 100%);
            border: 4px solid #ffffff;
            color: white;
            font-family: 'Press Start 2P', monospace;
            font-size: clamp(12px, 2.5vw, 16px);
            padding: 16px 28px;
            border-radius: 12px;
            cursor: pointer;
            box-shadow: 0 6px 0 #7f1d1d, 0 10px 20px rgba(0,0,0,0.5);
            transition: transform 0.1s;
        }

        .retro-btn:active {
            transform: translateY(4px);
            box-shadow: 0 2px 0 #7f1d1d;
        }

        /* Contrôles tactiles en bas pour mobile */
        #touch-controls {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 14px 20px;
            background: #1f2937;
            border-top: 4px solid #374151;
            flex-shrink: 0;
            z-index: 15;
        }

        .dpad-group {
            display: flex;
            gap: 16px;
        }

        .control-btn {
            width: 68px;
            height: 68px;
            border-radius: 16px;
            background: linear-gradient(180deg, #4b5563 0%, #374151 100%);
            border: 3px solid #9ca3af;
            color: white;
            font-family: 'Press Start 2P', monospace;
            font-size: 20px;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: 0 5px 0 #111827;
            cursor: pointer;
        }

        .control-btn.pressed, .control-btn:active {
            transform: translateY(4px);
            box-shadow: 0 1px 0 #111827;
            background: #6b7280;
            border-color: #fbbf24;
        }

        .jump-btn {
            width: 84px;
            height: 68px;
            background: linear-gradient(180deg, #10b981 0%, #059669 100%);
            border-color: #6ee7b7;
            box-shadow: 0 5px 0 #064e3b;
            font-size: 13px;
        }

        .jump-btn.pressed, .jump-btn:active {
            background: #34d399;
            box-shadow: 0 1px 0 #064e3b;
        }

        @media (min-width: 768px) {
            .hud-item { font-size: 13px; }
        }
    </style>
</head>
<body>

<div id="game-wrapper">
    <!-- Barre de score, vies et niveau -->
    <div id="hud">
        <div class="hud-item">❤️ <span id="lives-display">3</span></div>
        <div class="hud-item">🪙 <span id="score-display">0</span></div>
        <div class="hud-item" id="level-display">NIV 1: PRAIRIE</div>
        <button id="sound-btn" aria-label="Activer ou couper le son">🔊 SON</button>
    </div>

    <!-- Zone de jeu Canvas -->
    <div id="canvas-container">
        <canvas id="gameCanvas"></canvas>

        <!-- Écran d'accueil -->
        <div id="start-screen" class="overlay">
            <h1>SUPER QR BROS</h1>
            <p>
                Scanné avec succès !<br><br>
                Saute sur les monstres, ramasse les pièces d'or et atteins le drapeau à la fin des 3 niveaux !
            </p>
            <button id="start-btn" class="retro-btn">JOUER</button>
        </div>

        <!-- Écran de transition de niveau -->
        <div id="level-screen" class="overlay hidden">
            <h1 id="level-title">NIVEAU TERMINÉ !</h1>
            <p id="level-subtitle">Prépare-toi pour la suite...</p>
            <button id="next-level-btn" class="retro-btn">CONTINUER</button>
        </div>

        <!-- Écran de fin (Victoire ou Game Over) -->
        <div id="end-screen" class="overlay hidden">
            <h1 id="end-title">GAME OVER</h1>
            <p id="end-stats">Score final : 0</p>
            <button id="restart-btn" class="retro-btn">REJOUER</button>
        </div>
    </div>

    <!-- Contrôles tactiles Mobile -->
    <div id="touch-controls">
        <div class="dpad-group">
            <button class="control-btn" id="btn-left" aria-label="Gauche">◀</button>
            <button class="control-btn" id="btn-right" aria-label="Droite">▶</button>
        </div>
        <button class="control-btn jump-btn" id="btn-jump" aria-label="Sauter">SAUT</button>
    </div>
</div>

<script>
/**
 * Moteur de synthèse audio 8-bit via Web Audio API (aucun fichier externe requis)
 */
class RetroAudio {
    constructor() {
        this.ctx = null;
        this.muted = false;
    }

    init() {
        if (!this.ctx) {
            const AudioContext = window.AudioContext || window.webkitAudioContext;
            this.ctx = new AudioContext();
        }
        if (this.ctx.state === 'suspended') {
            this.ctx.resume();
        }
    }

    toggleMute() {
        this.muted = !this.muted;
        return this.muted;
    }

    playTone(freqStart, freqEnd, duration, type = 'square', volume = 0.12) {
        if (this.muted || !this.ctx) return;
        try {
            const osc = this.ctx.createOscillator();
            const gain = this.ctx.createGain();
            osc.type = type;
            osc.frequency.setValueAtTime(freqStart, this.ctx.currentTime);
            if (freqEnd !== freqStart) {
                osc.frequency.exponentialRampToValueAtTime(freqEnd, this.ctx.currentTime + duration);
            }
            gain.gain.setValueAtTime(volume, this.ctx.currentTime);
            gain.gain.exponentialRampToValueAtTime(0.001, this.ctx.currentTime + duration);
            osc.connect(gain);
            gain.connect(this.ctx.destination);
            osc.start();
            osc.stop(this.ctx.currentTime + duration);
        } catch (e) {
            // Ignorer les erreurs audio si contexte bloqué
        }
    }

    jump() {
        this.playTone(220, 580, 0.18, 'square', 0.1);
    }

    coin() {
        this.playTone(987, 987, 0.08, 'sine', 0.15);
        setTimeout(() => this.playTone(1318, 1318, 0.22, 'sine', 0.15), 75);
    }

    stomp() {
        this.playTone(180, 60, 0.15, 'sawtooth', 0.15);
    }

    hurt() {
        this.playTone(300, 90, 0.35, 'sawtooth', 0.18);
    }

    winLevel() {
        const notes = [523, 659, 783, 1046];
        notes.forEach((n, i) => {
            setTimeout(() => this.playTone(n, n, 0.15, 'square', 0.12), i * 110);
        });
    }
}

const audio = new RetroAudio();

/**
 * Définition des 3 niveaux progressifs
 * Légende de la grille :
 * '#' : Bloc de sol / brique
 * 'B' : Bloc bonus / plateforme surélevée
 * 'C' : Pièce d'or
 * 'E' : Ennemi (Goomba rétro)
 * 'S' : Pic / Lave (danger mortel)
 * 'F' : Drapeau de fin de niveau
 * 'P' : Position de départ du joueur
 */
const LEVELS = [
    {
        name: "NIV 1: PRAIRIE",
        skyColor: "#5c94fc",
        groundColor: "#c84c0c",
        grassColor: "#00a800",
        blockColor: "#f8b800",
        map: [
            "                                                                                ",
            "                                                                                ",
            "                                                                                ",
            "                                              C C C                             ",
            "                                            #########                           ",
            "                        C C                                         C C         ",
            "                      #######                                     #######       ",
            "            C                                         C   E                     ",
            "          #####           E                         #########               F   ",
            "  P                   #########       C   C                                ###  ",
            "######   #######     ###########     #######   E             E            ##### ",
            "######   #######     ###########     #######  #####   ################### ######"
        ]
    },
    {
        name: "NIV 2: CAVERNES",
        skyColor: "#1e1b4b",
        groundColor: "#475569",
        grassColor: "#38bdf8",
        blockColor: "#818cf8",
        map: [
            "                                                                                    ",
            "                                                                                    ",
            "                                          C C C                                     ",
            "                                        #########                     C C C         ",
            "                     C C                                            #########       ",
            "                   #######      E                     C   E                         ",
            "          C                   #####                 #########                   F   ",
            "        #####                                                                  ###  ",
            "  P               E                     C   C                    E            ##### ",
            "#####          #######   ###         #########   ###          #######        #######",
            "#####   SSS    #######   ###   SSS   #########   ###   SSS    #######   SSS  #######",
            "####################################################################################"
        ]
    },
    {
        name: "NIV 3: CHÂTEAU",
        skyColor: "#2a0808",
        groundColor: "#3f3f46",
        grassColor: "#ef4444",
        blockColor: "#f59e0b",
        map: [
            "                                                                                        ",
            "                                                                  C C C                 ",
            "                                          C   C                 #########               ",
            "                        C C             #########                                       ",
            "                      #######                       E                         C C       ",
            "            C                                     #####                     #######     ",
            "          #####           E                                     E                   F   ",
            "  P                   #########       C                     #########              ###  ",
            "#####              ##############   #####     E          ##############           ##### ",
            "#####   SSSSS      ##############   #####   #####   SSSS ##############   SSSSS  #######",
            "########################################################################################",
            "########################################################################################"
        ]
    }
];

// Références DOM
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const container = document.getElementById('canvas-container');

const livesDisplay = document.getElementById('lives-display');
const scoreDisplay = document.getElementById('score-display');
const levelDisplay = document.getElementById('level-display');
const soundBtn = document.getElementById('sound-btn');

const startScreen = document.getElementById('start-screen');
const levelScreen = document.getElementById('level-screen');
const endScreen = document.getElementById('end-screen');

// Variables d'état du jeu
const TILE_SIZE = 36;
const GRAVITY = 0.58;
const FRICTION = 0.82;

let gameState = 'START'; // START, PLAYING, LEVEL_TRANSITION, GAME_OVER, VICTORY
let currentLevelIdx = 0;
let score = 0;
let lives = 3;
let cameraX = 0;
let animationId = null;
let frameCount = 0;

// Entités du niveau courant
let tiles = [];
let coins = [];
let enemies = [];
let spikes = [];
let particles = [];
let flag = { x: 0, y: 0, width: TILE_SIZE, height: TILE_SIZE * 2 };
let levelWidth = 0;
let levelHeight = 0;

// Joueur
const player = {
    x: 50,
    y: 100,
    width: 26,
    height: 32,
    vx: 0,
    vy: 0,
    speed: 4.5,
    jumpPower: -12.2,
    grounded: false,
    facingRight: true,
    invincibleTimer: 0
};

// État des contrôles (Clavier + Tactile)
const keys = {
    left: false,
    right: false,
    jump: false
};

/**
 * Redimensionnement adaptatif du canvas
 */
function resizeCanvas() {
    canvas.width = container.clientWidth;
    canvas.height = container.clientHeight;
}
window.addEventListener('resize', resizeCanvas);

/**
 * Chargement et construction d'un niveau à partir de sa carte ASCII
 */
function loadLevel(index) {
    const levelData = LEVELS[index];
    levelDisplay.textContent = levelData.name;
    
    tiles = [];
    coins = [];
    enemies = [];
    spikes = [];
    particles = [];
    cameraX = 0;

    const rows = levelData.map.length;
    const cols = levelData.map[0].length;
    levelWidth = cols * TILE_SIZE;
    levelHeight = rows * TILE_SIZE;

    for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
            const char = levelData.map[r][c];
            const x = c * TILE_SIZE;
            const y = r * TILE_SIZE;

            if (char === '#') {
                tiles.push({ x, y, width: TILE_SIZE, height: TILE_SIZE, isTop: (r > 0 && levelData.map[r-1][c] !== '#') });
            } else if (char === 'C') {
                coins.push({ x: x + 8, y: y + 8, width: 20, height: 20, collected: false, bobOffset: Math.random() * Math.PI * 2 });
            } else if (char === 'E') {
                enemies.push({
                    x: x + 2,
                    y: y + 4,
                    width: 30,
                    height: 30,
                    vx: -1.4 - (index * 0.3), // Plus rapide selon le niveau
                    vy: 0,
                    alive: true
                });
            } else if (char === 'S') {
                spikes.push({ x, y: y + 14, width: TILE_SIZE, height: TILE_SIZE - 14 });
            } else if (char === 'F') {
                flag = { x: x + 10, y: y - TILE_SIZE, width: 16, height: TILE_SIZE * 2 };
            } else if (char === 'P') {
                player.x = x;
                player.y = y;
                player.vx = 0;
                player.vy = 0;
                player.grounded = false;
            }
        }
    }
}

/**
 * Création de particules visuelles (pièces, ennemis écrasés)
 */
function spawnParticles(x, y, color, count = 8) {
    for (let i = 0; i < count; i++) {
        particles.push({
            x,
            y,
            vx: (Math.random() - 0.5) * 6,
            vy: (Math.random() - 1) * 6,
            size: Math.random() * 5 + 3,
            color,
            life: 25
        });
    }
}

/**
 * Détection de collision AABB standard
 */
function checkAABB(a, b) {
    return (
        a.x < b.x + b.width &&
        a.x + a.width > b.x &&
        a.y < b.y + b.height &&
        a.y + a.height > b.y
    );
}

/**
 * Mise à jour de la physique et de la logique du jeu
 */
function update() {
    if (gameState !== 'PLAYING') return;
    frameCount++;

    if (player.invincibleTimer > 0) {
        player.invincibleTimer--;
    }

    // Déplacement horizontal du joueur
    if (keys.left) {
        player.vx = -player.speed;
        player.facingRight = false;
    } else if (keys.right) {
        player.vx = player.speed;
        player.facingRight = true;
    } else {
        player.vx *= FRICTION;
        if (Math.abs(player.vx) < 0.1) player.vx = 0;
    }

    // Saut
    if (keys.jump && player.grounded) {
        player.vy = player.jumpPower;
        player.grounded = false;
        audio.jump();
    }

    // Application de la gravité
    player.vy += GRAVITY;
    if (player.vy > 14) player.vy = 14;

    // Mouvement X et collisions horizontales
    player.x += player.vx;
    if (player.x < 0) player.x = 0;
    if (player.x + player.width > levelWidth) player.x = levelWidth - player.width;

    for (const tile of tiles) {
        if (checkAABB(player, tile)) {
            if (player.vx > 0) {
                player.x = tile.x - player.width;
            } else if (player.vx < 0) {
                player.x = tile.x + tile.width;
            }
            player.vx = 0;
        }
    }

    // Mouvement Y et collisions verticales
    player.y += player.vy;
    player.grounded = false;

    for (const tile of tiles) {
        if (checkAABB(player, tile)) {
            if (player.vy > 0) {
                player.y = tile.y - player.height;
                player.vy = 0;
                player.grounded = true;
            } else if (player.vy < 0) {
                player.y = tile.y + tile.height;
                player.vy = 0;
            }
        }
    }

    // Chute dans un trou
    if (player.y > levelHeight + 60) {
        damagePlayer(true);
        return;
    }

    // Collision avec les pics / lave
    for (const spike of spikes) {
        if (checkAABB(player, spike)) {
            damagePlayer(true);
            return;
        }
    }

    // Collecte des pièces
    for (const coin of coins) {
        if (!coin.collected && checkAABB(player, coin)) {
            coin.collected = true;
            score += 100;
            scoreDisplay.textContent = score;
            audio.coin();
            spawnParticles(coin.x + 10, coin.y + 10, '#fbbf24', 6);
        }
    }

    // Mise à jour des ennemis
    for (const enemy of enemies) {
        if (!enemy.alive) continue;

        enemy.vy += GRAVITY;
        enemy.x += enemy.vx;

        // Demi-tour contre les murs
        for (const tile of tiles) {
            if (checkAABB(enemy, tile)) {
                if (enemy.vx > 0) enemy.x = tile.x - enemy.width;
                else enemy.x = tile.x + tile.width;
                enemy.vx *= -1;
            }
        }

        enemy.y += enemy.vy;
        let onFloor = false;
        for (const tile of tiles) {
            if (checkAABB(enemy, tile)) {
                if (enemy.vy > 0) {
                    enemy.y = tile.y - enemy.height;
                    enemy.vy = 0;
                    onFloor = true;
                }
            }
        }

        // Demi-tour au bord des plateformes pour ne pas tomber dans le vide
        if (onFloor) {
            const checkAheadX = enemy.vx > 0 ? enemy.x + enemy.width + 4 : enemy.x - 4;
            const checkAheadY = enemy.y + enemy.height + 6;
            const hasGroundAhead = tiles.some(t =>
                checkAheadX >= t.x && checkAheadX <= t.x + t.width &&
                checkAheadY >= t.y && checkAheadY <= t.y + t.height
            );
            if (!hasGroundAhead) {
                enemy.vx *= -1;
            }
        }

        // Interaction Joueur <-> Ennemi
        if (checkAABB(player, enemy)) {
            // Si le joueur tombe sur la tête de l'ennemi
            if (player.vy > 0 && (player.y + player.height - player.vy) <= enemy.y + 14) {
                enemy.alive = false;
                player.vy = -8.5; // Rebond
                score += 250;
                scoreDisplay.textContent = score;
                audio.stomp();
                spawnParticles(enemy.x + 15, enemy.y + 15, '#a855f7', 10);
            } else if (player.invincibleTimer === 0) {
                damagePlayer(false);
            }
        }
    }

    // Drapeau de fin de niveau
    if (checkAABB(player, flag)) {
        completeLevel();
    }

    // Mise à jour des particules
    for (let i = particles.length - 1; i >= 0; i--) {
        const p = particles[i];
        p.x += p.vx;
        p.y += p.vy;
        p.vy += 0.25;
        p.life--;
        if (p.life <= 0) particles.splice(i, 1);
    }

    // Suivi fluide de la caméra
    const targetCamX = player.x - canvas.width * 0.35;
    cameraX += (targetCamX - cameraX) * 0.12;
    if (cameraX < 0) cameraX = 0;
    if (cameraX > levelWidth - canvas.width) {
        cameraX = Math.max(0, levelWidth - canvas.width);
    }
}

/**
 * Gestion des dégâts subis par le joueur
 */
function damagePlayer(instantRespawn) {
    audio.hurt();
    lives--;
    livesDisplay.textContent = lives;
    spawnParticles(player.x + 13, player.y + 16, '#ef4444', 12);

    if (lives <= 0) {
        gameState = 'GAME_OVER';
        document.getElementById('end-title').textContent = "GAME OVER";
        document.getElementById('end-title').style.color = "#ef4444";
        document.getElementById('end-stats').textContent = `Score final : ${score} pts`;
        endScreen.classList.remove('hidden');
    } else if (instantRespawn) {
        loadLevel(currentLevelIdx);
    } else {
        player.invincibleTimer = 75;
        player.vy = -7;
    }
}

/**
 * Passage au niveau suivant ou victoire finale
 */
function completeLevel() {
    audio.winLevel();
    score += 500;
    scoreDisplay.textContent = score;

    if (currentLevelIdx < LEVELS.length - 1) {
        gameState = 'LEVEL_TRANSITION';
        document.getElementById('level-title').textContent = `${LEVELS[currentLevelIdx].name} TERMINÉ !`;
        document.getElementById('level-subtitle').textContent = `Prochaine étape : ${LEVELS[currentLevelIdx + 1].name}`;
        levelScreen.classList.remove('hidden');
    } else {
        gameState = 'VICTORY';
        document.getElementById('end-title').textContent = "VICTOIRE TOTALE !";
        document.getElementById('end-title').style.color = "#34d399";
        document.getElementById('end-stats').textContent = `Bravo ! Tu as fini les 3 niveaux avec ${score} pts !`;
        endScreen.classList.remove('hidden');
    }
}

/**
 * Rendu graphique Pixel-Art sur le Canvas
 */
function draw() {
    const levelData = LEVELS[currentLevelIdx];

    // Fond du ciel
    ctx.fillStyle = levelData.skyColor;
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Décor d'arrière-plan (nuages ou étoiles selon le niveau)
    ctx.fillStyle = currentLevelIdx === 0 ? "rgba(255,255,255,0.45)" : "rgba(255,255,255,0.2)";
    for (let i = 0; i < 12; i++) {
        const bgX = ((i * 260) - cameraX * 0.3) % (canvas.width + 200);
        const drawX = bgX < -100 ? bgX + canvas.width + 200 : bgX;
        const bgY = 40 + (i % 3) * 55;
        if (currentLevelIdx === 0) {
            ctx.fillRect(drawX, bgY, 64, 20);
            ctx.fillRect(drawX + 12, bgY - 10, 40, 14);
        } else {
            ctx.fillRect(drawX, bgY, 6, 6);
        }
    }

    // Calcul du décalage vertical pour toujours voir le sol en bas de l'écran
    const offsetY = canvas.height - levelHeight;

    ctx.save();
    ctx.translate(-Math.floor(cameraX), Math.floor(offsetY));

    // Dessin des blocs de terrain
    for (const tile of tiles) {
        if (tile.x + tile.width < cameraX || tile.x > cameraX + canvas.width) continue;

        ctx.fillStyle = levelData.groundColor;
        ctx.fillRect(tile.x, tile.y, tile.width, tile.height);

        // Bordure pixel-art
        ctx.strokeStyle = "rgba(0,0,0,0.35)";
        ctx.lineWidth = 2;
        ctx.strokeRect(tile.x + 1, tile.y + 1, tile.width - 2, tile.height - 2);

        // Surface supérieure (herbe / pierre / lave)
        if (tile.isTop) {
            ctx.fillStyle = levelData.grassColor;
            ctx.fillRect(tile.x, tile.y, tile.width, 8);
        }
    }

    // Dessin des pics / dangers
    for (const spike of spikes) {
        ctx.fillStyle = "#ef4444";
        const teeth = 3;
        const toothW = spike.width / teeth;
        for (let t = 0; t < teeth; t++) {
            ctx.beginPath();
            ctx.moveTo(spike.x + t * toothW, spike.y + spike.height);
            ctx.lineTo(spike.x + t * toothW + toothW / 2, spike.y);
            ctx.lineTo(spike.x + (t + 1) * toothW, spike.y + spike.height);
            ctx.fill();
        }
    }

    // Dessin du drapeau de fin
    ctx.fillStyle = "#e5e7eb";
    ctx.fillRect(flag.x + 6, flag.y, 4, flag.height);
    ctx.fillStyle = "#fbbf24";
    ctx.beginPath();
    ctx.arc(flag.x + 8, flag.y, 6, 0, Math.PI * 2);
    ctx.fill();
    // Fanion flottant
    const wave = Math.sin(frameCount * 0.15) * 3;
    ctx.fillStyle = "#10b981";
    ctx.beginPath();
    ctx.moveTo(flag.x + 10, flag.y + 6);
    ctx.lineTo(flag.x + 38, flag.y + 16 + wave);
    ctx.lineTo(flag.x + 10, flag.y + 26);
    ctx.fill();

    // Dessin des pièces d'or animées
    for (const coin of coins) {
        if (coin.collected) continue;
        const bob = Math.sin(frameCount * 0.1 + coin.bobOffset) * 3;
        ctx.fillStyle = "#fbbf24";
        ctx.beginPath();
        ctx.ellipse(coin.x + 10, coin.y + 10 + bob, 8, 10, 0, 0, Math.PI * 2);
        ctx.fill();
        ctx.strokeStyle = "#b45309";
        ctx.lineWidth = 2;
        ctx.stroke();
        // Reflet
        ctx.fillStyle = "#fef3c7";
        ctx.fillRect(coin.x + 7, coin.y + 5 + bob, 3, 8);
    }

    // Dessin des ennemis (style Goomba rétro)
    for (const enemy of enemies) {
        if (!enemy.alive) continue;
        const step = Math.floor(frameCount / 10) % 2;

        // Corps champignon
        ctx.fillStyle = "#9333ea";
        ctx.beginPath();
        ctx.arc(enemy.x + 15, enemy.y + 14, 14, Math.PI, 0);
        ctx.lineTo(enemy.x + 29, enemy.y + 22);
        ctx.lineTo(enemy.x + 1, enemy.y + 22);
        ctx.fill();

        // Pieds animés
        ctx.fillStyle = "#1f2937";
        ctx.fillRect(enemy.x + 3 + (step * 3), enemy.y + 22, 9, 8);
        ctx.fillRect(enemy.x + 18 - (step * 3), enemy.y + 22, 9, 8);

        // Yeux fâchés
        ctx.fillStyle = "#ffffff";
        ctx.fillRect(enemy.x + 7, enemy.y + 11, 5, 6);
        ctx.fillRect(enemy.x + 18, enemy.y + 11, 5, 6);
        ctx.fillStyle = "#000000";
        ctx.fillRect(enemy.x + 9, enemy.y + 13, 2, 4);
        ctx.fillRect(enemy.x + 19, enemy.y + 13, 2, 4);
    }

    // Dessin du joueur (clignote s'il est invincible)
    if (player.invincibleTimer === 0 || Math.floor(frameCount / 4) % 2 === 0) {
        // Casquette rouge
        ctx.fillStyle = "#ef4444";
        ctx.fillRect(player.x + 2, player.y, 22, 7);
        if (player.facingRight) ctx.fillRect(player.x + 14, player.y + 4, 13, 3);
        else ctx.fillRect(player.x - 1, player.y + 4, 13, 3);

        // Visage
        ctx.fillStyle = "#fed7aa";
        ctx.fillRect(player.x + 4, player.y + 7, 18, 10);

        // Œil
        ctx.fillStyle = "#000000";
        const eyeX = player.facingRight ? player.x + 16 : player.x + 7;
        ctx.fillRect(eyeX, player.y + 9, 3, 4);

        // Salopette bleue & t-shirt rouge
        ctx.fillStyle = "#2563eb";
        ctx.fillRect(player.x + 4, player.y + 17, 18, 10);
        ctx.fillStyle = "#ef4444";
        ctx.fillRect(player.x + 2, player.y + 17, 4, 7);
        ctx.fillRect(player.x + 20, player.y + 17, 4, 7);

        // Chaussures animées
        ctx.fillStyle = "#78350f";
        const walkAnim = (Math.abs(player.vx) > 0.5 && player.grounded) ? (Math.floor(frameCount / 5) % 2) * 3 : 0;
        ctx.fillRect(player.x + 3 - walkAnim, player.y + 27, 8, 5);
        ctx.fillRect(player.x + 15 + walkAnim, player.y + 27, 8, 5);
    }

    // Dessin des particules
    for (const p of particles) {
        ctx.fillStyle = p.color;
        ctx.fillRect(p.x, p.y, p.size, p.size);
    }

    ctx.restore();
}

/**
 * Boucle principale du jeu
 */
function gameLoop() {
    update();
    draw();
    animationId = requestAnimationFrame(gameLoop);
}

/**
 * Démarrage et réinitialisation du jeu
 */
function startNewGame() {
    audio.init();
    score = 0;
    lives = 3;
    currentLevelIdx = 0;
    scoreDisplay.textContent = score;
    livesDisplay.textContent = lives;
    loadLevel(currentLevelIdx);
    startScreen.classList.add('hidden');
    levelScreen.classList.add('hidden');
    endScreen.classList.add('hidden');
    gameState = 'PLAYING';
}

// Événements des boutons UI
document.getElementById('start-btn').addEventListener('click', startNewGame);
document.getElementById('restart-btn').addEventListener('click', startNewGame);
document.getElementById('next-level-btn').addEventListener('click', () => {
    currentLevelIdx++;
    loadLevel(currentLevelIdx);
    levelScreen.classList.add('hidden');
    gameState = 'PLAYING';
});

soundBtn.addEventListener('click', () => {
    audio.init();
    const isMuted = audio.toggleMute();
    soundBtn.textContent = isMuted ? "🔇 MUET" : "🔊 SON";
});

// Contrôles Clavier (Flèches, ZQSD, Espace)
window.addEventListener('keydown', (e) => {
    if (e.code === 'ArrowLeft' || e.code === 'KeyA' || e.code === 'KeyQ') keys.left = true;
    if (e.code === 'ArrowRight' || e.code === 'KeyD') keys.right = true;
    if (e.code === 'ArrowUp' || e.code === 'KeyW' || e.code === 'KeyZ' || e.code === 'Space') {
        keys.jump = true;
    }
});

window.addEventListener('keyup', (e) => {
    if (e.code === 'ArrowLeft' || e.code === 'KeyA' || e.code === 'KeyQ') keys.left = false;
    if (e.code === 'ArrowRight' || e.code === 'KeyD') keys.right = false;
    if (e.code === 'ArrowUp' || e.code === 'KeyW' || e.code === 'KeyZ' || e.code === 'Space') {
        keys.jump = false;
    }
});

// Contrôles Tactiles (Multi-touch fluide pour bouger et sauter en même temps)
function bindTouchControl(btnId, keyName) {
    const btn = document.getElementById(btnId);

    const press = (e) => {
        e.preventDefault();
        audio.init();
        keys[keyName] = true;
        btn.classList.add('pressed');
    };

    const release = (e) => {
        e.preventDefault();
        keys[keyName] = false;
        btn.classList.remove('pressed');
    };

    btn.addEventListener('touchstart', press, { passive: false });
    btn.addEventListener('touchend', release, { passive: false });
    btn.addEventListener('touchcancel', release, { passive: false });
    btn.addEventListener('mousedown', press);
    btn.addEventListener('mouseup', release);
    btn.addEventListener('mouseleave', release);
}

bindTouchControl('btn-left', 'left');
bindTouchControl('btn-right', 'right');
bindTouchControl('btn-jump', 'jump');

// Initialisation au chargement de la fenêtre
window.onload = function() {
    resizeCanvas();
    loadLevel(0);
    gameLoop();
};
</script>
</body>
</html>
