<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Dino contra el tiempo</title>
  <style>
    body {
      margin:0;
      background:#000; /* Fondo negro */
      font-family:sans-serif;
      display:flex;
      justify-content:center;
      align-items:center;
      height:100vh;
    }
    canvas {
      background:#000; /* Canvas también negro */
      border:1px solid #444;
      display:block;
    }
  </style>
</head>
<body>
  <canvas id="game" width="800" height="200"></canvas>
  <script>
  (() => {
    const TIME_LIMIT = 60; // segundos
    let timer = TIME_LIMIT;
    let last = 0;
    let state = 'playing';

    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    const w = canvas.width, h = canvas.height;

    const dino = { x:50, y:h-60, w:40, h:50, vy:0, onGround:true };
    const GRAV = 2000, JUMP = -800;
    const speed = 260;
    let obstacles = [], spawn = 0;

    function reset() {
      timer = TIME_LIMIT;
      last = performance.now();
      state = 'playing';
      obstacles = [];
      dino.y = h - dino.h -10;
      dino.vy = 0; dino.onGround = true;
      loop(last);
    }

    function jump(){ 
      if(state==='playing' && dino.onGround){ 
        dino.vy = JUMP; 
        dino.onGround = false 
      } 
    }
    document.addEventListener('keydown',e=>{
      if(e.code==='Space'||e.code==='ArrowUp'){e.preventDefault(); jump()} 
      if(e.key==='r') reset()
    });
    canvas.addEventListener('pointerdown', jump);

    function spawnObs(){
      const size = 20 + Math.random()*30;
      obstacles.push({x:w+10,y:h - size -10,w:size*0.6,h:size});
      spawn = 0.8 + Math.random()*0.7;
    }

    function update(dt){
      if(state!=='playing') return;
      timer -= dt;
      if(timer <=0){ state='win'; return; }

      spawn -= dt;
      if(spawn <=0) spawnObs();

      dino.vy += GRAV*dt;
      dino.y += dino.vy*dt;
      if(dino.y >= h - dino.h -10){ dino.y = h - dino.h -10; dino.vy = 0; dino.onGround = true }

      obstacles.forEach(o=>o.x -= speed*dt);
      obstacles = obstacles.filter(o=>o.x + o.w > 0);

      obstacles.forEach(o=>{
        if(dino.x < o.x + o.w && dino.x + dino.w > o.x && dino.y < o.y + o.h && dino.y + dino.h > o.y){
          state='gameover';
        }
      });
    }

    function render(){
      ctx.clearRect(0,0,w,h);

      // Dino blanco
      ctx.fillStyle = '#fff';
      ctx.fillRect(dino.x, dino.y, dino.w, dino.h);

      // Obstáculos grises
      ctx.fillStyle = '#aaa';
      obstacles.forEach(o=>ctx.fillRect(o.x,o.y,o.w,o.h));

      // Texto
      ctx.fillStyle = '#0f0';
      ctx.font = '20px monospace';
      ctx.fillText(`Tiempo: ${Math.ceil(timer)}s`, 10, 25);

      if(state==='win'){
        ctx.fillStyle='rgba(0,0,0,0.8)'; ctx.fillRect(0,0,w,h);
        ctx.fillStyle='#0f0'; ctx.fillText('¡Ganaste por tiempo!', w/2-85, h/2);
        ctx.fillText('Pulsa R para reiniciar', w/2-95, h/2+30);
      } else if(state==='gameover'){
        ctx.fillStyle='rgba(0,0,0,0.8)'; ctx.fillRect(0,0,w,h);
        ctx.fillStyle='#f00'; ctx.fillText('¡Chocaste! (Game Over)', w/2-110, h/2);
        ctx.fillText('Pulsa R para reiniciar', w/2-95, h/2+30);
      }
    }

    function loop(ts){
      const dt = Math.min((ts-last)/1000, 0.05);
      last = ts;
      update(dt);
      render();
      requestAnimationFrame(loop);
    }

    reset();
  })();
  </script>
</body>
</html>
