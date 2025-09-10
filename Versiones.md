
## Versión 1 — Detección básica (colores sobre movimiento)
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
## Versión 2 — Fondo negro + estelas
```javascript
let video, prevFrame, trails;

function setup(){
  createCanvas(640,480);
  video = createCapture(VIDEO);
  video.size(width,height);
  video.hide();
  prevFrame = createImage(width,height);
  trails = createGraphics(width,height);
  trails.background(0);
}

function draw(){
  background(0);
  video.loadPixels();
  prevFrame.loadPixels();
  trails.fill(0,0,0,40);
  trails.rect(0,0,width,height);

  for(let y=0; y<height; y+=2){
    for(let x=0; x<width; x+=2){
      let i = 4*(y*width+x);
      let r = video.pixels[i], g = video.pixels[i+1], b = video.pixels[i+2];
      let rP = prevFrame.pixels[i], gP = prevFrame.pixels[i+1], bP = prevFrame.pixels[i+2];
      let diff = (abs(r-rP)+abs(g-gP)+abs(b-bP))/3;

      if(diff > 25){
        trails.fill(random(255),random(255),random(255),200);
        trails.ellipse(x,y,2,2);
      }
    }
  }
  image(trails,0,0);
  prevFrame.copy(video,0,0,width,height,0,0,width,height);
}
```
## Versión 3 — Modos automáticos (Holograma ↔ Pincel de Luz)
```javascript
let video, prevFrame, trails;
let energyEMA = 0, mode = 1;
const E_UP = 0.05, E_DOWN = 0.02;

function setup(){
  createCanvas(640,480);
  video = createCapture(VIDEO);
  video.size(width,height);
  video.hide();
  prevFrame = createImage(width,height);
  trails = createGraphics(width,height);
  trails.background(0);
}

function draw(){
  background(0);
  video.loadPixels();
  prevFrame.loadPixels();
  trails.fill(0,0,0,40);
  trails.rect(0,0,width,height);

  let activeFD = 0, blocks=0;

  for(let y=0; y<height; y+=2){
    for(let x=0; x<width; x+=2){
      let i = 4*(y*width+x);
      let r = video.pixels[i], g = video.pixels[i+1], b = video.pixels[i+2];
      let rP = prevFrame.pixels[i], gP = prevFrame.pixels[i+1], bP = prevFrame.pixels[i+2];
      let dFD = (abs(r-rP)+abs(g-gP)+abs(b-bP))/3;
      blocks++;
      if(dFD > 25){
        activeFD++;
        if(mode === 1){
          trails.fill(0,200,255,170);
          trails.rect(x,y,2,2);
          trails.fill(0,200,255,110);
          trails.ellipse(x,y,2,2);
        } else {
          let sz = 2 + noise(x*0.02,y*0.02)*4;
          trails.fill(255,100,200,200);
          trails.ellipse(x,y,sz,sz);
        }
      }
    }
  }
  image(trails,0,0);
  prevFrame.copy(video,0,0,width,height,0,0,width,height);

  // energía promedio
  let energy = activeFD/blocks;
  energyEMA = lerp(energyEMA,energy,0.2);
  if(energyEMA > E_UP) mode = 2;
  if(energyEMA < E_DOWN) mode = 1;
}
```
## Versión 4 — Visual neón multicapa
```javascript
let video, prevFrame, trails;
let hueOffset=0;

function setup(){
  createCanvas(640,480);
  colorMode(RGB);
  video = createCapture(VIDEO);
  video.size(width,height);
  video.hide();
  prevFrame = createImage(width,height);
  trails = createGraphics(width,height);
  trails.background(0);
}

function hsvToRgb(h, s, v){
  let r,g,b;
  let i=Math.floor(h*6);
  let f=h*6-i;
  let p=v*(1-s);
  let q=v*(1-f*s);
  let t=v*(1-(1-f)*s);
  switch(i%6){
    case 0: r=v; g=t; b=p; break;
    case 1: r=q; g=v; b=p; break;
    case 2: r=p; g=v; b=t; break;
    case 3: r=p; g=q; b=v; break;
    case 4: r=t; g=p; b=v; break;
    case 5: r=v; g=p; b=q; break;
  }
  return [r*255,g*255,b*255];
}

function draw(){
  background(0);
  hueOffset=(hueOffset+0.5)%360;
  video.loadPixels();
  prevFrame.loadPixels();
  trails.fill(0,0,0,40);
  trails.rect(0,0,width,height);

  for(let y=0;y<height;y+=2){
    for(let x=0;x<width;x+=2){
      let i=4*(y*width+x);
      let r=video.pixels[i],g=video.pixels[i+1],b=video.pixels[i+2];
      let rP=prevFrame.pixels[i],gP=prevFrame.pixels[i+1],bP=prevFrame.pixels[i+2];
      let diff=(abs(r-rP)+abs(g-gP)+abs(b-bP))/3;
      if(diff>25){
        let hue=(hueOffset+x*0.1+y*0.1)%360;
        let col=hsvToRgb(hue/360,0.9,0.9);
        trails.fill(col[0],col[1],col[2],80);
        trails.ellipse(x,y,4,4);
        trails.fill(col[0],col[1],col[2],180);
        trails.ellipse(x,y,2,2);
      }
    }
  }
  image(trails,0,0);
  prevFrame.copy(video,0,0,width,height,0,0,width,height);
}
```
