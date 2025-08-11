# Certified-Cyclone-Pac-Man-
Old retro Pac-Man game 
const CELL_SIZE = 20;
const GRID_WIDTH = 21;
const GRID_HEIGHT = 18;
const MAZE_OFFSET_X = 0;
const MAZE_OFFSET_Y = 40;
const GAME_WIDTH = GRID_WIDTH * CELL_SIZE;
const GAME_HEIGHT = MAZE_OFFSET_Y + GRID_HEIGHT * CELL_SIZE;
const DOT_RADIUS = 3;
const PELLET_RADIUS = 6;
const PLAYER_RADIUS = 8;
const GHOST_SIZE = 16;
const PLAYER_SPEED = 2;
const GHOST_SPEED = 1.6;
const VULNERABLE_DURATION = 8000;
const GHOST_RESPAWN_DELAY = 3000;
const POWER_PELLET_FLASH_INTERVAL = 300;
const GHOST_FLASH_INTERVAL = 200;
const PLAYER_LIVES = 3;
const GameState = {
  LOADING: 0,
  READY: 1,
  PLAY: 2,
  GAME_OVER: 3,
  WIN: 4
};
const Direction = {
  NONE: 0,
  LEFT: 1,
  RIGHT: 2,
  UP: 3,
  DOWN: 4
};
const GHOST_COLORS = [0xff3333, 0xffb8de, 0x00ffff, 0xffa500];
const MAZE = [
  [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
  [1,3,0,0,0,1,0,0,0,1,0,1,0,0,0,1,0,0,0,3,1],
  [1,0,1,1,0,1,0,1,0,1,0,1,0,1,0,1,0,1,1,0,1],
  [1,0,1,1,0,0,0,1,0,0,0,0,0,1,0,0,0,1,1,0,1],
  [1,0,0,0,0,1,0,0,0,1,1,1,0,0,0,1,0,0,0,0,1],
  [1,0,1,1,0,1,1,1,0,1,0,1,0,1,1,1,0,1,1,0,1],
  [1,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,1],
  [1,1,0,1,0,1,1,1,1,1,0,1,1,1,1,1,0,1,0,1,1],
  [0,1,0,0,0,1,0,0,0,0,0,0,0,0,0,1,0,0,0,1,0],
  [1,1,1,1,0,1,0,1,1,1,0,1,1,1,0,1,0,1,1,1,1],
  [1,3,0,0,0,0,0,1,0,0,0,0,0,1,0,0,0,0,0,3,1],
  [1,0,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,0,1],
  [1,0,0,1,0,0,0,0,0,1,1,0,0,0,0,0,0,1,0,0,1],
  [1,1,0,1,1,1,0,1,1,1,1,1,1,1,0,1,1,1,0,1,1],
  [1,0,0,0,0,1,0,0,0,0,0,0,0,0,0,1,0,0,0,0,1],
  [1,0,1,1,0,1,1,1,1,1,0,1,1,1,1,1,0,1,1,0,1],
  [1,3,0,0,0,0,0,1,0,0,0,0,0,1,0,0,0,0,0,3,1],
  [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]
];
class Maze {
  constructor(layer) {
    this.layer = layer;
    this.draw();
  }
  draw() {
    this.layer.removeChildren();
    for (let y = 0; y < GRID_HEIGHT; y++) {
      for (let x = 0; x < GRID_WIDTH; x++) {
        if (MAZE[y][x] === 1) {
          const g = new PIXI.Graphics();
          g.beginFill(0x162d66);
          g.drawRect(x * CELL_SIZE, y * CELL_SIZE, CELL_SIZE, CELL_SIZE);
          g.endFill();
          this.layer.addChild(g);
        }
      }
    }
  }
  isWall(x, y) {
    if (x < 0 || y < 0 || x >= GRID_WIDTH || y >= GRID_HEIGHT) return true;
    return MAZE[y][x] === 1;
  }
  isValid(x, y) {
    return !this.isWall(x, y);
  }
}
class DotManager {
  constructor(layer) {
    this.layer = layer;
    this.dots = [];
    this.count = 0;
    this.reset();
  }
  reset() {
    this.layer.removeChildren();
    this.dots = [];
    this.count = 0;
    for (let y = 0; y < GRID_HEIGHT; y++) {
      for (let x = 0; x < GRID_WIDTH; x++) {
        if (MAZE[y][x] === 0) {
          const g = new PIXI.Graphics();
          g.beginFill(0xffffff);
          g.drawCircle(x * CELL_SIZE + CELL_SIZE / 2, y * CELL_SIZE + CELL_SIZE / 2, DOT_RADIUS);
          g.endFill();
          this.layer.addChild(g);
          this.dots.push({x, y, g});
          this.count++;
        }
      }
    }
  }
  eatDot(px, py) {
    for (let i = 0; i < this.dots.length; i++) {
      const d = this.dots[i];
      if (d.x === px && d.y === py) {
        this.layer.removeChild(d.g);
        this.dots.splice(i, 1);
        this.count--;
        return true;
      }
    }
    return false;
  }
  remaining() {
    return this.count;
  }
}
class PelletManager {
  constructor(layer) {
    this.layer = layer;
    this.pellets = [];
    this.flashTime = 0;
    this.flashOn = true;
    this.reset();
  }
  reset() {
    this.layer.removeChildren();
    this.pellets = [];
    for (let y = 0; y < GRID_HEIGHT; y++) {
      for (let x = 0; x < GRID_WIDTH; x++) {
        if (MAZE[y][x] === 3) {
          const g = new PIXI.Graphics();
          g.beginFill(0xffffff);
          g.drawCircle(x * CELL_SIZE + CELL_SIZE / 2, y * CELL_SIZE + CELL_SIZE / 2, PELLET_RADIUS);
          g.endFill();
          this.layer.addChild(g);
          this.pellets.push({x, y, g, visible: true});
        }
      }
    }
    this.flashTime = 0;
    this.flashOn = true;
  }
  update(dt) {
    this.flashTime += dt;
    if (this.flashTime > POWER_PELLET_FLASH_INTERVAL) {
      this.flashOn = !this.flashOn;
      this.flashTime = 0;
      for (const p of this.pellets) {
        p.g.alpha = this.flashOn ? 1 : 0.3;
      }
    }
  }
  eatPellet(px, py) {
    for (let i = 0; i < this.pellets.length; i++) {
      const p = this.pellets[i];
      if (p.x === px && p.y === py && p.visible) {
        this.layer.removeChild(p.g);
        this.pellets.splice(i, 1);
        return true;
      }
    }
    return false;
  }
}
class Player {
  constructor(layer) {
    this.layer = layer;
    this.gfxOpen = new PIXI.Graphics();
    this.gfxClosed = new PIXI.Graphics();
    this.gfxOpen.visible = true;
    this.gfxClosed.visible = false;
    this.layer.addChild(this.gfxOpen);
    this.layer.addChild(this.gfxClosed);
    this.x = 10;
    this.y = 15;
    this.dir = Direction.NONE;
    this.nextDir = Direction.NONE;
    this.mouthAnim = 0;
    this.mouthOpen = true;
    this.alive = true;
    this.reset();
  }
  reset() {
    this.x = 10;
    this.y = 15;
    this.pixelX = this.x * CELL_SIZE + CELL_SIZE / 2;
    this.pixelY = this.y * CELL_SIZE + CELL_SIZE / 2;
    this.dir = Direction.NONE;
    this.nextDir = Direction.NONE;
    this.mouthAnim = 0;
    this.mouthOpen = true;
    this.alive = true;
    this.draw();
  }
  setDirection(dir) {
    this.nextDir = dir;
  }
  update(maze, dt) {
    if (!this.alive) return;
    this.mouthAnim += dt;
    if (this.mouthAnim > 120) {
      this.mouthOpen = !this.mouthOpen;
      this.mouthAnim = 0;
      this.gfxOpen.visible = this.mouthOpen;
      this.gfxClosed.visible = !this.mouthOpen;
    }
    let gridX = Math.round((this.pixelX - CELL_SIZE / 2) / CELL_SIZE);
    let gridY = Math.round((this.pixelY - CELL_SIZE / 2) / CELL_SIZE);
    if (this.nextDir !== Direction.NONE && this.canMove(maze, gridX, gridY, this.nextDir)) {
      this.dir = this.nextDir;
      this.nextDir = Direction.NONE;
    }
    if (this.dir !== Direction.NONE && this.canMove(maze, gridX, gridY, this.dir)) {
      const move = this.getMoveDelta(this.dir, PLAYER_SPEED);
      this.pixelX += move.dx;
      this.pixelY += move.dy;
      let newGridX = Math.round((this.pixelX - CELL_SIZE / 2) / CELL_SIZE);
      let newGridY = Math.round((this.pixelY - CELL_SIZE / 2) / CELL_SIZE);
      if (maze.isWall(newGridX, newGridY)) {
        this.pixelX -= move.dx;
        this.pixelY -= move.dy;
      }
    }
    if (this.pixelX < 0) this.pixelX = GAME_WIDTH - CELL_SIZE / 2;
    if (this.pixelX > GAME_WIDTH) this.pixelX = CELL_SIZE / 2;
    this.x = Math.round((this.pixelX - CELL_SIZE / 2) / CELL_SIZE);
    this.y = Math.round((this.pixelY - CELL_SIZE / 2) / CELL_SIZE);
    this.draw();
  }
  canMove(maze, x, y, dir) {
    let nx = x;
    let ny = y;
    if (dir === Direction.LEFT) nx--;
    else if (dir === Direction.RIGHT) nx++;
    else if (dir === Direction.UP) ny--;
    else if (dir === Direction.DOWN) ny++;
    return !maze.isWall(nx, ny);
  }
  getMoveDelta(dir, speed) {
    if (dir === Direction.LEFT) return {dx: -speed, dy: 0};
    if (dir === Direction.RIGHT) return {dx: speed, dy: 0};
    if (dir === Direction.UP) return {dx: 0, dy: -speed};
    if (dir === Direction.DOWN) return {dx: 0, dy: speed};
    return {dx: 0, dy: 0};
  }
  draw() {
    this.gfxOpen.clear();
    this.gfxOpen.beginFill(0xffeb3b);
    let startAngle = 0.25 * Math.PI;
    let endAngle = 1.75 * Math.PI;
    let rot = 0;
    if (this.dir === Direction.LEFT) rot = Math.PI;
    else if (this.dir === Direction.UP) rot = -0.5 * Math.PI;
    else if (this.dir === Direction.DOWN) rot = 0.5 * Math.PI;
    this.gfxOpen.moveTo(this.pixelX, this.pixelY);
    this.gfxOpen.arc(this.pixelX, this.pixelY, PLAYER_RADIUS, startAngle + rot, endAngle + rot);
    this.gfxOpen.lineTo(this.pixelX, this.pixelY);
    this.gfxOpen.endFill();
    this.gfxClosed.clear();
    this.gfxClosed.beginFill(0xffeb3b);
    this.gfxClosed.drawCircle(this.pixelX, this.pixelY, PLAYER_RADIUS);
    this.gfxClosed.endFill();
  }
}
class Ghost {
  constructor(layer, color, sx, sy) {
    this.layer = layer;
    this.color = color;
    this.sx = sx;
    this.sy = sy;
    this.x = sx;
    this.y = sy;
    this.pixelX = this.x * CELL_SIZE + CELL_SIZE / 2;
    this.pixelY = this.y * CELL_SIZE + CELL_SIZE / 2;
    this.dir = Direction.LEFT;
    this.nextDir = Direction.NONE;
    this.vulnerable = false;
    this.vulTime = 0;
    this.respawning = false;
    this.respawnTime = 0;
    this.gfx = new PIXI.Graphics();
    this.legAnim = 0;
    this.legFrame = false;
    this.flashOn = false;
    this.layer.addChild(this.gfx);
    this.visible = true;
    this.draw();
  }
  reset() {
    this.x = this.sx;
    this.y = this.sy;
    this.pixelX = this.x * CELL_SIZE + CELL_SIZE / 2;
    this.pixelY = this.y * CELL_SIZE + CELL_SIZE / 2;
    this.dir = Direction.LEFT;
    this.vulnerable = false;
    this.vulTime = 0;
    this.respawning = false;
    this.respawnTime = 0;
    this.visible = true;
    this.flashOn = false;
    this.legAnim = 0;
    this.legFrame = false;
    this.gfx.visible = true;
    this.draw();
  }
  setVulnerable(vul) {
    this.vulnerable = vul;
    this.vulTime = 0;
    this.flashOn = false;
  }
  startRespawn() {
    this.respawning = true;
    this.respawnTime = 0;
    this.visible = false;
    this.gfx.visible = false;
  }
  update(maze, player, dt, globalVul, vulTimeLeft) {
    if (this.respawning) {
      this.respawnTime += dt;
      if (this.respawnTime > GHOST_RESPAWN_DELAY) {
        this.reset();
      }
      return;
    }
    if (!this.visible) return;
    if (globalVul) {
      this.vulnerable = true;
      this.vulTime = vulTimeLeft;
    } else {
      this.vulnerable = false;
    }
    this.legAnim += dt;
    if (this.legAnim > 120) {
      this.legFrame = !this.legFrame;
      this.legAnim = 0;
    }
    let gridX = Math.round((this.pixelX - CELL_SIZE / 2) / CELL_SIZE);
    let gridY = Math.round((this.pixelY - CELL_SIZE / 2) / CELL_SIZE);
    if (this.atCenter()) {
      let dirs = [];
      if (!maze.isWall(gridX - 1, gridY) && this.dir !== Direction.RIGHT) dirs.push(Direction.LEFT);
      if (!maze.isWall(gridX + 1, gridY) && this.dir !== Direction.LEFT) dirs.push(Direction.RIGHT);
      if (!maze.isWall(gridX, gridY - 1) && this.dir !== Direction.DOWN) dirs.push(Direction.UP);
      if (!maze.isWall(gridX, gridY + 1) && this.dir !== Direction.UP) dirs.push(Direction.DOWN);
      if (dirs.length > 1) {
        if (!this.vulnerable && Math.random() < 0.7) {
          let tx = player.x, ty = player.y;
          let best = this.dir;
          let bestDist = 9999;
          for (let d of dirs) {
            let nx = gridX, ny = gridY;
            if (d === Direction.LEFT) nx--;
            else if (d === Direction.RIGHT) nx++;
            else if (d === Direction.UP) ny--;
            else if (d === Direction.DOWN) ny++;
            let dist = Math.abs(nx - tx) + Math.abs(ny - ty);
            if (dist < bestDist) {
              bestDist = dist;
              best = d;
            }
          }
          this.dir = best;
        } else {
          this.dir = dirs[Math.floor(Math.random() * dirs.length)];
        }
      }
    }
    const move = this.getMoveDelta(this.dir, GHOST_SPEED);
    this.pixelX += move.dx;
    this.pixelY += move.dy;
    let newGridX = Math.round((this.pixelX - CELL_SIZE / 2) / CELL_SIZE);
    let newGridY = Math.round((this.pixelY - CELL_SIZE / 2) / CELL_SIZE);
    if (maze.isWall(newGridX, newGridY)) {
      this.pixelX -= move.dx;
      this.pixelY -= move.dy;
      this.dir = [Direction.LEFT, Direction.RIGHT, Direction.UP, Direction.DOWN][Math.floor(Math.random() * 4)];
    }
    if (this.pixelX < 0) this.pixelX = GAME_WIDTH - CELL_SIZE / 2;
    if (this.pixelX > GAME_WIDTH) this.pixelX = CELL_SIZE / 2;
    this.x = Math.round((this.pixelX - CELL_SIZE / 2) / CELL_SIZE);
    this.y = Math.round((this.pixelY - CELL_SIZE / 2) / CELL_SIZE);
    this.draw(globalVul, vulTimeLeft);
  }
  atCenter() {
    let cx = ((this.pixelX - CELL_SIZE / 2) % CELL_SIZE);
    let cy = ((this.pixelY - CELL_SIZE / 2) % CELL_SIZE);
    return Math.abs(cx) < 1 && Math.abs(cy) < 1;
  }
  getMoveDelta(dir, speed) {
    if (dir === Direction.LEFT) return {dx: -speed, dy: 0};
    if (dir === Direction.RIGHT) return {dx: speed, dy: 0};
    if (dir === Direction.UP) return {dx: 0, dy: -speed};
    if (dir === Direction.DOWN) return {dx: 0, dy: speed};
    return {dx: 0, dy: 0};
  }
  draw(globalVul, vulTimeLeft) {
    this.gfx.clear();
    let color = this.color;
    let drawVul = this.vulnerable;
    if (drawVul && vulTimeLeft < 2000) {
      if (!this.flashOn) {
        color = 0xffffff;
      } else {
        color = 0x2b59f3;
      }
      this.flashOn = !this.flashOn;
    } else if (drawVul) {
      color = 0x2b59f3;
    }
    this.gfx.beginFill(color);
    this.gfx.drawRect(this.pixelX - GHOST_SIZE / 2, this.pixelY - GHOST_SIZE / 2, GHOST_SIZE, GHOST_SIZE - 4);
    this.gfx.endFill();
    let legY = this.pixelY + GHOST_SIZE / 2 - 4;
    if (this.legFrame) {
      this.gfx.beginFill(color);
      this.gfx.drawPolygon([
        this.pixelX - GHOST_SIZE / 2, legY,
        this.pixelX - GHOST_SIZE / 4, legY + 4,
        this.pixelX, legY,
        this.pixelX + GHOST_SIZE / 4, legY + 4,
        this.pixelX + GHOST_SIZE / 2, legY
      ]);
      this.gfx.endFill();
    } else {
      this.gfx.beginFill(color);
      this.gfx.drawPolygon([
        this.pixelX - GHOST_SIZE / 2, legY + 4,
        this.pixelX - GHOST_SIZE / 4, legY,
        this.pixelX, legY + 4,
        this.pixelX + GHOST_SIZE / 4, legY,
        this.pixelX + GHOST_SIZE / 2, legY + 4
      ]);
      this.gfx.endFill();
    }
    this.gfx.beginFill(0xffffff);
    this.gfx.drawCircle(this.pixelX - 4, this.pixelY - 4, 3);
    this.gfx.drawCircle(this.pixelX + 4, this.pixelY - 4, 3);
    this.gfx.endFill();
    this.gfx.beginFill(0x222222);
    this.gfx.drawCircle(this.pixelX - 4, this.pixelY - 4, 1.5);
    this.gfx.drawCircle(this.pixelX + 4, this.pixelY - 4, 1.5);
    this.gfx.endFill();
    if (drawVul) {
      this.gfx.beginFill(0xffffff);
      this.gfx.drawRect(this.pixelX - 2, this.pixelY + 2, 4, 2);
      this.gfx.endFill();
    }
  }
}
class UI {
  constructor(layer) {
    this.layer = layer;
    this.scoreText = new PIXI.Text('SCORE: 0', {fontFamily: 'monospace', fontSize: 18, fill: 0xffffff});
    this.scoreText.x = 10;
    this.scoreText.y = 8;
    this.livesText = new PIXI.Text('LIVES: 3', {fontFamily: 'monospace', fontSize: 18, fill: 0xffffff});
    this.livesText.x = GAME_WIDTH - 120;
    this.livesText.y = 8;
    this.layer.addChild(this.scoreText);
    this.layer.addChild(this.livesText);
    this.centerText = new PIXI.Text('', {fontFamily: 'monospace', fontSize: 32, fill: 0xffff00, align: 'center'});
    this.centerText.anchor.set(0.5);
    this.centerText.x = GAME_WIDTH / 2;
    this.centerText.y = GAME_HEIGHT / 2 + 30;
    this.layer.addChild(this.centerText);
  }
  updateScore(score) {
    this.scoreText.text = 'SCORE: ' + score;
  }
  updateLives(lives) {
    this.livesText.text = 'LIVES: ' + lives;
  }
  showCenter(text, color) {
    this.centerText.text = text;
    this.centerText.style.fill = color;
    this.centerText.visible = true;
  }
  hideCenter() {
    this.centerText.visible = false;
  }
}
class Game {
  constructor() {
    this.app = new PIXI.Application({
      width: GAME_WIDTH,
      height: GAME_HEIGHT,
      backgroundColor: 0x111111,
      antialias: true,
      resolution: 1
    });
    document.body.appendChild(this.app.view);
    this.root = new PIXI.Container();
    this.mazeLayer = new PIXI.Container();
    this.dotLayer = new PIXI.Container();
    this.pelletLayer = new PIXI.Container();
    this.ghostLayer = new PIXI.Container();
    this.playerLayer = new PIXI.Container();
    this.uiLayer = new PIXI.Container();
    this.root.addChild(this.mazeLayer);
    this.root.addChild(this.dotLayer);
    this.root.addChild(this.pelletLayer);
    this.root.addChild(this.ghostLayer);
    this.root.addChild(this.playerLayer);
    this.root.addChild(this.uiLayer);
    this.root.y = MAZE_OFFSET_Y;
    this.uiLayer.y = 0;
    this.app.stage.addChild(this.root);
    this.state = GameState.LOADING;
    this.score = 0;
    this.lives = PLAYER_LIVES;
    this.globalVul = false;
    this.vulTimeLeft = 0;
    this.readyTimer = 0;
    this.ghostRespawnTimers = [0,0,0,0];
    this.lastTime = performance.now();
    this.maze = new Maze(this.mazeLayer);
    this.dots = new DotManager(this.dotLayer);
    this.pellets = new PelletManager(this.pelletLayer);
    this.player = new Player(this.playerLayer);
    this.ghosts = [
      new Ghost(this.ghostLayer, GHOST_COLORS[0], 10, 8),
      new Ghost(this.ghostLayer, GHOST_COLORS[1], 9, 9),
      new Ghost(this.ghostLayer, GHOST_COLORS[2], 10, 9),
      new Ghost(this.ghostLayer, GHOST_COLORS[3], 11, 9)
    ];
    this.ui = new UI(this.uiLayer);
    this.inputDir = Direction.NONE;
    this.keyReady = false;
    window.addEventListener('keydown', e => {
      if (this.state === GameState.LOADING) {
        this.startGame();
      } else if (this.state === GameState.READY) {
        this.startPlay();
      } else if (this.state === GameState.PLAY) {
        if (e.key === 'ArrowLeft') this.player.setDirection(Direction.LEFT);
        else if (e.key === 'ArrowRight') this.player.setDirection(Direction.RIGHT);
        else if (e.key === 'ArrowUp') this.player.setDirection(Direction.UP);
        else if (e.key === 'ArrowDown') this.player.setDirection(Direction.DOWN);
      } else if (this.state === GameState.GAME_OVER || this.state === GameState.WIN) {
        if (e.key === ' ' || e.key === 'r' || e.key === 'R') {
          this.restart();
        }
      }
    });
    this.showLoading();
    this.app.ticker.add((delta) => this.update(delta));
  }
  showLoading() {
    this.ui.showCenter('PRESS ANY KEY TO START', 0xffffff);
  }
  startGame() {
    this.state = GameState.READY;
    this.score = 0;
    this.lives = PLAYER_LIVES;
    this.globalVul = false;
    this.vulTimeLeft = 0;
    this.dots.reset();
    this.pellets.reset();
    this.player.reset();
    for (let g of this.ghosts) g.reset();
    this.ui.updateScore(this.score);
    this.ui.updateLives(this.lives);
    this.ui.showCenter('READY!', 0xffff00);
    this.readyTimer = 0;
  }
  startPlay() {
    this.state = GameState.PLAY;
    this.ui.hideCenter();
  }
  restart() {
    this.state = GameState.READY;
    this.score = 0;
    this.lives = PLAYER_LIVES;
    this.globalVul = false;
    this.vulTimeLeft = 0;
    this.dots.reset();
    this.pellets.reset();
    this.player.reset();
    for (let g of this.ghosts) g.reset();
    this.ui.updateScore(this.score);
    this.ui.updateLives(this.lives);
    this.ui.showCenter('READY!', 0xffff00);
    this.readyTimer = 0;
  }
  update(delta) {
    const now = performance.now();
    const dt = now - (this.lastTime || now);
    this.lastTime = now;
    if (this.state === GameState.LOADING) return;
    if (this.state === GameState.READY) {
      this.readyTimer += dt;
      if (this.readyTimer > 2000) {
        this.startPlay();
      }
      return;
    }
    if (this.state === GameState.PLAY) {
      this.player.update(this.maze, dt);
      this.pellets.update(dt);
      for (let g of this.ghosts) g.update(this.maze, this.player, dt, this.globalVul, this.vulTimeLeft);
      this.checkDotCollisions();
      this.checkPelletCollisions();
      this.checkGhostCollisions();
      this.ui.updateScore(this.score);
      this.ui.updateLives(this.lives);
      if (this.dots.remaining() === 0) {
        this.state = GameState.WIN;
        this.ui.showCenter('YOU WIN!\nSCORE: ' + this.score + '\nPRESS SPACE TO RESTART', 0x00ff00);
      }
      if (this.globalVul) {
        this.vulTimeLeft -= dt;
        if (this.vulTimeLeft <= 0) {
          this.globalVul = false;
        }
      }
    }
  }
  checkDotCollisions() {
    if (this.player.alive) {
      if (this.dots.eatDot(this.player.x, this.player.y)) {
        this.score += 10;
      }
    }
  }
  checkPelletCollisions() {
    if (this.player.alive) {
      if (this.pellets.eatPellet(this.player.x, this.player.y)) {
        this.globalVul = true;
        this.vulTimeLeft = VULNERABLE_DURATION;
        for (let g of this.ghosts) g.setVulnerable(true);
      }
    }
  }
  checkGhostCollisions() {
    if (!this.player.alive) return;
    for (let g of this.ghosts) {
      if (!g.visible) continue;
      let dx = g.pixelX - this.player.pixelX;
      let dy = g.pixelY - this.player.pixelY;
      let dist = Math.sqrt(dx * dx + dy * dy);
      if (dist < PLAYER_RADIUS + GHOST_SIZE / 2 - 2) {
        if (this.globalVul && g.vulnerable && !g.respawning) {
          g.startRespawn();
          this.score += 200;
        } else if (!g.vulnerable && !g.respawning) {
          this.lives--;
          this.player.alive = false;
          this.ui.updateLives(this.lives);
          if (this.lives <= 0) {
            this.state = GameState.GAME_OVER;
            this.ui.showCenter('GAME OVER\nSCORE: ' + this.score + '\nPRESS SPACE TO RESTART', 0xff4444);
          } else {
            setTimeout(() => {
              this.player.reset();
              for (let g of this.ghosts) g.reset();
              this.player.alive = true;
              this.state = GameState.READY;
              this.ui.showCenter('READY!', 0xffff00);
              this.readyTimer = 0;
            }, 1200);
          }
          break;
        }
      }
    }
  }
}
new Game();
