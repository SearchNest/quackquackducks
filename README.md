<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>SearchNest</title>
  <style>
    body, h1, button, input, p {
      font-family: 'Bookman Old Style', serif;
      margin: 0;
      background-color: black;
      color: white;
      overflow: hidden;
      display: flex;
      flex-direction: column;
      align-items: center;
      height: 100vh;
      justify-content: flex-start;
    }
    .banner {
      margin-top: 40px;
      margin-bottom: 20px;
      font-size: 2.5rem;
      user-select: none;
      z-index: 10;
    }
    .search-wrapper {
      display: flex;
      justify-content: center;
      gap: 8px;
      margin-bottom: 10px;
      width: 320px;
      z-index: 10;
    }
    .search-bar {
      background-color: white;
      border: 1px solid #ccc;
      padding: 6px 10px;
      font-family: 'Bookman Old Style', serif;
      width: 100%;
      font-size: 1rem;
      border-radius: 4px;
      outline: none;
      color: black;
      height: 32px;
      box-sizing: border-box;
    }
    .search-go-button {
      background-color: red;
      color: white;
      border: none;
      padding: 6px 16px;
      cursor: pointer;
      font-family: 'Bookman Old Style', serif;
      font-size: 1rem;
      border-radius: 4px;
      transition: background-color 0.3s ease;
      user-select: none;
      height: 32px;
      box-sizing: border-box;
    }
    .search-go-button:hover {
      background-color: darkred;
    }
    #searchLabel {
      font-family: 'Bookman Old Style', serif;
      color: white;
      font-size: 1.2rem;
      user-select: none;
      margin-top: 0;
      margin-bottom: 40px;
      text-align: center;
      z-index: 10;
    }
    canvas#particle-canvas {
      position: fixed;
      top: 0;
      left: 0;
      z-index: 0;
      width: 100vw;
      height: 100vh;
      display: block;
    }
    #searchFrame {
      position: fixed;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      border: none;
      display: none;
      z-index: 9999;
    }
    #closeIframeBtn {
      position: fixed;
      top: 15px;
      right: 15px;
      background-color: red;
      border: none;
      color: white;
      padding: 8px 12px;
      font-family: 'Bookman Old Style', serif;
      font-size: 1rem;
      border-radius: 4px;
      cursor: pointer;
      z-index: 10000;
      display: none;
      user-select: none;
    }
    #closeIframeBtn:hover {
      background-color: darkred;
    }
  </style>
</head>
<body>
  <canvas id="particle-canvas"></canvas>
  <h1 class="banner">Welcome to SearchNest!</h1>
  <div class="search-wrapper">
    <input class="search-bar" id="search" placeholder="Browse" autocomplete="off" spellcheck="false"/>
    <button class="search-go-button" id="searchButton">Search</button>
  </div>
  <p id="searchLabel">SearchNest</p>
  <iframe id="searchFrame"></iframe>
  <button id="closeIframeBtn">Close Search</button>

  <script>
    (() => {
      const canvas = document.getElementById("particle-canvas");
      const ctx = canvas.getContext("2d");
      let width = window.innerWidth;
      let height = window.innerHeight;
      function resize() {
        width = window.innerWidth;
        height = window.innerHeight;
        canvas.width = width;
        canvas.height = height;
      }
      resize();
      window.addEventListener('resize', resize);
      const NUM_PARTICLES = 200;
      const particles = [];
      let mouseX = width / 2;
      let mouseY = height / 2;
      class Particle {
        constructor() {
          this.x = Math.random() * width;
          this.y = Math.random() * height;
          this.size = 1.5 + Math.random() * 3.5;
          this.vx = (Math.random() - 0.5) * 0.5;
          this.vy = (Math.random() - 0.5) * 0.5;
          this.phase = Math.random() * Math.PI * 2;
        }
        update() {
          const dx = this.x - mouseX;
          const dy = this.y - mouseY;
          const dist = Math.sqrt(dx * dx + dy * dy);
          if (dist < 60 && dist > 0) {
            const force = (60 - dist) * 0.01;
            this.vx += (dx / dist) * force;
            this.vy += (dy / dist) * force;
          }
          this.vx *= 0.95;
          this.vy *= 0.95;
          this.x += this.vx;
          this.y += this.vy;
        }
        draw(time) {
          const pulse = (Math.sin(time / 500 + this.phase) + 1) / 2;
          const alpha = 0.7 * pulse;
          const glowSize = this.size * 2.5;
          const grad = ctx.createRadialGradient(this.x, this.y, this.size * 0.7, this.x, this.y, glowSize);
          grad.addColorStop(0, `rgba(255, 255, 255, ${alpha * 0.5})`);
          grad.addColorStop(1, 'rgba(255, 255, 255, 0)');
          ctx.fillStyle = grad;
          ctx.beginPath();
          ctx.arc(this.x, this.y, glowSize, 0, Math.PI * 2);
          ctx.fill();
          ctx.fillStyle = `rgba(255, 255, 255, ${alpha})`;
          ctx.beginPath();
          ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
          ctx.fill();
        }
      }
      for (let i = 0; i < NUM_PARTICLES; i++) {
        particles.push(new Particle());
      }
      function animate(time = 0) {
        ctx.clearRect(0, 0, width, height);
        ctx.globalCompositeOperation = 'lighter';
        for (const p of particles) {
          p.update();
          p.draw(time);
        }
        requestAnimationFrame(animate);
      }
      animate();
      window.addEventListener('mousemove', e => {
        mouseX = e.clientX;
        mouseY = e.clientY;
      });
    })();

    const searchInput = document.getElementById('search');
    const searchButton = document.getElementById('searchButton');
    const iframe = document.getElementById('searchFrame');
    const closeBtn = document.getElementById('closeIframeBtn');

    searchButton.addEventListener('click', () => {
      const query = searchInput.value.trim();
      if (!query) return;
      const url = `https://southern-political-capacity.glitch.me/search?q=${encodeURIComponent(query)}`;
      iframe.src = url;
      iframe.style.display = 'block';
      closeBtn.style.display = 'block';
      searchInput.style.display = 'none';
      searchButton.style.display = 'none';
      document.getElementById('searchLabel').style.display = 'none';
      document.querySelector('.banner').style.display = 'none';
    });

    closeBtn.addEventListener('click', () => {
      iframe.style.display = 'none';
      iframe.src = '';
      closeBtn.style.display = 'none';
      searchInput.style.display = 'block';
      searchButton.style.display = 'block';
      document.getElementById('searchLabel').style.display = 'block';
      document.querySelector('.banner').style.display = 'block';
    });

    searchInput.addEventListener('keydown', e => {
      if (e.key === 'Enter') {
        searchButton.click();
      }
    });
  </script>
</body>
</html>
