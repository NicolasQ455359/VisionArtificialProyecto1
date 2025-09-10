# VisionArtificialProyecto1

## Entrega #1 [Espejo de Energía – Documento en Gamma](https://gamma.app/docs/Espejo-de-Energia-4umzop6adaul19w)

## Entrega #2 [Espejo de Energía – Documento en Gamma (modo doc)](https://gamma.app/docs/Espejo-de-Energia-9o6nm824pz1zrk9?mode=doc)

### Espeko de Energía: 
El proyecto Espejo de Energía se fundamenta en el uso de técnicas de visión por computadora para captar y reinterpretar los movimientos del cuerpo humano en tiempo real. Dos enfoques principales sostienen el sistema:

- Background Subtraction (sustracción de fondo): diferencia entre el estado actual de la cámara y una imagen de referencia, permitiendo distinguir al usuario del entorno estático.

- Frame Difference (diferencia entre cuadros consecutivos): analiza cambios de píxeles entre frames para detectar movimiento continuo y gestos.

La combinación de ambos métodos proporciona una detección robusta, reduciendo errores ocasionados por variaciones de luz o ruido ambiental.

### Arte Digital Interactivo

El proyecto se inscribe dentro del campo del arte digital y new media, en el cual el espectador deja de ser un observador pasivo para convertirse en agente activo que genera y transforma la obra. Aquí, el cuerpo se convierte en la herramienta principal de interacción, y la cámara en el medio de traducción hacia lo digital.

La propuesta se inspira en referencias de instalaciones audiovisuales que buscan explorar la relación entre tecnología, percepción y movimiento, transformando los gestos en experiencias estéticas inmersivas.

### Estética Visual: Neón y Estelas

El estilo visual del Espejo de Energía se apoya en una paleta de colores neón dinámicos, renderizados sobre fondo negro, lo que intensifica el contraste y resalta la energía del movimiento.

Los movimientos suaves generan un modo Holograma, con siluetas delicadas y continuas.

Los movimientos rápidos activan el modo Pincel de Luz, produciendo trazos brillantes, partículas y estelas que permanecen unos instantes antes de desvanecerse.

Este enfoque visual busca comunicar la idea de que cada gesto deja huella, transformando lo efímero del movimiento corporal en un trazo lumínico que persiste momentáneamente en el espacio digital.

### Interacción y Experiencia del Usuario

El usuario se reconoce a sí mismo en el espejo digital, pero en lugar de ver una réplica fiel, observa una reinvención energética de su cuerpo. La experiencia resulta atractiva, lúdica y contemplativa al mismo tiempo:

Fondo negro que refuerza el protagonismo del movimiento.

Estelas temporales que representan la memoria del gesto.

Partículas luminosas que aportan dinamismo sin saturar la escena.

El Espejo de Energía convierte la interacción en un diálogo entre cuerpo y máquina, donde cada acción física se transforma en un lenguaje visual.

### Marco Teórico de Referencia

Interactividad en el arte digital: Lev Manovich y la noción de nuevas interfaces culturales.

Estética del movimiento: influencia de instalaciones audiovisuales que usan captura de movimiento para explorar la corporalidad.

Arte generativo: procesos algorítmicos que producen resultados visuales dinámicos en tiempo real.

