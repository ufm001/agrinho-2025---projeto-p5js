// CONFIGURAÇÕES PERSONALIZÁVEIS
let player;
let trees = [];
let treesPlanted = 0;
let plantGoal = 20;       // META MAIOR
let timeLimit = 60 * 60;  // 60 segundos, 60fps
let timer = 0;

let gameState = "menu"; // "menu", "jogando", "fim"

// Cores do cenário
const skyColor = [135, 206, 235];
const grassColor = [60, 180, 60];
const sunColor = [255, 223, 0];

// Variáveis para nuvens
let clouds = [];

// Botão para iniciar jogo
let startButton;

function setup() {
  createCanvas(700, 450);
  frameRate(60);

  initGame();

  // Criar botão no centro (invisível inicialmente)
  startButton = createButton('Jogar');
  startButton.style('font-size', '24px');
  startButton.style('padding', '12px 24px');
  startButton.position(width / 2 - 60, height / 2 + 40);
  startButton.mousePressed(() => {
    if (gameState === "menu") {
      startGame();
    } else if (gameState === "fim") {
      gameState = "menu";
      startButton.html('Jogar');
      startButton.position(width / 2 - 60, height / 2 + 40);
    }
  });
}

function initGame() {
  player = {
    x: width / 2,
    y: height * 0.75,
    size: 40,
    speed: 4,
    step: 0,
    stepDir: 1,
  };
  trees = [];
  treesPlanted = 0;
  timer = 0;

  clouds = [];
  for (let i = 0; i < 5; i++) {
    clouds.push({
      x: random(width),
      y: random(30, 100),
      size: random(60, 120),
      speed: random(0.3, 0.8),
    });
  }
}

function startGame() {
  gameState = "jogando";
  initGame();
  loop();
  startButton.hide();
}

function draw() {
  if (gameState === "menu") {
    drawMenu();
  } else if (gameState === "jogando") {
    drawGame();
  } else if (gameState === "fim") {
    drawEndScreen();
  }
}

function drawMenu() {
  background(...skyColor);
  fill(0, 100, 0);
  textAlign(CENTER, CENTER);
  textSize(48);
  text("Plantando Árvores", width / 2, height / 3);
  textSize(18);
  text("Use WASD para andar e espaço para plantar árvores.", width / 2, height / 3 + 50);
  text("Plante 20 árvores em 60 segundos para vencer!", width / 2, height / 3 + 80);
  startButton.show();
}

function drawGame() {
  drawSky();
  drawSun();
  drawClouds();
  drawGrass();

  handlePlayerMovement();
  drawPlayer();

  drawTrees();

  drawUI();

  timer++;
  if (timer > timeLimit || treesPlanted >= plantGoal) {
    gameState = "fim";
    startButton.html('Jogar Novamente');
    startButton.position(width / 2 - 90, height / 2 + 80);
    startButton.show();
    noLoop();
  }
}

function drawEndScreen() {
  drawSky();
  drawSun();
  drawClouds();
  drawGrass();
  drawTrees();
  drawPlayer();

  fill(0, 180);
  rect(0, 0, width, height);

  textAlign(CENTER, CENTER);
  textSize(36);
  fill(treesPlanted >= plantGoal ? [0, 180, 0] : [180, 0, 0]);
  if (treesPlanted >= plantGoal) {
    text("🎉 Parabéns! Você venceu! 🎉", width / 2, height / 2 - 40);
    textSize(24);
    fill(255);
    text(`Você plantou ${treesPlanted} árvores!`, width / 2, height / 2 + 10);
  } else {
    text("😞 Você perdeu!", width / 2, height / 2 - 40);
    textSize(24);
    fill(255);
    text(`Plante pelo menos ${plantGoal} árvores para vencer.`, width / 2, height / 2 + 10);
    text(`Você plantou ${treesPlanted}. Tente novamente!`, width / 2, height / 2 + 40);
  }
}

function drawSky() {
  background(...skyColor);
}

function drawSun() {
  noStroke();
  fill(...sunColor);
  ellipse(width - 100, 100, 80, 80);

  stroke(...sunColor);
  strokeWeight(3);
  for (let angle = 0; angle < TWO_PI; angle += PI / 6) {
    let x1 = width - 100 + cos(angle) * 50;
    let y1 = 100 + sin(angle) * 50;
    let x2 = width - 100 + cos(angle) * 70;
    let y2 = 100 + sin(angle) * 70;
    line(x1, y1, x2, y2);
  }
  noStroke();
}

