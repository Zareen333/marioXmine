<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta
    name="viewport"
    content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no"
  />
  <title>Blockbound Villager</title>

  <style>
    * {
      box-sizing: border-box;
      -webkit-tap-highlight-color: transparent;
    }

    html,
    body {
      margin: 0;
      width: 100%;
      height: 100%;
      overflow: hidden;
      background: #101820;
      color: white;
      font-family: monospace;
    }

    body {
      display: grid;
      place-items: center;
    }

    #gameShell {
      position: relative;
      width: min(100vw, 1200px);
      height: min(100vh, 720px);
      overflow: hidden;
      background: #79c8ff;
      border: 5px solid #172117;
      box-shadow: 0 0 0 5px #52633e, 0 14px 40px #000a;
      touch-action: none;
    }

    canvas {
      display: block;
      width: 100%;
      height: 100%;
      image-rendering: pixelated;
      image-rendering: crisp-edges;
    }

    #hud {
      position: absolute;
      top: 12px;
      left: 12px;
      right: 12px;
      z-index: 3;
      display: flex;
      justify-content: space-between;
      gap: 10px;
      pointer-events: none;
      text-shadow: 3px 3px #1a1a1a;
      font-weight: bold;
      font-size: clamp(12px, 2vw, 19px);
    }

    .hudPanel {
      padding: 8px 12px;
      border: 3px solid #202020;
      background: #0009;
      box-shadow: inset 0 0 0 2px #ffffff22;
    }

    #controls {
      position: absolute;
      z-index: 4;
      left: 16px;
      right: 16px;
      bottom: 14px;
      display: flex;
      justify-content: space-between;
      align-items: end;
      pointer-events: none;
    }

    .controlGroup {
      display: flex;
      gap: 11px;
      pointer-events: auto;
    }

    button {
      width: clamp(58px, 10vw, 80px);
      height: clamp(58px, 10vw, 80px);
      padding: 0;
      border: 4px solid #151515;
      border-radius: 8px;
      background: #5b646bde;
      color: white;
      font-size: clamp(25px, 5vw, 38px);
      font-weight: 900;
      font-family: monospace;
      box-shadow:
        inset 4px 4px #ffffff45,
        inset -5px -5px #0005,
        0 5px #111;
      user-select: none;
      touch-action: none;
      cursor: pointer;
    }

    button.active,
    button:active {
      transform: translateY(4px);
      box-shadow:
        inset 3px 3px #0005,
        inset -3px -3px #ffffff22,
        0 1px #111;
      background: #76838b;
    }

    #jumpBtn {
      background: #477638e8;
    }

    #overlay {
      position: absolute;
      inset: 0;
      z-index: 8;
      display: grid;
      place-items: center;
      padding: 20px;
      background: #07120dcc;
    }

    #overlay.hidden {
      display: none;
    }

    #card {
      width: min(620px, 94%);
      padding: 28px;
      text-align: center;
      border: 6px solid #1c1c1c;
      background:
        linear-gradient(90deg, #6f492b11 50%, transparent 50%),
        linear-gradient(#6f492b11 50%, transparent 50%),
        #84603a;
      background-size: 32px 32px;
      box-shadow:
        inset 0 0 0 4px #b88a55,
        0 14px 35px #000c;
    }

    h1 {
      margin: 0 0 12px;
      color: #f6e7b0;
      font-size: clamp(27px, 6vw, 52px);
      line-height: 1;
      text-shadow: 4px 4px #352211;
    }

    #message {
      margin: 12px auto 20px;
      line-height: 1.6;
      font-size: clamp(14px, 2.5vw, 19px);
      max-width: 510px;
    }

    #startBtn {
      width: auto;
      height: auto;
      padding: 14px 24px;
      background: #4f8039;
      font-size: 20px;
    }

    @media (max-height: 500px) {
      #gameShell {
        width: 100vw;
        height: 100vh;
        border: 0;
      }

      #controls {
        bottom: 8px;
      }

      button {
        width: 55px;
        height: 55px;
      }
    }
  </style>
</head>

