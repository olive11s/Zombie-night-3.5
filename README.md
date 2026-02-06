# Zombie-night-3.5
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Moonlight Survivor - Giselle ❤️</title>
    <style>
        /* FULL ZOOM & GESTURE LOCK */
        * { 
            margin: 0; padding: 0; box-sizing: border-box; 
            -webkit-tap-highlight-color: transparent; 
            touch-action: none; 
        }
        
        body, html { width: 100%; height: 100%; overflow: hidden; background: #000; position: fixed; font-family: -apple-system, sans-serif; }

        #game-stage { position: relative; width: 100vw; height: 100vh; background: #0a0a0a; overflow: hidden; }
        #sky { position: absolute; inset: 0; background: linear-gradient(to bottom, #050505, #1a1a2e); z-index: -2; }
        #moon { position: absolute; top: 15%; left: 50%; transform: translateX(-50%); font-size: 80px; z-index: -1; filter: drop-shadow(0 0 20px #f1c40f); }

        /* FIXED: BARREL FACING UP & POSITIONED AT 25% */
        #blaster { 
            position: absolute; bottom: 25%; left: 50%; 
            transform: translateX(-50%) rotate(-90deg); 
            font-size: 75px; z-index: 100; transition: left 0.1s ease-out; 
        }
        .shield-active { filter: drop-shadow(0 0 20px #00d2d3) brightness(1.3); }

        #hud { position: absolute; top: 8%; width: 100%; display: flex; flex-direction: column; align-items: center; z-index: 200; color: #fff; font-weight: bold; text-shadow: 2px 2px 4px #000; }
        #boss-hp-bar { display: none; width: 70%; height: 16px; background: #222; border: 2px solid #fff; border-radius: 8px; margin-top: 10px; overflow: hidden; }
        #boss-hp-fill { width: 100%; height: 100%; background: #2ecc71; transition: width 0.1s; }

        .zombie, .shield-item { position: absolute; font-size: 55px; transform: translateX(-50%); z-index: 50; }
        .boss { font-size: 140px !important; filter: drop-shadow(0 0 40px red); }
        .bolt { position: absolute; width: 6px; height: 35px; background: #f1c40f; border-radius: 5px; box-shadow: 0 0 10px #f1c40f; z-index: 60; }
        .fireball { position: absolute; font-size: 45px; transform: translateX(-50%); z-index: 55; }
        
        .sparkle { position: absolute; font-size: 20px; z-index: 70; pointer-events: none; animation: burst 0.6s forwards; }
        @keyframes burst { to { transform: translate(var(--tx), var(--ty)) scale(0); opacity: 0; } }

        #sleep-scene { display: none; position: fixed; inset: 0; background: #1a1a2e; z-index: 2000; flex-direction: column; justify-content: center; align-items: center; text-align: center; color: white; padding: 20px; }
        .overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.95); display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center; z-index: 1000; padding: 40px; color: white; }
    </style>
</head>
<body oncontextmenu="return false;">

<div id="game-stage">
    <div id="sky"></div>
    <div id="moon">🌕</div>
    <div id="blaster">🔫</div>
    <div id="hud">
        <div id="wave-txt" style="font-size: 1.6rem; color: #ff4757;">WAVE 1/10</div>
        <div id="boss-hp-bar"><div id="boss-hp-fill"></div></div>
        <div id="combo-txt" style="color: #00d2d3; font-size: 20px;">COMBO x0</div>
        <div id="lives" style="font-size: 24px; margin-top: 5px;">❤️❤️❤️</div>
    </div>
</div>

<div id="start-screen" class="overlay">
    <h1 style="color: #ff4757;">NIGHTMARE MODE 🧟‍♂️</h1>
    <p style="margin-top: 15px;">Barrel fixed! Defend the absolute bottom, Giselle!</p>
    <button style="padding: 20px 60px; border-radius: 50px; background: #ff4757; color: white; border: none; font-size: 1.3rem; font-weight: bold; margin-top: 25px;" onclick="startGame()">START DEFENSE 🎶</button>
</div>

<div id="sleep-scene">
    <div style="font-size: 100px; margin-bottom: 20px;">🛌💤</div>
    <h1 style="color: #f1c40f; font-size: 1.8rem; margin-bottom: 15px;">It was all a dream...</h1>
    <p style="line-height: 1.6; font-size: 1.1rem; padding: 0 20px;">
        I love you Giselle! My gorgeous, precious, smart, talented, sweet perfect baby!<br><br>
        Goodnight my love and sweet dreams! Remember I love you with all my heart. ❤️
    </p>
</div>

<audio id="actionMusic" loop><source src="https://www.bensound.com/bensound-music/bensound-epic.mp3" type="audio/mpeg"></audio>

<script>
    // PREVENT ALL DEFAULT ZOOMING GESTURES
    document.addEventListener('touchstart', function (e) { if (e.touches.length > 1) e.preventDefault(); }, { passive: false });
    let lastT = 0; document.addEventListener('touchend', function (e) { let n = Date.now(); if (n - lastT <= 300) e.preventDefault(); lastT = n; }, false);

    let active = false, currentWave = 1, zombiesToSpawn = 0, zombiesActive = 0, lives = 3, bossMaxHP = 60, bossHP = 60, combo = 0, hasShield = false;
    const blaster = document.getElementById('blaster');
    const hpFill = document.getElementById('boss-hp-fill');
    const music = document.getElementById('actionMusic');

    function startGame() { document.getElementById('start-screen').style.display = 'none'; active = true; music.play().catch(() => {}); startWave(); }

    function startWave() {
        if (currentWave > 10) { endToSleep(); return; }
        document.getElementById('wave-txt').innerText = currentWave === 10 ? "🚨 NIGHTMARE BOSS 🚨" : `WAVE ${currentWave}/10`;
        if (currentWave === 10) { document.getElementById('boss-hp-bar').style.display = 'block'; zombiesToSpawn = 1; zombiesActive = 1; }
        else { zombiesToSpawn = currentWave * 4; zombiesActive = zombiesToSpawn; }
        spawnZombies();
        if(currentWave > 3 && Math.random() > 0.6) spawnShield();
    }

    document.addEventListener('touchstart', (e) => { 
        if (!active) return; 
        let x = e.touches[0].clientX; 
        blaster.style.left = x + 'px'; 
        fireBolt(x); 
    }, {passive: false});

    function fireBolt(x) {
        const b = document.createElement('div'); b.className = 'bolt'; b.style.left = x + 'px'; b.style.bottom = '25%'; document.body.appendChild(b);
        let hit = false, bY = 25;
        const fly = setInterval(() => {
            bY += 10; b.style.bottom = bY + '%';
            document.querySelectorAll('.zombie').forEach(z => {
                let bR = b.getBoundingClientRect(), zR = z.getBoundingClientRect();
                if (bR.left < zR.right && bR.right > zR.left && bR.top < zR.bottom && bR.bottom > zR.top) {
                    hit = true; combo++; document.getElementById('combo-txt').innerText = `COMBO x${combo}`;
                    createSparkle(zR.left + (zR.width/2), zR.top + (zR.height/2));
                    z.classList.add('dead'); // MARK AS DEAD SO IT CAN'T TAKE HEARTS
                    z.remove();
                    b.remove(); clearInterval(fly);
                    if (z.classList.contains('boss')) { 
                        bossHP--; updateBossHP(); 
                        if (bossHP <= 0) { currentWave++; setTimeout(startWave, 800); } 
                    } else { 
                        zombiesActive--; 
                        if (zombiesActive <= 0) { currentWave++; setTimeout(startWave, 800); } 
                    }
                }
            });
            if (bY > 100) { if(!hit && !hasShield) { combo = 0; document.getElementById('combo-txt').innerText = 'COMBO x0'; } b.remove(); clearInterval(fly); }
        }, 10);
    }

    function createSparkle(x, y) {
        for(let i=0; i<8; i++) {
            const s = document.createElement('div');
            s.className = 'sparkle'; s.innerHTML = '✨';
            s.style.left = x + 'px'; s.style.top = y + 'px';
            s.style.setProperty('--tx', (Math.random()*150-75)+'px');
            s.style.setProperty('--ty', (Math.random()*150-75)+'px');
            document.body.appendChild(s);
            setTimeout(() => s.remove(), 600);
        }
    }

    function spawnShield() {
        if (!active) return;
        const s = document.createElement('div'); s.className = 'shield-item'; s.innerHTML = '✨';
        s.style.left = (Math.random() * 80 + 10) + 'vw'; s.style.top = '-50px'; document.body.appendChild(s);
        let y = -50; const fall = setInterval(() => {
            y += 5; s.style.top = y + 'px';
            let pR = blaster.getBoundingClientRect(), sR = s.getBoundingClientRect();
            if (sR.left < pR.right && sR.right > pR.left && sR.top < pR.bottom && sR.bottom > pR.top) { hasShield = true; blaster.classList.add('shield-active'); s.remove(); clearInterval(fall); }
            if (y > window.innerHeight) { s.remove(); clearInterval(fall); }
        }, 16);
    }

    function takeDamage() {
        if (hasShield) { hasShield = false; blaster.classList.remove('shield-active'); }
        else { lives--; combo = 0; document.getElementById('combo-txt').innerText = 'COMBO x0'; document.getElementById('lives').innerText = "❤️".repeat(Math.max(0, lives)); if (lives <= 0) { alert("Game Over! Try again, Giselle!"); location.reload(); } }
    }

    function spawnZombies() {
        if (!active || zombiesToSpawn <= 0) return;
        const z = document.createElement('div'); z.className = 'zombie';
        z.innerHTML = currentWave === 10 ? '👹' : '🧟‍♂️'; if(currentWave === 10) z.classList.add('boss');
        z.style.left = '50vw'; z.style.top = '-150px'; document.body.appendChild(z);
        let y = -150;
        let speed = currentWave === 10 ? 1.5 : 3.5 + (currentWave * 0.5);
        const fall = setInterval(() => {
            if (!active || z.classList.contains('dead')) { clearInterval(fall); return; }
            y += speed; z.style.top = y + 'px';
            if(currentWave === 10) {
                z.style.left = (50 + Math.sin(Date.now()/350) * 40) + 'vw';
                if (Math.random() < (bossHP < 20 ? 0.12 : 0.05)) throwFireball(z.style.left);
            }
            // DAMAGE ONLY IF ZOMBIE HITS THE ABSOLUTE BOTTOM
            if (y > window.innerHeight && !z.classList.contains('dead')) { 
                takeDamage(); z.remove(); zombiesActive--; 
                if (zombiesActive <= 0) { currentWave++; setTimeout(startWave, 800); } 
                clearInterval(fall); 
            }
        }, 16);
        zombiesToSpawn--;
        if (currentWave < 10) setTimeout(spawnZombies, 900 - (currentWave * 70));
    }

    function throwFireball(xP) {
        const f = document.createElement('div'); f.className = 'fireball'; f.innerHTML = '🔥';
        f.style.left = xP; f.style.top = '100px'; document.body.appendChild(f);
        let fY = 100; let fS = bossHP < 20 ? 16 : 10;
        const fM = setInterval(() => {
            if(!active) { clearInterval(fM); f.remove(); return; }
            fY += fS; f.style.top = fY + 'px';
            let pR = blaster.getBoundingClientRect(), fR = f.getBoundingClientRect();
            if (fR.left < pR.right && fR.right > pR.left && fR.top < pR.bottom && fR.bottom > pR.top) { takeDamage(); f.remove(); clearInterval(fM); }
            if (fY > window.innerHeight) { f.remove(); clearInterval(fM); }
        }, 16);
    }

    function updateBossHP() {
        const p = (bossHP / bossMaxHP) * 100; hpFill.style.width = p + '%';
        if (p > 60) hpFill.style.backgroundColor = "#2ecc71";
        else if (p > 30) hpFill.style.backgroundColor = "#f1c40f";
        else hpFill.style.backgroundColor = "#ff4757";
    }

    function endToSleep() { active = false; music.pause(); document.getElementById('game-stage').style.display='none'; document.getElementById('sleep-scene').style.display='flex'; }
</script>
</body>
</html>