function drawClouds() {
  fill(255, 240);
  noStroke();
  for (let c of clouds) {
    ellipse(c.x, c.y, c.size * 0.6, c.size * 0.6);
    ellipse(c.x + c.size * 0.3, c.y + 10, c.size * 0.7, c.size * 0.7);
    ellipse(c.x - c.size * 0.3, c.y + 5, c.size * 0.5, c.size * 0.5);

    c.x += c.speed;
    if (c.x - c.size > width) {
      c.x = -c.size;
      c.y = random(30, 100);
    }
  }
}

function drawGrass() {
  noStroke();
  fill(...grassColor);
  rect(0, height * 0.7, width, height * 0.3);

  stroke(40, 150, 40);
  strokeWeight(2);
  for (let x = 0; x < width; x += 10) {
    let h = map(sin(frameCount * 0.1 + x * 0.2), -1, 1, 10, 20);
    line(x, height * 0.7 + 30, x, height * 0.7 + 30 - h);
  }
  noStroke();
}

function handlePlayerMovement() {
  let moved = false;
  if (keyIsDown(87)) { // W
    player.y -= player.speed;
    moved = true;
  }
  if (keyIsDown(83)) { // S
    player.y += player.speed;
    moved = true;
  }
  if (keyIsDown(65)) { // A
    player.x -= player.speed;
    moved = true;
  }
  if (keyIsDown(68)) { // D
    player.x += player.speed;
    moved = true;
  }

  player.x = constrain(player.x, player.size / 2, width - player.size / 2);
  player.y = constrain(player.y, height * 0.7 + player.size / 2, height - player.size / 2);

  if (moved) {
    player.step += player.stepDir * 0.2;
    if (player.step > 1 || player.step < -1) player.stepDir *= -1;
  } else {
    player.step = 0;
  }
}

function drawPlayer() {
  push();
  translate(player.x, player.y);

  fill(220, 150, 100);
  rectMode(CENTER);
  rect(0, 5, player.size * 0.6, player.size);

  fill(255, 220, 180);
  ellipse(0, -player.size * 0.3, player.size * 0.6, player.size * 0.6);

  fill(0);
  ellipse(-player.size * 0.15, -player.size * 0.35, 5, 8);
  ellipse(player.size * 0.15, -player.size * 0.35, 5, 8);

  stroke(220, 150, 100);
  strokeWeight(8);
  let armSwing = sin(player.step * PI) * 10;
  line(-player.size * 0.3, 0, -player.size * 0.6, armSwing);
  line(player.size * 0.3, 0, player.size * 0.6, -armSwing);

  stroke(80, 50, 20);
  strokeWeight(10);
  let legSwing = sin(player.step * PI) * 15;
  line(-player.size * 0.15, player.size * 0.5, -player.size * 0.15 + legSwing, player.size);
  line(player.size * 0.15, player.size * 0.5, player.size * 0.15 - legSwing, player.size);

  pop();
}

function drawTrees() {
  for (let t of trees) {
    drawTree(t.x, t.y, t.size, t.offset);
  }
}

function drawTree(x, y, size, offset) {
  push();
  translate(x, y);

  fill(101, 67, 33);
  rectMode(CENTER);
  rect(0, 0, size * 0.25, size * 0.6);

  stroke(80, 50, 20);
  strokeWeight(2);
  for (let i = -size * 0.12; i <= size * 0.12; i += 5) {
    line(i, -size * 0.3, i, size * 0.3);
  }
  noStroke();

  fill(50, 160, 50);
  let sway = sin(frameCount * 0.05 + offset) * 5;
  ellipse(sway, -size * 0.7, size * 0.9, size * 0.9);
  ellipse(sway + size * 0.3, -size * 0.75, size * 0.7, size * 0.7);
  ellipse(sway - size * 0.3, -size * 0.75, size * 0.7, size * 0.7);

  pop();
}

function drawUI() {
  fill(0);
  textSize(20);
  textAlign(LEFT, TOP);
  text(`Árvores plantadas: ${treesPlanted} / ${plantGoal}`, 10, 10);

  let timeLeft = max(0, floor((timeLimit - timer) / 60));
  text(`Tempo restante: ${timeLeft}s`, 10, 35);

  fill(80, 50, 20);
  textSize(12);
  text("Use WASD para se mover, espaço para plantar árvore.", 10, height - 30);
}

function keyPressed() {
  if (gameState === "jogando" && key === ' ') {
    let canPlant = true;
    let plantDist = 40;
    for (let t of trees) {
      if (dist(player.x, player.y, t.x, t.y) < plantDist) {
        canPlant = false;
        break;
      }
    }
    if (canPlant) {
      trees.push({
        x: player.x,
        y: player.y,
        size: random(35, 50),
        offset: random(1000)
      });
      treesPlanted++;
    }
  }
}
