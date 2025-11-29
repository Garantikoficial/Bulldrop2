<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Bulldrop Ultimate</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/js/all.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700;900&display=swap" rel="stylesheet">
    
    <style>
        :root { --bg: #09090b; --card: #1c1c1c; --accent: #eab308; --danger: #ef4444; --success: #22c55e; --text: #fff; }
        body { background: var(--bg); color: var(--text); font-family: 'Montserrat', sans-serif; margin: 0; padding-bottom: 90px; user-select: none; }
        
        /* HEADER */
        .header { padding: 20px; display: flex; justify-content: space-between; align-items: center; background: rgba(24,24,27,0.9); position: sticky; top: 0; z-index: 100; border-bottom: 1px solid #333; }
        .user-info { display: flex; align-items: center; gap: 10px; }
        .avatar { width: 40px; height: 40px; border-radius: 10px; border: 2px solid var(--accent); }
        .bal-val { font-size: 18px; font-weight: 900; color: var(--accent); }

        /* SCREENS */
        .screen { display: none; padding: 20px; animation: fadeIn 0.3s; }
        .screen.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
        @keyframes shake { 0% { transform: translate(1px, 1px) rotate(0deg); } 10% { transform: translate(-1px, -2px) rotate(-1deg); } 20% { transform: translate(-3px, 0px) rotate(1deg); } 30% { transform: translate(3px, 2px) rotate(0deg); } 40% { transform: translate(1px, -1px) rotate(1deg); } 50% { transform: translate(-1px, 2px) rotate(-1deg); } 60% { transform: translate(-3px, 1px) rotate(0deg); } 70% { transform: translate(3px, 1px) rotate(-1deg); } 80% { transform: translate(-1px, -1px) rotate(1deg); } 90% { transform: translate(1px, 2px) rotate(0deg); } 100% { transform: translate(1px, -2px) rotate(-1deg); } }
        .shake { animation: shake 0.5s; }

        /* SAPPER GRID */
        .sapper-grid { display: grid; grid-template-columns: repeat(5, 1fr); gap: 8px; margin-top: 20px; }
        .cell {
            aspect-ratio: 1; background: #27272a; border-radius: 8px; cursor: pointer;
            display: flex; justify-content: center; align-items: center; font-size: 20px;
            border: 1px solid #3f3f46; transition: 0.2s;
        }
        .cell:active { transform: scale(0.9); }
        .cell.revealed { background: #18181b; border-color: #000; }
        .cell.gem { background: rgba(34, 197, 94, 0.2); border-color: var(--success); }
        .cell.bomb { background: rgba(239, 68, 68, 0.2); border-color: var(--danger); }

        /* CARDS & BUTTONS */
        .card { background: var(--card); padding: 15px; border-radius: 16px; margin-bottom: 15px; border: 1px solid #333; text-align: center; }
        .btn { width: 100%; padding: 14px; border-radius: 12px; border: none; font-weight: 800; cursor: pointer; font-size: 14px; margin-top: 10px; }
        .btn-gold { background: var(--accent); color: #000; }
        .btn-green { background: var(--success); color: #fff; }
        .btn-red { background: var(--danger); color: #fff; }
        .btn:disabled { opacity: 0.5; }

        /* ROULETTE */
        .roulette-wrapper { height: 100px; background: #000; border: 2px solid var(--accent); border-radius: 12px; margin-bottom: 20px; overflow: hidden; position: relative; display: none; }
        .r-track { display: flex; height: 100%; align-items: center; }
        .r-item { min-width: 90px; height: 80px; margin: 0 5px; background: #222; border-radius: 8px; display: flex; justify-content: center; align-items: center; font-size: 30px; }
        .r-line { position: absolute; left: 50%; top: 0; bottom: 0; width: 3px; background: #fff; transform: translateX(-50%); z-index: 10; }

        /* PET */
        .pet-box { text-align: center; background: #222; padding: 30px; border-radius: 20px; border: 1px solid #333; }
        .pet-emoji { font-size: 80px; display: inline-block; animation: float 3s infinite ease-in-out; }
        @keyframes float { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-10px); } }
        .bars { display: flex; gap: 10px; margin-top: 20px; }
        .bar-wrap { flex: 1; background: #333; height: 6px; border-radius: 3px; overflow: hidden; }
        .bar-fill { height: 100%; width: 50%; background: var(--accent); transition: 0.3s; }

        /* NAV */
        .nav { position: fixed; bottom: 0; width: 100%; background: #111; display: flex; justify-content: space-around; padding: 15px 0; border-top: 1px solid #333; }
        .nav-item { color: #666; font-size: 20px; }
        .nav-item.active { color: var(--accent); }
    </style>
</head>
<body>

    <div class="header">
        <div class="user-info">
            <img src="" id="ava" class="avatar">
            <span id="name">User</span>
        </div>
        <div class="bal-val"><span id="refs">0</span> REF</div>
    </div>

    <!-- MAIN SHOP -->
    <div id="screen-shop" class="screen active">
        <div class="roulette-wrapper" id="roulette">
            <div class="r-line"></div>
            <div class="r-track" id="track"></div>
        </div>

        <h2>Магазин</h2>
        <div class="card">
            <i class="fa-solid fa-box-open" style="font-size: 40px; color: var(--accent); margin-bottom: 10px;"></i>
            <h3>Кейс Удачи</h3>
            <p style="color: #888; font-size: 12px;">Прокрутка рулетки с шансом на код.</p>
            <button class="btn btn-gold" onclick="spin(3)">ОТКРЫТЬ (3 REF)</button>
        </div>
        <div class="card">
            <i class="fa-solid fa-bomb" style="font-size: 40px; color: var(--danger); margin-bottom: 10px;"></i>
            <h3>Сапёр</h3>
            <p style="color: #888; font-size: 12px;">Найди кристаллы и умножь рефералов.</p>
            <button class="btn btn-red" onclick="nav('sapper')">ИГРАТЬ</button>
        </div>
    </div>

    <!-- SAPPER GAME -->
    <div id="screen-sapper" class="screen">
        <div class="card">
            <h3>💣 Минное поле</h3>
            <p style="color:#aaa; font-size: 12px;">Ставка: <span id="sap-bet">1</span> REF</p>
            <div style="display:flex; gap:10px; margin: 10px 0;">
                <button class="btn btn-gold" style="margin:0" onclick="adjBet(-0.5)">-</button>
                <button class="btn btn-gold" style="margin:0" onclick="adjBet(0.5)">+</button>
            </div>
            <button class="btn btn-green" id="btn-start-sap" onclick="startSapper()">НАЧАТЬ ИГРУ</button>
            <button class="btn btn-gold" id="btn-cashout" onclick="cashOutSapper()" style="display:none;">ЗАБРАТЬ (<span id="sap-win">0</span>)</button>
        </div>
        <div class="sapper-grid" id="grid"></div>
    </div>

    <!-- PET -->
    <div id="screen-pet" class="screen">
        <div class="pet-box">
            <div class="pet-emoji" id="pet">🐶</div>
            <div class="bars">
                <div class="bar-wrap"><div class="bar-fill" id="b-hunger" style="background: #22c55e;"></div></div>
                <div class="bar-wrap"><div class="bar-fill" id="b-mood" style="background: #eab308;"></div></div>
            </div>
            <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 20px;">
                <button class="btn btn-gold" onclick="feed()">Кормить</button>
                <button class="btn btn-gold" onclick="play()">Играть</button>
            </div>
        </div>
    </div>

    <!-- NAV -->
    <div class="nav">
        <div class="nav-item active" onclick="nav('shop', this)"><i class="fa-solid fa-shop"></i></div>
        <div class="nav-item" onclick="nav('pet', this)"><i class="fa-solid fa-paw"></i></div>
    </div>

    <script>
        const tg = window.Telegram.WebApp;
        tg.expand();

        // User Data
        const params = new URLSearchParams(window.location.search);
        let refs = parseFloat(params.get('refs')) || 0;
        const uid = params.get('uid');
        const user = tg.initDataUnsafe.user || { first_name: 'Guest' };
        
        document.getElementById('name').innerText = user.first_name;
        document.getElementById('ava').src = user.photo_url || `https://ui-avatars.com/api/?name=${user.first_name}&background=eab308&color=000`;
        document.getElementById('refs').innerText = refs;

        // --- SAPPER LOGIC ---
        let sapBet = 1;
        let sapGameActive = false;
        let sapGrid = [];
        let sapMultiplier = 1.0;

        function adjBet(v) {
            if(sapGameActive) return;
            sapBet = Math.max(0.5, sapBet + v);
            document.getElementById('sap-bet').innerText = sapBet;
        }

        function startSapper() {
            if(refs < sapBet) return tg.showAlert("Не хватает рефералов!");
            refs -= sapBet;
            document.getElementById('refs').innerText = refs.toFixed(1);
            
            sapGameActive = true;
            sapMultiplier = 1.0;
            document.getElementById('btn-start-sap').style.display = 'none';
            document.getElementById('btn-cashout').style.display = 'none'; // Скрыто пока не найдешь первый гем
            
            // Генерация поля (5x4 = 20 ячеек, 4 бомбы)
            sapGrid = Array(20).fill('gem');
            for(let i=0; i<4; i++) sapGrid[i] = 'bomb';
            sapGrid.sort(() => Math.random() - 0.5);

            const gridEl = document.getElementById('grid');
            gridEl.innerHTML = '';
            sapGrid.forEach((type, idx) => {
                const cell = document.createElement('div');
                cell.className = 'cell';
                cell.onclick = () => clickCell(cell, idx, type);
                gridEl.appendChild(cell);
            });
        }

        function clickCell(el, idx, type) {
            if(!sapGameActive || el.classList.contains('revealed')) return;
            
            el.classList.add('revealed');
            tg.HapticFeedback.impactOccurred('light');

            if(type === 'bomb') {
                el.classList.add('bomb');
                el.innerHTML = '💣';
                document.body.classList.add('shake');
                setTimeout(() => document.body.classList.remove('shake'), 500);
                tg.HapticFeedback.notificationOccurred('error');
                endSapper(0);
            } else {
                el.classList.add('gem');
                el.innerHTML = '💎';
                sapMultiplier += 0.3; // Множитель растет
                const win = (sapBet * sapMultiplier).toFixed(2);
                document.getElementById('sap-win').innerText = win;
                document.getElementById('btn-cashout').style.display = 'block';
            }
        }

        function cashOutSapper() {
            if(!sapGameActive) return;
            const win = parseFloat((sapBet * sapMultiplier).toFixed(2));
            endSapper(win);
        }

        function endSapper(win) {
            sapGameActive = false;
            // Открываем все бомбы
            document.querySelectorAll('.cell').forEach((c, i) => {
                if(sapGrid[i] === 'bomb') {
                    c.classList.add('revealed', 'bomb');
                    c.innerHTML = '💣';
                }
            });

            setTimeout(() => {
                tg.sendData(JSON.stringify({ 
                    action: 'sapper_result', 
                    win_amount: win, 
                    bet: sapBet 
                }));
                tg.close();
            }, 1500);
        }

        // --- ROULETTE ---
        function spin(cost) {
            if(refs < cost) return tg.showAlert("Мало рефералов!");
            refs -= cost;
            document.getElementById('refs').innerText = refs;

            const r = document.getElementById('roulette');
            const t = document.getElementById('track');
            r.style.display = 'block';
            
            let html = '';
            for(let i=0; i<50; i++) {
                const item = Math.random() > 0.8 ? '🎁' : (Math.random() > 0.5 ? '💩' : '🪙');
                html += `<div class="r-item">${item}</div>`;
            }
            t.innerHTML = html;

            setTimeout(() => {
                t.style.transition = 'transform 4s cubic-bezier(0.1, 0.9, 0.2, 1)';
                t.style.transform = 'translateX(-3500px)';
                let s = setInterval(() => tg.HapticFeedback.selectionChanged(), 100);
                setTimeout(() => clearInterval(s), 4000);
            }, 50);

            setTimeout(() => {
                tg.HapticFeedback.notificationOccurred('success');
                confetti({ particleCount: 100, spread: 70, origin: { y: 0.6 } });
                setTimeout(() => {
                    tg.sendData(JSON.stringify({ action: 'open_case' }));
                    tg.close();
                }, 1000);
            }, 4100);
        }

        // --- NAV & PET ---
        function nav(id, el) {
            document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
            document.getElementById('screen-' + id).classList.add('active');
            if(el) {
                document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
                el.classList.add('active');
            }
        }

        let hunger = 80, mood = 80;
        function feed() { hunger = Math.min(100, hunger+20); updatePet(); tg.HapticFeedback.impactOccurred('light'); }
        function play() { mood = Math.min(100, mood+20); updatePet(); tg.HapticFeedback.impactOccurred('light'); }
        function updatePet() {
            document.getElementById('b-hunger').style.width = hunger + '%';
            document.getElementById('b-mood').style.width = mood + '%';
        }
    </script>
</body>
</html>