## Codigo final: 
```javascript
let video, prevFrame, bgFrame, trails;
const W = 640, H = 480;
const step = 2;              // muestreo (2 = 1/4 de píxeles procesados)

let learnRate = 0.006;       // fondo aprende moderado
let baseBG = 28;             // umbral base para BG
let baseFD = 10;             // umbral base para FD

let energyEMA = 0;
let mode = 1;                // 1 Holograma, 2 Pincel
const E_UP = 0.05, E_DOWN = 0.02;
let hiHold = 0, loHold = 0;

let fadeStrength = 0.90;     // persistencia de estelas
const FADE_MIN = 0.86, FADE_MAX = 0.96;

let hueOffset = 0, t = 0;
let warmup = 45;             // frames de calentamiento para estabilizar fondo

function setup(){
  createCanvas(W, H);
  pixelDensity(1);
  background(0);

  video = createCapture(VIDEO);
  video.size(W, H);
  video.hide();

  prevFrame = createImage(W, H);
  bgFrame   = createImage(W, H);

  trails = createGraphics(W, H);
  trails.pixelDensity(1);
  trails.background(0);
  trails.noStroke();

  // fondo inicial
  video.loadPixels();
  bgFrame.loadPixels();
  for (let i=0; i<video.pixels.length; i++){
    bgFrame.pixels[i] = video.pixels[i];
  }
  bgFrame.updatePixels();
}

function draw(){
  background(0);
  t += 0.016;
  hueOffset = (hueOffset + 0.12) % 360;

  video.loadPixels();
  prevFrame.loadPixels();
  bgFrame.loadPixels();

  // Medición global (promedios)
  let luminSum = 0, noiseFDSum = 0, blocks = 0;

  for (let y=0; y<H; y+=step){
    for (let x=0; x<W; x+=step){
      const i = 4*(y*W + x);
      const r=video.pixels[i], g=video.pixels[i+1], b=video.pixels[i+2];
      const rP=prevFrame.pixels[i], gP=prevFrame.pixels[i+1], bP=prevFrame.pixels[i+2];
      luminSum += (r+g+b)/3;
      noiseFDSum += (abs(r-rP)+abs(g-gP)+abs(b-bP))/3;
      blocks++;
    }
  }
  const luminAvg = luminSum/blocks;
  const noiseFD  = noiseFDSum/blocks;

  // Umbrales adaptativos suaves
  const bgThr = baseBG + map(luminAvg, 0,255, -6, 6);          // BG
  const fdThr = baseFD + map(constrain(noiseFD,0,25),0,25, 0,6); // FD

  // Fading de estelas sobre negro
  trails.push();
  trails.noStroke();
  trails.fill(0,0,0, map(1 - fadeStrength, 0, 1, 0, 255));
  trails.rect(0,0,W,H);
  trails.pop();

  // Dibujo (ADD) sólo donde hay movimiento real
  trails.push();
  trails.blendMode(ADD);
  trails.noStroke();

  let activeFD = 0;

  for (let y=0; y<H; y+=step){
    for (let x=0; x<W; x+=step){
      const i = 4*(y*W + x);

      const r = video.pixels[i],   g = video.pixels[i+1], b = video.pixels[i+2];
      const rB= bgFrame.pixels[i], gB= bgFrame.pixels[i+1], bB= bgFrame.pixels[i+2];
      const rP= prevFrame.pixels[i],gP= prevFrame.pixels[i+1], bP= prevFrame.pixels[i+2];

      const dBG = (abs(r-rB)+abs(g-gB)+abs(b-bB))/3;
      const dFD = (abs(r-rP)+abs(g-gP)+abs(b-bP))/3;

      // Actualiza fondo (running average)
      bgFrame.pixels[i]   = (1-learnRate)*rB + learnRate*r;
      bgFrame.pixels[i+1] = (1-learnRate)*gB + learnRate*g;
      bgFrame.pixels[i+2] = (1-learnRate)*bB + learnRate*b;
      bgFrame.pixels[i+3] = 255;

      // Energía SOLO con FD (gesto real)
      if (dFD > fdThr) activeFD++;

      // DIBUJO SÓLO si ambos superan su umbral (evita “todo blanco”)
      if (dFD > fdThr && dBG > bgThr && warmup<=0){
        const motion = 0.6*dBG + 0.4*dFD;
        const m = constrain((motion - (bgThr+fdThr)/2) / ( (bgThr+fdThr)/2 + 40 ), 0, 1);

        // Color vibrante controlado (HSV → RGB)
        const hue = (hueOffset + (x+y)*0.03 + m*35) % 360;
        const sat = 0.85;
        const val = 0.80 + 0.15*m; // brillo máx 0.95 para no quemar
        const col = hsvToRgb(hue/360, sat, val); // [0..255]

        const cx = x + step*0.5, cy = y + step*0.5;

        if (mode === 1){
          // HOLOGRAMA: bloque + highlight (silueta continua)
          trails.fill(col[0], col[1], col[2], 170);
          trails.rect(x, y, step, step);
          trails.fill(col[0], col[1], col[2], 110);
          trails.ellipse(cx, cy, step*0.7, step*0.7);
        } else {
          // PINCEL DE LUZ: círculo orgánico (estela)
          const v = noise(x*0.02, y*0.02, t*1.1);
          const sz = step*0.8 + v*step*1.8 + m*step*1.2;
          trails.fill(col[0], col[1], col[2], 200);
          trails.ellipse(cx, cy, sz, sz);
        }
      }
    }
  }
  trails.pop();

  // Actualiza buffers
  bgFrame.updatePixels();
  prevFrame.copy(video, 0, 0, W, H, 0, 0, W, H);

  // Warmup: los primeros frames NO dibujan para estabilizar el fondo
  if (warmup > 0) { warmup--; }

  // Energía y modos
  const energy = constrain(activeFD / blocks, 0, 1);
  energyEMA = lerp(energyEMA, energy, 0.18);
  fadeStrength = constrain(map(energyEMA, 0, 0.10, FADE_MIN, FADE_MAX), FADE_MIN, FADE_MAX);

  if (energyEMA > E_UP) { hiHold++; loHold = max(0, loHold-1); }
  else if (energyEMA < E_DOWN) { loHold++; hiHold = max(0, hiHold-1); }
  else { hiHold = max(0, hiHold-1); loHold = max(0, loHold-1); }
  if (hiHold > 6) { mode = 2; hiHold = 0; }
  if (loHold >10) { mode = 1; loHold = 0; }

  // Composición final (sólo estelas sobre negro)
  image(trails, 0, 0);

  // HUD oscuro (no “lava” la imagen)
  drawHUD();
}

// ---- Utilidades ----
function hsvToRgb(h, s, v){
  let r,g,b;
  const i = Math.floor(h*6);
  const f = h*6 - i;
  const p = v*(1-s);
  const q = v*(1-f*s);
  const t = v*(1-(1-f)*s);
  switch(i % 6){
    case 0: r=v; g=t; b=p; break;
    case 1: r=q; g=v; b=p; break;
    case 2: r=p; g=v; b=t; break;
    case 3: r=p; g=q; b=v; break;
    case 4: r=t; g=p; b=v; break;
    case 5: r=v; g=p; b=q; break;
  }
  return [Math.round(r*255), Math.round(g*255), Math.round(b*255)];
}

function drawHUD(){
  push();
  noStroke();
  fill(0, 170); // caja oscura
  rect(10, 10, 240, 36, 8); // más pequeño, solo para el título
  fill(255);
  textStyle(BOLD);
  text("Espejo de Energía", 18, 32);
  pop();
}
```













