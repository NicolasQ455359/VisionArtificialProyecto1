### Codigos durante el semestre

### Codigos

## 1. Detección básica con Background Subtraction y Frame Difference

Objetivo: probar los métodos de detección de movimiento y visualizar diferencias de píxeles en colores llamativos.

Se implementó un fondo de referencia con aprendizaje progresivo.

Se combinaron diferencias de fondo (BG) y de frames consecutivos (FD).

Resultado: se detectaba movimiento, pero aún aparecían “pantallas blancas” por cambios de luz.
Próximo paso: refinar umbrales adaptativos y evitar saturación visual.

## 2. Fondo negro con siluetas y estelas

Objetivo: mejorar la estética y centrar la atención en el movimiento.

Se eliminó el video en bruto, dejando solo trazos sobre fondo negro.

Se añadieron estelas con transparencia para simular persistencia del movimiento.

Resultado: el usuario veía su silueta formada por trazos dinámicos.
Próximo paso: hacer el cambio de modo automático (Holograma ↔ Pincel de Luz).

## 3. Cambio de modos según energía del movimiento

Objetivo: que el sistema decidiera automáticamente el modo visual.

Se calculó la energía del gesto (proporción de píxeles activos en FD).

Se establecieron dos umbrales (E_UP, E_DOWN) para evitar cambios bruscos.

Resultado: movimientos suaves generaban Holograma y movimientos intensos → Pincel de Luz.
Próximo paso: enriquecer los colores y dar un estilo más atractivo (neón).

## 4. Visual neón multicapa

Objetivo: darle un carácter artístico más llamativo.

Se pasó de colores planos a un sistema HSV→RGB con variaciones por posición, tiempo y ruido.

Se introdujeron bloques y círculos con alfas distintos para simular glow.

Resultado: visual más vivo y atractivo, pero con exceso de “puntitos”.
Próximo paso: limpiar el render y añadir partículas controladas.

## 5. Partículas controladas y refinamiento visual

Objetivo: aportar dinamismo sin saturar la pantalla.

Se añadió un sistema de partículas con límites duros (MAX_PARTICLES, MAX_SPAWN_PER_FRAME).

Cada partícula tenía núcleo + halo, tamaño variable y vida corta.

Resultado: se logró un equilibrio entre estelas, trazos y chispas.

# 🗂️ Fase de Documentación del Proyecto *Espejo de Energía*

---

## 1. Versión inicial: Detección básica de movimiento  
**Qué se hizo:**  
- Primer test con `background subtraction` (BG) y `frame difference` (FD).  
- Se pintaban píxeles en colores llamativos cuando había movimiento.  
- Problema: se saturaba fácilmente con luz → “pantalla blanca”.

```javascript
let video, prevFrame;

function setup(){
  createCanvas(640,480);
  video = createCapture(VIDEO);
  video.size(width,height);
  video.hide();
  prevFrame = createImage(width,height);
}

function draw(){
  background(0);
  video.loadPixels();
  prevFrame.loadPixels();
  loadPixels();
  
  for(let i=0; i<video.pixels.length; i+=4){
    let r = video.pixels[i];
    let g = video.pixels[i+1];
    let b = video.pixels[i+2];
    let rP = prevFrame.pixels[i];
    let gP = prevFrame.pixels[i+1];
    let bP = prevFrame.pixels[i+2];
    let diff = (abs(r-rP)+abs(g-gP)+abs(b-bP))/3;

    if(diff > 30){
      pixels[i]   = (r+frameCount*2)%255;
      pixels[i+1] = (g+frameCount*3)%255;
      pixels[i+2] = (b+frameCount*4)%255;
      pixels[i+3] = 255;
    }
  }
  updatePixels();
  prevFrame.copy(video,0,0,width,height,0,0,width,height);
}
```
# 2. Fondo negro + estelas
Qué se hizo:

Se eliminó el video en bruto → solo movimiento sobre negro.

Se agregó buffer trails con transparencia → persistencia visual.

```javascript
trails = createGraphics(width,height);
trails.background(0);

function draw(){
  background(0);
  trails.fill(0,0,0,40); // fading
  trails.rect(0,0,width,height);

  if(diff > fdThr && dBG > bgThr){
    trails.fill(random(255),random(255),random(255),200);
    trails.ellipse(x,y,step,step);
  }
  image(trails,0,0);
}
```
## 3. Cambio de modos (Holograma ↔ Pincel de Luz)
Qué se hizo:

Se calculó la energía del movimiento (energyEMA).

Dos umbrales (E_UP, E_DOWN) controlan el cambio de modo.

```javascript
if(energyEMA > E_UP){ mode = 2; } 
if(energyEMA < E_DOWN){ mode = 1; }

if(mode === 1){
  trails.fill(col[0],col[1],col[2],170);
  trails.rect(x,y,step,step);
} else {
  let sz = step*0.8 + noise(x*0.02,y*0.02)*step*2;
  trails.fill(col[0],col[1],col[2],200);
  trails.ellipse(x,y,sz,sz);
}
```
Modo 1 (Holograma): bloques + highlights → silueta continua.

Modo 2 (Pincel de Luz): círculos orgánicos con trazos dinámicos.

## 4. Visual Neón Multicapa
Qué se hizo:

Conversión HSV→RGB para colores vivos.

Se añadieron halos y núcleos para efecto glow.

```javascript
const hue = (hueOffset + x*0.2 + y*0.15 + m*50) % 360;
const col = hsvToRgb(hue/360,0.9,0.9);

// Glow multicapa
trails.fill(col[0],col[1],col[2],80);
trails.ellipse(cx,cy,step*2,step*2);
trails.fill(col[0],col[1],col[2],180);
trails.ellipse(cx,cy,step,step);
```
## 5. Partículas controladas
Qué se hizo:

Sistema de partículas con vida limitada y emisión moderada.

Límite de cantidad (MAX_PARTICLES) y spawn por frame.

```javascript
if(particles.length < MAX_PARTICLES && random() < 0.1){
  particles.push({
    x:cx, y:cy,
    vx:random(-1,1), vy:random(-1,1),
    life:30, col:col
  });
}

for(let p of particles){
  trails.fill(p.col[0],p.col[1],p.col[2],100);
  trails.ellipse(p.x,p.y,p.life/2);
}
```




