<body>
  <main id="gameShell">
    <canvas id="game" width="1200" height="720"></canvas>

    <div id="hud">
      <div class="hudPanel">
        LEVEL <span id="levelText">1</span>/10
      </div>

      <div class="hudPanel">
        COINS <span id="coinText">0</span>
        &nbsp; HP <span id="healthText">♥♥♥</span>
      </div>
    </div>

    <div id="controls">
      <div class="controlGroup">
        <button id="leftBtn" aria-label="Move left">◀</button>
        <button id="rightBtn" aria-label="Move right">▶</button>
      </div>

      <div class="controlGroup">
        <button id="jumpBtn" aria-label="Jump">▲</button>
      </div>
    </div>

    <div id="overlay">
      <section id="card">
        <h1 id="title">BLOCKBOUND VILLAGER</h1>
        <p id="message">
          Cross ten blocky worlds, collect emerald coins, avoid monsters,
          break yellow bricks, and enter each glowing portal.<br /><br />
          Yellow bricks break after <b>3 upward smacks</b>.<br />
          Use <b>← →</b> or <b>A D</b> to move.<br />
          Use <b>↑</b>, <b>W</b>, or <b>Space</b> to jump.
        </p>
        <button id="startBtn">START ADVENTURE</button>
      </section>
    </div>
  </main>

  <script>
    "use strict";

    const canvas = document.getElementById("game");
    const ctx = canvas.getContext("2d");
    ctx.imageSmoothingEnabled = false;

    const levelText = document.getElementById("levelText");
    const coinText = document.getElementById("coinText");
    const healthText = document.getElementById("healthText");

    const overlay = document.getElementById("overlay");
    const titleElement = document.getElementById("title");
    const messageElement = document.getElementById("message");
    const startButton = document.getElementById("startBtn");

    const TILE = 48;
    const GRAVITY = 1800;
    const MAX_FALL_SPEED = 900;
    const BRICK_MAX_HITS = 3;

    const input = {
      left: false,
      right: false,
      jump: false
    };

    const game = {
      running: false,
      level: 1,
      coins: 0,
      health: 3,
      cameraX: 0,
      time: 0,
      shake: 0
    };

    let world = null;
    let player = null;
    let lastTime = performance.now();

    const clamp = (value, min, max) =>
      Math.max(min, Math.min(max, value));

    const overlap = (a, b) =>
      a.x < b.x + b.w &&
      a.x + a.w > b.x &&
      a.y < b.y + b.h &&
      a.y + a.h > b.y;

    function seededRandom(seed) {
      let state = seed >>> 0;

      return function () {
        state += 0x6D2B79F5;
        let n = state;
        n = Math.imul(n ^ (n >>> 15), n | 1);
        n ^= n + Math.imul(n ^ (n >>> 7), n | 61);
        return ((n ^ (n >>> 14)) >>> 0) / 4294967296;
      };
    }

    function rect(x, y, w, h, color) {
      ctx.fillStyle = color;
      ctx.fillRect(
        Math.round(x),
        Math.round(y),
        Math.round(w),
        Math.round(h)
      );
    }

    function createLevel(levelNumber) {
      const random = seededRandom(7301 + levelNumber * 941);
      const width = 54 + levelNumber * 9;
      const groundRow = 12;

      const tiles = Array.from(
        { length: 15 },
        () => Array(width).fill(0)
      );

      const brickHits = Array.from(
        { length: 15 },
        () => Array(width).fill(0)
      );

      for (let x = 0; x < width; x++) {
        tiles[groundRow][x] = 1;
        tiles[groundRow + 1][x] = 2;
        tiles[groundRow + 2][x] = 2;
      }

      const pits = [];
      const pitCount = 2 + Math.floor(levelNumber * 1.1);

      for (let i = 0; i < pitCount; i++) {
        const maxStart = width - 10;
        const start =
          9 + Math.floor(random() * Math.max(1, maxStart - 9));

        const pitWidth =
          levelNumber < 4
            ? 1 + Math.floor(random() * 2)
            : 1 +
              Math.floor(
                random() * Math.min(4, 2 + levelNumber / 3)
              );

        if (start < 7 || start + pitWidth > width - 6) continue;

        pits.push({
          start,
          width: pitWidth
        });

        for (let x = start; x < start + pitWidth; x++) {
          for (let y = groundRow; y < 15; y++) {
            tiles[y][x] = 0;
          }
        }
      }

      const isPit = x =>
        pits.some(
          pit => x >= pit.start && x < pit.start + pit.width
        );

      const platforms = [];
      const platformCount = 8 + levelNumber * 2;

      for (let i = 0; i < platformCount; i++) {
        const platformWidth = 2 + Math.floor(random() * 5);
        const x = 5 + Math.floor(random() * (width - 12));
        const y =
          10 -
          Math.floor(
            random() *
              Math.min(6, 2 + Math.floor(levelNumber / 2))
          );

        if (x + platformWidth >= width - 4) continue;

        platforms.push({
          x,
          y,
          width: platformWidth
        });

        for (let px = x; px < x + platformWidth; px++) {
          tiles[y][px] = random() < 0.35 ? 3 : 1;
        }
      }

      if (levelNumber >= 4) {
        const stairStart = Math.floor(width * 0.6);

        for (
          let step = 0;
          step < Math.min(5, 2 + levelNumber);
          step++
        ) {
          const x = stairStart + step * 2;
          const y = groundRow - 1 - step;

          for (
            let px = x;
            px < x + 2 && px < width - 4;
            px++
          ) {
            tiles[y][px] = 4;
          }
        }
      }

      const coins = [];

      for (const platform of platforms) {
        if (random() < 0.78) {
          for (
            let x = platform.x;
            x < platform.x + platform.width;
            x += 2
          ) {
            coins.push({
              x: x * TILE + TILE / 2,
              y: platform.y * TILE - 17,
              radius: 11,
              collected: false,
              phase: random() * Math.PI * 2
            });
          }
        }
      }

      for (
        let x = 5;
        x < width - 5;
        x += 5 + Math.floor(random() * 5)
      ) {
        if (!isPit(x)) {
          coins.push({
            x: x * TILE + TILE / 2,
            y: groundRow * TILE - 30,
            radius: 11,
            collected: false,
            phase: random() * Math.PI * 2
          });
        }
      }

      const enemies = [];
      const enemyCount = 3 + levelNumber * 2;

      for (let i = 0; i < enemyCount; i++) {
        const tileX =
          8 + Math.floor(random() * (width - 16));

        if (isPit(tileX)) continue;

        enemies.push({
          x: tileX * TILE + 5,
          y: groundRow * TILE - 38,
          w: 39,
          h: 38,
          vx:
            (random() < 0.5 ? -1 : 1) *
            (65 + levelNumber * 7 + random() * 30),
          minX:
            Math.max(
              3,
              tileX - 3 - Math.floor(random() * 3)
            ) * TILE,
          maxX:
            Math.min(
              width - 4,
              tileX + 4 + Math.floor(random() * 3)
            ) * TILE,
          dead: false,
          color: random() < 0.45 ? "#386c2f" : "#6c4935"
        });
      }

      const spikes = [];

      if (levelNumber >= 2) {
        const spikeCount = levelNumber + 1;

        for (let i = 0; i < spikeCount; i++) {
          const x =
            10 + Math.floor(random() * (width - 20));

          if (!isPit(x)) {
            spikes.push({
              x: x * TILE,
              y: groundRow * TILE - 25,
              w: TILE,
              h: 25
            });
          }
        }
      }

      const movingPlatforms = [];

      if (levelNumber >= 5) {
        const count = Math.floor((levelNumber - 3) / 2);

        for (let i = 0; i < count; i++) {
          const x =
            (15 + Math.floor(random() * (width - 25))) *
            TILE;

          const y =
            (6 + Math.floor(random() * 4)) * TILE;

          movingPlatforms.push({
            x,
            baseX: x,
            y,
            w: TILE * (2 + Math.floor(random() * 2)),
            h: 18,
            range: TILE * (2 + levelNumber / 2),
            speed: 0.8 + random() * 0.7,
            phase: random() * Math.PI * 2,
            previousX: x
          });
        }
      }

      return {
        width,
        height: 15,
        groundRow,
        tiles,
        brickHits,
        coins,
        enemies,
        spikes,
        movingPlatforms,
        spawn: {
          x: TILE * 2,
          y: groundRow * TILE - 58
        },
        goal: {
          x: (width - 3) * TILE,
          y: groundRow * TILE - 96,
          w: 48,
          h: 96
        }
      };
    }

    function createPlayer(spawn) {
      return {
        x: spawn.x,
        y: spawn.y,
        w: 34,
        h: 58,
        vx: 0,
        vy: 0,
        speed: 310,
        jumpPower: 690,
        onGround: false,
        jumpLocked: false,
        facing: 1,
        invulnerable: 0,
        spawnX: spawn.x,
        spawnY: spawn.y
      };
    }

    function loadLevel(levelNumber) {
      game.level = levelNumber;
      game.time = 0;
      world = createLevel(levelNumber);
      player = createPlayer(world.spawn);
      game.cameraX = 0;
      updateHud();
    }

    function updateHud() {
      levelText.textContent = game.level;
      coinText.textContent = game.coins;

      healthText.textContent =
        "♥".repeat(Math.max(0, game.health)) +
        "♡".repeat(Math.max(0, 3 - game.health));
    }

    function startGame() {
      game.running = true;
      game.level = 1;
      game.coins = 0;
      game.health = 3;
      game.time = 0;

      loadLevel(1);
      overlay.classList.add("hidden");
    }

    function showOverlay(title, text, buttonText, action) {
      titleElement.textContent = title;
      messageElement.innerHTML = text;
      startButton.textContent = buttonText;
      startButton.onclick = action;
      overlay.classList.remove("hidden");
    }

    startButton.onclick = startGame;

    function tileAt(column, row) {
      if (column < 0 || column >= world.width) return 2;
      if (row < 0 || row >= world.height) return 0;
      return world.tiles[row][column];
    }

    function isSolidTile(tile) {
      return (
        tile === 1 ||
        tile === 2 ||
        tile === 3 ||
        tile === 4
      );
    }

    function resolveHorizontal(entity, dt) {
      entity.x += entity.vx * dt;

      const top =
        Math.floor((entity.y + 4) / TILE);

      const bottom =
        Math.floor((entity.y + entity.h - 4) / TILE);

      if (entity.vx > 0) {
        const right =
          Math.floor((entity.x + entity.w) / TILE);

        for (let row = top; row <= bottom; row++) {
          if (isSolidTile(tileAt(right, row))) {
            entity.x =
              right * TILE - entity.w - 0.01;

            entity.vx = 0;
            break;
          }
        }
      } else if (entity.vx < 0) {
        const left = Math.floor(entity.x / TILE);

        for (let row = top; row <= bottom; row++) {
          if (isSolidTile(tileAt(left, row))) {
            entity.x =
              (left + 1) * TILE + 0.01;

            entity.vx = 0;
            break;
          }
        }
      }
    }

    function smackBrick(column, row) {
      if (tileAt(column, row) !== 3) return;

      world.brickHits[row][column]++;

      if (
        world.brickHits[row][column] >=
        BRICK_MAX_HITS
      ) {
        world.tiles[row][column] = 0;
        world.brickHits[row][column] = 0;
        game.coins++;
        game.shake = 10;
        updateHud();
      } else {
        game.shake = 4;
      }
    }

    function resolveVertical(entity, dt) {
      entity.y += entity.vy * dt;
      entity.onGround = false;

      const left =
        Math.floor((entity.x + 5) / TILE);

      const right =
        Math.floor((entity.x + entity.w - 5) / TILE);

      if (entity.vy > 0) {
        const bottom =
          Math.floor((entity.y + entity.h) / TILE);

        for (
          let column = left;
          column <= right;
          column++
        ) {
          if (isSolidTile(tileAt(column, bottom))) {
            entity.y =
              bottom * TILE - entity.h - 0.01;

            entity.vy = 0;
            entity.onGround = true;
            break;
          }
        }
      } else if (entity.vy < 0) {
        const top = Math.floor(entity.y / TILE);
        const hitColumns = [];

        for (
          let column = left;
          column <= right;
          column++
        ) {
          if (isSolidTile(tileAt(column, top))) {
            hitColumns.push(column);
          }
        }

        if (hitColumns.length > 0) {
          const playerCenter =
            entity.x + entity.w / 2;

          hitColumns.sort((a, b) => {
            const centerA =
              a * TILE + TILE / 2;

            const centerB =
              b * TILE + TILE / 2;

            return (
              Math.abs(centerA - playerCenter) -
              Math.abs(centerB - playerCenter)
            );
          });

          smackBrick(hitColumns[0], top);

          entity.y =
            (top + 1) * TILE + 0.01;

          entity.vy = 0;
        }
      }
    }

    function updateMovingPlatforms(dt) {
      for (const platform of world.movingPlatforms) {
        platform.previousX = platform.x;

        platform.x =
          platform.baseX +
          Math.sin(
            game.time * platform.speed +
              platform.phase
          ) *
            platform.range;

        const playerBottom =
          player.y + player.h;

        const wasAbove =
          playerBottom <= platform.y + 12;

        const horizontallyAligned =
          player.x + player.w > platform.x &&
          player.x < platform.x + platform.w;

        if (
          player.vy >= 0 &&
          wasAbove &&
          horizontallyAligned &&
          playerBottom + player.vy * dt >= platform.y
        ) {
          player.y =
            platform.y - player.h;

          player.vy = 0;
          player.onGround = true;

          player.x +=
            platform.x - platform.previousX;
        }
      }
    }

    function updateEnemies(dt) {
      for (const enemy of world.enemies) {
        if (enemy.dead) continue;

        enemy.x += enemy.vx * dt;

        if (
          enemy.x <= enemy.minX ||
          enemy.x + enemy.w >= enemy.maxX
        ) {
          enemy.vx *= -1;

          enemy.x = clamp(
            enemy.x,
            enemy.minX,
            enemy.maxX - enemy.w
          );
        }

        const footColumn =
          enemy.vx > 0
            ? Math.floor(
                (enemy.x + enemy.w + 4) / TILE
              )
            : Math.floor(
                (enemy.x - 4) / TILE
              );

        const footRow =
          Math.floor(
            (enemy.y + enemy.h + 5) / TILE
          );

        if (
          !isSolidTile(
            tileAt(footColumn, footRow)
          )
        ) {
          enemy.vx *= -1;
        }

        if (overlap(player, enemy)) {
          const playerBottom =
            player.y + player.h;

          if (
            player.vy > 100 &&
            playerBottom - enemy.y < 25
          ) {
            enemy.dead = true;
            player.vy = -390;
            game.coins += 2;
            game.shake = 7;
            updateHud();
          } else {
            hurtPlayer(enemy.x);
          }
        }
      }
    }

    function collectCoins() {
      for (const coin of world.coins) {
        if (coin.collected) continue;

        const hitbox = {
          x: coin.x - coin.radius,
          y: coin.y - coin.radius,
          w: coin.radius * 2,
          h: coin.radius * 2
        };

        if (overlap(player, hitbox)) {
          coin.collected = true;
          game.coins++;
          updateHud();
        }
      }
    }

    function checkSpikes() {
      for (const spike of world.spikes) {
        if (overlap(player, spike)) {
          hurtPlayer(spike.x);
          break;
        }
      }
    }

    function hurtPlayer(sourceX) {
      if (player.invulnerable > 0) return;

      game.health--;
      player.invulnerable = 1.5;
      player.vy = -420;
      player.vx =
        player.x < sourceX ? -330 : 330;

      game.shake = 12;
      updateHud();

      if (game.health <= 0) {
        game.running = false;

        showOverlay(
          "ADVENTURE OVER",
          `You reached level <b>${game.level}</b>
          and collected <b>${game.coins}</b>
          emerald coins.`,
          "TRY AGAIN",
          startGame
        );
      }
    }

    function respawnAfterFall() {
      game.health--;
      updateHud();

      if (game.health <= 0) {
        game.running = false;

        showOverlay(
          "LOST IN THE VOID",
          `You reached level <b>${game.level}</b>
          and collected <b>${game.coins}</b>
          emerald coins.`,
          "TRY AGAIN",
          startGame
        );

        return;
      }

      player.x = player.spawnX;
      player.y = player.spawnY;
      player.vx = 0;
      player.vy = 0;
      player.invulnerable = 1.5;

      game.cameraX = Math.max(
        0,
        player.x - canvas.width * 0.3
      );
    }

    function completeLevel() {
      game.running = false;

      if (game.level >= 10) {
        showOverlay(
          "MASTER BUILDER!",
          `You conquered all ten worlds and collected
          <b>${game.coins}</b> emerald coins.`,
          "PLAY AGAIN",
          startGame
        );

        return;
      }

      const nextLevel = game.level + 1;

      showOverlay(
        `LEVEL ${game.level} COMPLETE`,
        `The next world is harder.<br />
        Emerald coins collected:
        <b>${game.coins}</b>`,
        `ENTER LEVEL ${nextLevel}`,
        () => {
          game.health =
            Math.min(3, game.health + 1);

          loadLevel(nextLevel);
          game.running = true;
          overlay.classList.add("hidden");
        }
      );
    }

    function update(dt) {
      if (
        !game.running ||
        !world ||
        !player
      ) {
        return;
      }

      game.time += dt;

      player.invulnerable = Math.max(
        0,
        player.invulnerable - dt
      );

      const acceleration =
        player.onGround ? 2300 : 1400;

      const desiredVelocity =
        (input.right ? 1 : 0) -
        (input.left ? 1 : 0);

      if (desiredVelocity !== 0) {
        player.vx +=
          desiredVelocity * acceleration * dt;

        player.facing = desiredVelocity;
      } else {
        player.vx *= Math.pow(
          player.onGround ? 0.0008 : 0.06,
          dt
        );
      }

      player.vx = clamp(
        player.vx,
        -player.speed,
        player.speed
      );

      if (
        input.jump &&
        player.onGround &&
        !player.jumpLocked
      ) {
        player.vy = -player.jumpPower;
        player.onGround = false;
        player.jumpLocked = true;
      }

      if (!input.jump) {
        player.jumpLocked = false;

        if (player.vy < -180) {
          player.vy += 1200 * dt;
        }
      }

      player.vy = Math.min(
        MAX_FALL_SPEED,
        player.vy + GRAVITY * dt
      );

      resolveHorizontal(player, dt);
      resolveVertical(player, dt);
      updateMovingPlatforms(dt);
      updateEnemies(dt);
      collectCoins();
      checkSpikes();

      if (player.y > canvas.height + 180) {
        respawnAfterFall();
      }

      if (
        game.running &&
        overlap(player, world.goal)
      ) {
        completeLevel();
      }

      const desiredCamera =
        player.x -
        canvas.width * 0.38 +
        player.vx * 0.3;

      const maxCamera = Math.max(
        0,
        world.width * TILE - canvas.width
      );

      game.cameraX +=
        (
          clamp(
            desiredCamera,
            0,
            maxCamera
          ) - game.cameraX
        ) *
        Math.min(1, dt * 6);

      game.shake *= Math.pow(0.01, dt);
    }

    function drawSky() {
      const gradient =
        ctx.createLinearGradient(
          0,
          0,
          0,
          canvas.height
        );

      gradient.addColorStop(0, "#70c7f4");
      gradient.addColorStop(0.65, "#aee2fa");
      gradient.addColorStop(1, "#d6f4ff");

      ctx.fillStyle = gradient;
      ctx.fillRect(
        0,
        0,
        canvas.width,
        canvas.height
      );

      const sunX =
        canvas.width -
        140 -
        game.cameraX * 0.03;

      rect(sunX, 75, 72, 72, "#fff2a5");
      rect(
        sunX + 9,
        84,
        54,
        54,
        "#ffe36e"
      );

      drawCloud(
        130 - game.cameraX * 0.12,
        120,
        1.1
      );

      drawCloud(
        550 - game.cameraX * 0.1,
        180,
        0.85
      );

      drawCloud(
        980 - game.cameraX * 0.15,
        92,
        1.25
      );

      drawMountainLayer(
        0.08,
        420,
        "#72916a",
        130
      );

      drawMountainLayer(
        0.16,
        505,
        "#456f51",
        105
      );
    }

    function drawCloud(x, y, scale) {
      const wrapWidth = canvas.width + 300;

      x =
        ((x % wrapWidth) + wrapWidth) %
          wrapWidth -
        120;

      rect(
        x,
        y + 18 * scale,
        100 * scale,
        28 * scale,
        "#ffffffd9"
      );

      rect(
        x + 20 * scale,
        y,
        42 * scale,
        48 * scale,
        "#ffffffdf"
      );

      rect(
        x + 58 * scale,
        y + 8 * scale,
        54 * scale,
        38 * scale,
        "#ffffffdf"
      );
    }

    function drawMountainLayer(
      parallax,
      baseY,
      color,
      size
    ) {
      ctx.fillStyle = color;
      ctx.beginPath();
      ctx.moveTo(0, canvas.height);

      const offset =
        -((game.cameraX * parallax) % size);

      for (
        let x = offset - size;
        x < canvas.width + size;
        x += size
      ) {
        ctx.lineTo(x, baseY);
        ctx.lineTo(
          x + size * 0.5,
          baseY - size * 0.75
        );
        ctx.lineTo(x + size, baseY);
      }

      ctx.lineTo(
        canvas.width,
        canvas.height
      );

      ctx.closePath();
      ctx.fill();
    }

    function drawBrickCracks(x, y, hits) {
      if (hits <= 0) return;

      ctx.strokeStyle = "#5f3b12";
      ctx.lineWidth = 3;
      ctx.lineCap = "square";
      ctx.lineJoin = "miter";

      if (hits >= 1) {
        ctx.beginPath();
        ctx.moveTo(x + 23, y + 4);
        ctx.lineTo(x + 18, y + 15);
        ctx.lineTo(x + 25, y + 23);
        ctx.stroke();
      }

      if (hits >= 2) {
        ctx.beginPath();
        ctx.moveTo(x + 25, y + 23);
        ctx.lineTo(x + 15, y + 31);
        ctx.lineTo(x + 20, y + 44);

        ctx.moveTo(x + 25, y + 23);
        ctx.lineTo(x + 37, y + 29);
        ctx.lineTo(x + 32, y + 41);
        ctx.stroke();
      }
    }

    function drawTile(
      tile,
      x,
      y,
      column,
      row
    ) {
      if (tile === 1) {
        rect(
          x,
          y,
          TILE,
          TILE,
          "#7a4d2b"
        );

        rect(
          x,
          y,
          TILE,
          12,
          "#65a83c"
        );

        rect(
          x,
          y + 12,
          TILE,
          5,
          "#4f8530"
        );

        rect(
          x + 7,
          y + 23,
          12,
          8,
          "#5c381e"
        );

        rect(
          x + 29,
          y + 34,
          15,
          8,
          "#98633b"
        );
      } else if (tile === 2) {
        rect(
          x,
          y,
          TILE,
          TILE,
          "#75492b"
        );

        rect(
          x + 3,
          y + 3,
          20,
          17,
          "#835536"
        );

        rect(
          x + 27,
          y + 4,
          17,
          24,
          "#654026"
        );

        rect(
          x + 6,
          y + 26,
          17,
          18,
          "#684127"
        );

        rect(
          x + 27,
          y + 31,
          17,
          13,
          "#8d5b36"
        );
      } else if (tile === 3) {
        rect(
          x,
          y,
          TILE,
          TILE,
          "#bf8c29"
        );

        rect(
          x + 4,
          y + 4,
          TILE - 8,
          TILE - 8,
          "#dfaa34"
        );

        rect(
          x + 18,
          y + 11,
          12,
          7,
          "#fff0a3"
        );

        rect(
          x + 22,
          y + 18,
          7,
          15,
          "#8d611a"
        );

        rect(
          x + 21,
          y + 36,
          8,
          6,
          "#fff0a3"
        );

        const hits =
          world.brickHits?.[row]?.[column] || 0;

        drawBrickCracks(x, y, hits);
      } else if (tile === 4) {
        rect(
          x,
          y,
          TILE,
          TILE,
          "#858b8c"
        );

        rect(
          x + 3,
          y + 3,
          19,
          18,
          "#a2a7a8"
        );

        rect(
          x + 26,
          y + 3,
          19,
          12,
          "#73797a"
        );

        rect(
          x + 5,
          y + 25,
          16,
          18,
          "#6f7577"
        );

        rect(
          x + 25,
          y + 19,
          19,
          24,
          "#969c9d"
        );
      }

      ctx.strokeStyle = "#00000025";
      ctx.lineWidth = 2;

      ctx.strokeRect(
        Math.round(x),
        Math.round(y),
        TILE,
        TILE
      );
    }

    function drawCoins() {
      for (const coin of world.coins) {
        if (coin.collected) continue;

        const x =
          coin.x - game.cameraX;

        const y =
          coin.y +
          Math.sin(
            game.time * 5 + coin.phase
          ) *
            5;

        const squish =
          0.35 +
          Math.abs(
            Math.sin(
              game.time * 4 + coin.phase
            )
          ) *
            0.65;

        rect(
          x - 12 * squish,
          y - 15,
          24 * squish,
          30,
          "#116e3b"
        );

        rect(
          x - 8 * squish,
          y - 11,
          16 * squish,
          22,
          "#36d26f"
        );

        rect(
          x - 4 * squish,
          y - 7,
          8 * squish,
          14,
          "#b8ffd2"
        );
      }
    }

    function drawEnemy(enemy) {
      if (enemy.dead) return;

      const x =
        enemy.x - game.cameraX;

      const y = enemy.y;

      rect(
        x + 3,
        y + 8,
        enemy.w - 6,
        enemy.h - 8,
        enemy.color
      );

      rect(
        x,
        y + 14,
        enemy.w,
        17,
        enemy.color
      );

      rect(
        x + 6,
        y + 2,
        27,
        11,
        "#476f35"
      );

      const facingRight =
        enemy.vx > 0;

      const eyeX =
        facingRight ? x + 24 : x + 8;

      rect(
        eyeX,
        y + 14,
        6,
        7,
        "#f2f2d7"
      );

      rect(
        eyeX + (facingRight ? 3 : 0),
        y + 16,
        3,
        5,
        "#111"
      );

      rect(
        x + 8,
        y + 31,
        8,
        7,
        "#241711"
      );

      rect(
        x + 25,
        y + 31,
        8,
        7,
        "#241711"
      );
    }

    function drawSpikes() {
      for (const spike of world.spikes) {
        const x =
          spike.x - game.cameraX;

        const y = spike.y;

        ctx.fillStyle = "#c8d0d2";

        for (let i = 0; i < 3; i++) {
          const sx = x + i * 16;

          ctx.beginPath();
          ctx.moveTo(
            sx,
            y + spike.h
          );

          ctx.lineTo(
            sx + 8,
            y
          );

          ctx.lineTo(
            sx + 16,
            y + spike.h
          );

          ctx.closePath();
          ctx.fill();
        }

        rect(
          x,
          y + spike.h - 4,
          spike.w,
          4,
          "#666"
        );
      }
    }

    function drawMovingPlatforms() {
      for (
        const platform of world.movingPlatforms
      ) {
        const x =
          platform.x - game.cameraX;

        rect(
          x,
          platform.y,
          platform.w,
          platform.h,
          "#3c5264"
        );

        rect(
          x,
          platform.y,
          platform.w,
          5,
          "#82a1b5"
        );

        for (
          let px = 8;
          px < platform.w;
          px += 28
        ) {
          rect(
            x + px,
            platform.y + 8,
            6,
            6,
            "#1b2730"
          );
        }
      }
    }

    function drawGoal() {
      const x =
        world.goal.x - game.cameraX;

      const y = world.goal.y;

      rect(
        x + 8,
        y,
        32,
        96,
        "#25222b"
      );

      rect(
        x + 12,
        y + 7,
        24,
        82,
        "#7540b8"
      );

      const glow =
        0.5 +
        Math.sin(game.time * 5) * 0.18;

      ctx.fillStyle =
        `rgba(192, 105, 255, ${glow})`;

      ctx.fillRect(
        x + 16,
        y + 12,
        16,
        72
      );

      rect(
        x,
        y - 8,
        48,
        10,
        "#1a181d"
      );

      rect(
        x,
        y + 92,
        48,
        10,
        "#1a181d"
      );
    }

    function drawVillager() {
      if (
        player.invulnerable > 0 &&
        Math.floor(
          player.invulnerable * 12
        ) %
          2 ===
          0
      ) {
        return;
      }

      const x =
        player.x - game.cameraX;

      const y = player.y;
      const flip = player.facing < 0;

      ctx.save();

      if (flip) {
        ctx.translate(
          x + player.w / 2,
          0
        );

        ctx.scale(-1, 1);

        ctx.translate(
          -(x + player.w / 2),
          0
        );
      }

      rect(
        x + 4,
        y,
        26,
        24,
        "#b9875e"
      );

      rect(
        x + 1,
        y + 5,
        32,
        10,
        "#a87753"
      );

      rect(
        x + 24,
        y + 10,
        11,
        10,
        "#c8956d"
      );

      rect(
        x + 8,
        y + 7,
        5,
        5,
        "#f3f0de"
      );

      rect(
        x + 10,
        y + 8,
        3,
        4,
        "#2d8f55"
      );

      rect(
        x + 20,
        y + 7,
        5,
        5,
        "#f3f0de"
      );

      rect(
        x + 20,
        y + 8,
        3,
        4,
        "#2d8f55"
      );

      rect(
        x + 4,
        y + 24,
        26,
        27,
        "#72512e"
      );

      rect(
        x + 2,
        y + 29,
        30,
        16,
        "#88653d"
      );

      rect(
        x + 8,
        y + 34,
        19,
        7,
        "#b58a5d"
      );

      const stride =
        player.onGround &&
        Math.abs(player.vx) > 30
          ? Math.sin(game.time * 13) * 3
          : 0;

      rect(
        x + 6,
        y + 50,
        9,
        8 + stride,
        "#3d2d20"
      );

      rect(
        x + 20,
        y + 50,
        9,
        8 - stride,
        "#3d2d20"
      );

      ctx.restore();
    }

    function drawWorld() {
      const startColumn = Math.max(
        0,
        Math.floor(game.cameraX / TILE) - 1
      );

      const endColumn = Math.min(
        world.width - 1,
        Math.ceil(
          (game.cameraX + canvas.width) / TILE
        ) + 1
      );

      for (
        let row = 0;
        row < world.height;
        row++
      ) {
        for (
          let column = startColumn;
          column <= endColumn;
          column++
        ) {
          const tile =
            world.tiles[row][column];

          if (tile) {
            drawTile(
              tile,
              column * TILE - game.cameraX,
              row * TILE,
              column,
              row
            );
          }
        }
      }

      drawGoal();
      drawCoins();
      drawSpikes();
      drawMovingPlatforms();

      for (const enemy of world.enemies) {
        drawEnemy(enemy);
      }

      drawVillager();
    }

    function drawLevelBanner() {
      if (
        game.time > 2 ||
        !game.running
      ) {
        return;
      }

      const alpha = clamp(
        1 - Math.max(0, game.time - 1),
        0,
        1
      );

      ctx.save();
      ctx.globalAlpha = alpha;
      ctx.textAlign = "center";
      ctx.font = "bold 34px monospace";
      ctx.fillStyle = "#fff";
      ctx.strokeStyle = "#1b1b1b";
      ctx.lineWidth = 7;

      const text =
        `LEVEL ${game.level} — ` +
        difficultyName(game.level);

      ctx.strokeText(
        text,
        canvas.width / 2,
        125
      );

      ctx.fillText(
        text,
        canvas.width / 2,
        125
      );

      ctx.restore();
    }

    function difficultyName(level) {
      const names = [
        "Meadow Start",
        "Broken Ground",
        "Monster March",
        "Stone Steps",
        "Moving Blocks",
        "Spike Valley",
        "High Platforms",
        "Danger Run",
        "Portal Gauntlet",
        "Master World"
      ];

      return names[level - 1];
    }

    function render() {
      ctx.save();

      if (game.shake > 0.5) {
        ctx.translate(
          (Math.random() - 0.5) *
            game.shake,
          (Math.random() - 0.5) *
            game.shake
        );
      }

      drawSky();

      if (world && player) {
        drawWorld();
        drawLevelBanner();
      }

      ctx.restore();
    }

    function gameLoop(now) {
      const dt = Math.min(
        0.033,
        (now - lastTime) / 1000
      );

      lastTime = now;

      update(dt);
      render();

      requestAnimationFrame(gameLoop);
    }

    function setKey(key, pressed) {
      if (
        key === "arrowleft" ||
        key === "a"
      ) {
        input.left = pressed;
      }

      if (
        key === "arrowright" ||
        key === "d"
      ) {
        input.right = pressed;
      }

      if (
        key === "arrowup" ||
        key === "w" ||
        key === " " ||
        key === "spacebar"
      ) {
        input.jump = pressed;
      }
    }

    window.addEventListener(
      "keydown",
      event => {
        const key =
          event.key.toLowerCase();

        if (
          [
            "arrowleft",
            "arrowright",
            "arrowup",
            "a",
            "d",
            "w",
            " ",
            "spacebar"
          ].includes(key)
        ) {
          event.preventDefault();
        }

        setKey(key, true);
      }
    );

    window.addEventListener(
      "keyup",
      event => {
        setKey(
          event.key.toLowerCase(),
          false
        );
      }
    );

    window.addEventListener(
      "blur",
      () => {
        input.left = false;
        input.right = false;
        input.jump = false;
      }
    );

    function bindControl(
      buttonId,
      property
    ) {
      const button =
        document.getElementById(buttonId);

      const press = event => {
        event.preventDefault();
        input[property] = true;
        button.classList.add("active");

        if (
          button.setPointerCapture &&
          event.pointerId !== undefined
        ) {
          button.setPointerCapture(
            event.pointerId
          );
        }
      };

      const release = event => {
        event.preventDefault();
        input[property] = false;
        button.classList.remove("active");
      };

      button.addEventListener(
        "pointerdown",
        press
      );

      button.addEventListener(
        "pointerup",
        release
      );

      button.addEventListener(
        "pointercancel",
        release
      );

      button.addEventListener(
        "lostpointercapture",
        release
      );

      button.addEventListener(
        "contextmenu",
        event => event.preventDefault()
      );
    }

    bindControl("leftBtn", "left");
    bindControl("rightBtn", "right");
    bindControl("jumpBtn", "jump");

    function resizeCanvas() {
      const shell =
        document.getElementById("gameShell");

      const bounds =
        shell.getBoundingClientRect();

      const ratio = Math.min(
        2,
        window.devicePixelRatio || 1
      );

      canvas.width = Math.max(
        640,
        Math.floor(bounds.width * ratio)
      );

      canvas.height = Math.max(
        360,
        Math.floor(bounds.height * ratio)
      );

      ctx.imageSmoothingEnabled = false;
    }

    window.addEventListener(
      "resize",
      resizeCanvas
    );

    resizeCanvas();
    requestAnimationFrame(gameLoop);
  </script>
</body>
</html>
