<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Bulldrop Ultimate</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/js/all.min.js"></script>
    <!-- Подключаем CanvasConfetti для салюта -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700;900&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --bg: #09090b;
            --card-bg: rgba(30, 30, 30, 0.6);
            --accent: #fbbf24;
            --accent-glow: rgba(251, 191, 36, 0.5);
            --danger: #ef4444;
            --success: #22c55e;
            --text: #ffffff;
        }

        body {
            background-color: var(--bg);
            background-image: radial-gradient(circle at 50% 0%, #1a1a1a 0%, var(--bg) 70%);
            color: var(--text);
            font-family: 'Montserrat', sans-serif;
            margin: 0;
            padding-bottom: 90px;
            overflow-x: hidden;
            -webkit-font-smoothing: antialiased;
        }

        /* АНИМАЦИИ */
        @keyframes float { 0% { transform: translateY(0px); } 50% { transform: translateY(-10px); } 100% { transform: translateY(0px); } }
        @keyframes breathe { 0% { transform: scale(1); } 50% { transform: scale(1.05); } 100% { transform: scale(1); } }
        @keyframes slideIn { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
        @keyframes glitch { 0% { transform: translate(0); } 20% { transform: translate(-2px, 2px); } 40% { transform: translate(-2px, -2px); } 60% { transform: translate(2px, 2px); } 80% { transform: translate(2px, -2px); } 100% { transform: translate(0); } }

        /* HEADER */
        .header {
            padding: 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: rgba(10, 10, 10, 0.8);
            backdrop-filter: blur(15px);
            border-bottom: 1px solid rgba(255,255,255,0.1);
            position: sticky; top: 0; z-index: 100;
        }

        .user-info { display: flex; align-items: center; gap: 12px; }
        .avatar { width: 45px; height: 45px; border-radius: 12px; border: 2px solid var(--accent); box-shadow: 0 0 15px var(--accent-glow); }
        .balance-box { text-align: right; }
        .balance-val { font-size: 20px; font-weight: 900; color: var(--accent); text-shadow: 0 0 10px var(--accent-glow); }
        .balance-label { font-size: 10px; color: #888; letter-spacing: 1px; }

        /* SCREENS */
        .screen { display: none; padding: 20px; animation: slideIn 0.4s ease-out; }
        .screen.active { display: block; }

        /* PET SYSTEM */
        .pet-container {
            text-align: center;
            margin-top: 20px;
            background: var(--card-bg);
            border-radius: 24px;
            padding: 30px;
            border: 1px solid rgba(255,255,255,0.05);
            position: relative;
            overflow: hidden;
        }
        
        /* Свечение за петом */
        .pet-glow {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            width: 150px; height: 150px;
            background: var(--accent);
            filter: blur(80px);
            opacity: 0.2;
            z-index: 0;
        }

        .pet-avatar {
            font-size: 100px;
            margin-bottom: 20px;
            display: inline-block;
            animation: float 3s ease-in-out infinite;
            position: relative;
            z-index: 1;
            filter: drop-shadow(0 10px 20px rgba(0,0,0,0.5));
        }

        .pet-stats { display: flex; justify-content: space-between; margin-bottom: 20px; gap: 10px; position: relative; z-index: 1; }
        .stat-bar { flex: 1; background: rgba(255,255,255,0.1); height: 8px; border-radius: 4px; overflow: hidden; }
        .stat-fill { height: 100%; transition: width 0.5s ease; }
        .stat-fill.hunger { background: var(--success); }
        .stat-fill.mood { background: var(--accent); }
        .stat-label { font-size: 10px; color: #aaa; margin-bottom: 4px; text-align: left; }

        .pet-actions { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; position: relative; z-index: 1; }
        .action-btn {
            background: rgba(255,255,255,0.05);
            border: 1px solid rgba(255,255,255,0.1);
            color: #fff;
            padding: 12px;
            border-radius: 12px;
            cursor: pointer;
            transition: 0.2s;
            display: flex; flex-direction: column; align-items: center; gap: 5px;
        }
        .action-btn:active { transform: scale(0.95); background: rgba(255,255,255,0.1); }
        .action-btn i { font-size: 20px; color: var(--accent); }

        /* SHOP CARDS (3D Effect) */
        .card {
            background: var(--card-bg);
            border-radius: 20px;
            padding: 20px;
            margin-bottom: 20px;
            border: 1px solid rgba(255,255,255,0.05);
            position: relative;
            transform-style: preserve-3d;
            transition: transform 0.1s;
        }
        
        .card-header { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 15px; }
        .card-icon { font-size: 40px; color: var(--accent); filter: drop-shadow(0 0 10px var(--accent-glow)); }
        .card-price { background: #222; padding: 6px 12px; border-radius: 20px; font-size: 12px; font-weight: bold; border: 1px solid #333; }
        
        .main-btn {
            width: 100%;
            padding: 15px;
            border-radius: 16px;
            border: none;
            background: linear-gradient(135deg, var(--accent), #d97706);
            color: #000;
            font-weight: 900;
            font-size: 16px;
            text-transform: uppercase;
            cursor: pointer;
            box-shadow: 0 5px 20px rgba(251, 191, 36, 0.3);
            position: relative;
            overflow: hidden;
        }
        .main-btn::after {
            content: ''; position: absolute; top: -50%; left: -50%; width: 200%; height: 200%;
            background: linear-gradient(rgba(255,255,255,0.2), transparent);
            transform: rotate(45deg); animation: shine 3s infinite;
        }
        @keyframes shine { 0% { left: -100%; } 20% { left: 100%; } 100% { left: 100%; } }
        .main-btn:active { transform: scale(0.98); }

        /* ROULETTE STYLES */
        .roulette-wrapper {
            background: #000; border: 2px solid var(--accent); border-radius: 16px; 
            height: 100px; overflow: hidden; position: relative; margin-bottom: 20px;
            display: none; box-shadow: 0 0 20px var(--accent-glow);
        }
        .roulette-track { display: flex; height: 100%; align-items: center; transition: transform 6s cubic-bezier(0.1, 0.95, 0.1, 1); }
        .r-item { 
            min-width: 90px; height: 80px; margin: 0 4px; background: #1a1a1a; 
            border-radius: 8px; display: flex; align-items: center; justify-content: center; font-size: 24px;
            border-bottom: 3px solid #333;
        }
        .r-line { 
            position: absolute; left: 50%; top: 0; bottom: 0; width: 2px; 
            background: #fff; z-index: 10; transform: translateX(-50%); 
            box-shadow: 0 0 10px white;
        }

        /* NAVIGATION */
        .nav {
            position: fixed; bottom: 20px; left: 20px; right: 20px;
            background: rgba(30,30,30,0.9); backdrop-filter: blur(20px);
            border-radius: 20px; padding: 15px;
            display: flex; justify-content: space-around;
            border: 1px solid rgba(255,255,255,0.1);
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
        }
        .nav-item { 
            color: #666; font-size: 20px; cursor: pointer; position: relative; 
            transition: color 0.3s;
        }
        .nav-item.active { color: var(--accent); transform: translateY(-2px); }
        .nav-item.active::after {
            content: ''; position: absolute; bottom: -5px; left: 50%; transform: translateX(-50%);
            width: 4px; height: 4px; background: var(--accent); border-radius: 50%;
        }

    </style>
</head>
<body>

    <!-- HEADER -->
    <div class="header">
        <div class="user-info">
            <img src="" id="avatar" class="avatar">
            <div>
                <div id="username" style="font-weight: 800; font-size: 14px;">User</div>
                <div style="font-size: 10px; color: var(--accent);">PRO MEMBER</div>
            </div>
        </div>
        <div class="balance-box">
            <div class="balance-label">БАЛАНС</div>
            <div class="balance-val"><span id="refs">0</span> REF</div>
        </div>
    </div>

    <!-- 1. SHOP SCREEN -->
    <div id="screen-shop" class="screen active">
        <!-- ROULETTE AREA -->
        <div class="roulette-wrapper" id="roulette">
            <div class="r-line"></div>
            <div class="roulette-track" id="track"></div>
        </div>

        <h2 style="margin-bottom: 15px;">Магазин Кейсов</h2>

        <div class="card">
            <div class="card-header">
                <i class="fa-solid fa-briefcase card-icon"></i>
                <div class="card-price">3 REF</div>
            </div>
            <h3>Standart Case</h3>
            <p style="color: #aaa; font-size: 12px; margin-bottom: 15px;">Высокий шанс выпадения промокода на 100G.</p>
            <button class="main-btn" onclick="spin(3)">ОТКРЫТЬ</button>
        </div>

        <div class="card">
            <div class="card-header">
                <i class="fa-solid fa-gem card-icon" style="color: #a855f7;"></i>
                <div class="card-price">5 REF</div>
            </div>
            <h3>Premium Random</h3>
            <p style="color: #aaa; font-size: 12px; margin-bottom: 15px;">Моментальная покупка случайного кода без анимации.</p>
            <button class="main-btn" style="background: linear-gradient(135deg, #a855f7, #7c3aed);" onclick="buyRandom(5)">КУПИТЬ</button>
        </div>
    </div>

    <!-- 2. PET SCREEN -->
    <div id="screen-pet" class="screen">
        <div class="pet-container">
            <div class="pet-glow"></div>
            <div class="pet-avatar" id="pet-visual">🐶</div>
            
            <div class="pet-stats">
                <div style="flex:1">
                    <div class="stat-label">Сытость</div>
                    <div class="stat-bar"><div class="stat-fill hunger" id="bar-hunger" style="width: 80%;"></div></div>
                </div>
                <div style="flex:1">
                    <div class="stat-label">Настроение</div>
                    <div class="stat-bar"><div class="stat-fill mood" id="bar-mood" style="width: 60%;"></div></div>
                </div>
            </div>

            <div class="pet-actions">
                <div class="action-btn" onclick="feedPet()">
                    <i class="fa-solid fa-bone"></i>
                    <span>Кормить</span>
                </div>
                <div class="action-btn" onclick="playPet()">
                    <i class="fa-solid fa-basketball"></i>
                    <span>Играть</span>
                </div>
            </div>
        </div>
        <div style="text-align: center; margin-top: 20px; font-size: 12px; color: #666;">
            Ухаживай за питомцем, чтобы повышать удачу в кейсах!
        </div>
    </div>

    <!-- 3. PROFILE SCREEN -->
    <div id="screen-profile" class="screen">
        <div class="card" style="text-align: center; padding: 40px 20px;">
            <i class="fa-solid fa-share-nodes" style="font-size: 50px; color: var(--accent); margin-bottom: 20px;"></i>
            <h2>Пригласи друга</h2>
            <p style="color: #888; font-size: 13px; margin-bottom: 20px;">
                За каждого приглашенного друга ты получаешь +1 REF на баланс.
            </p>
            <button class="main-btn" onclick="invite()">ПОЛУЧИТЬ ССЫЛКУ</button>
        </div>
    </div>

    <!-- NAVIGATION -->
    <div class="nav">
        <div class="nav-item active" onclick="nav('shop', this)"><i class="fa-solid fa-store"></i></div>
        <div class="nav-item" onclick="nav('pet', this)"><i class="fa-solid fa-paw"></i></div>
        <div class="nav-item" onclick="nav('profile', this)"><i class="fa-solid fa-user"></i></div>
    </div>

    <script>
        const tg = window.Telegram.WebApp;
        tg.expand();
        tg.enableClosingConfirmation();

        // 1. ПОЛУЧЕНИЕ ДАННЫХ ИЗ URL (СВЯЗЬ С БОТОМ)
        const urlParams = new URLSearchParams(window.location.search);
        let currentRefs = parseFloat(urlParams.get('refs')) || 0;
        const uid = urlParams.get('uid');

        // Инициализация пользователя
        const user = tg.initDataUnsafe.user || { first_name: 'Guest', id: uid || 0 };
        document.getElementById('username').innerText = user.first_name;
        document.getElementById('avatar').src = user.photo_url || `https://ui-avatars.com/api/?name=${user.first_name}&background=fbbf24&color=000`;
        document.getElementById('refs').innerText = currentRefs;

        // --- PET SYSTEM (Тамагочи) ---
        let pet = JSON.parse(localStorage.getItem('bulldrop_pet')) || { hunger: 80, mood: 80, lastUpdate: Date.now() };
        
        // Обновление состояния пета (со временем он голодает)
        const now = Date.now();
        const hoursPassed = (now - pet.lastUpdate) / (1000 * 60 * 60);
        if(hoursPassed > 1) {
            pet.hunger = Math.max(0, pet.hunger - (hoursPassed * 5));
            pet.mood = Math.max(0, pet.mood - (hoursPassed * 5));
            pet.lastUpdate = now;
        }

        function updatePetUI() {
            document.getElementById('bar-hunger').style.width = pet.hunger + '%';
            document.getElementById('bar-mood').style.width = pet.mood + '%';
            
            const visual = document.getElementById('pet-visual');
            if(pet.hunger < 30 || pet.mood < 30) {
                visual.innerText = '🥺'; // Грустный
                visual.style.animation = 'none';
            } else {
                visual.innerText = '🐶'; // Веселый
                visual.style.animation = 'float 3s ease-in-out infinite';
            }
            localStorage.setItem('bulldrop_pet', JSON.stringify(pet));
        }
        updatePetUI();

        function feedPet() {
            if(pet.hunger >= 100) return tg.showAlert("Питомец не голоден!");
            pet.hunger = Math.min(100, pet.hunger + 20);
            animatePet('😋');
            updatePetUI();
            tg.HapticFeedback.impactOccurred('light');
        }

        function playPet() {
            if(pet.mood >= 100) return tg.showAlert("Питомец счастлив!");
            pet.mood = Math.min(100, pet.mood + 20);
            pet.hunger = Math.max(0, pet.hunger - 5); // Игра тратит сытость
            animatePet('🤪');
            updatePetUI();
            tg.HapticFeedback.impactOccurred('light');
        }

        function animatePet(emoji) {
            const visual = document.getElementById('pet-visual');
            const old = visual.innerText;
            visual.innerText = emoji;
            visual.style.transform = 'scale(1.2) rotate(10deg)';
            setTimeout(() => {
                visual.innerText = '🐶';
                visual.style.transform = 'scale(1) rotate(0deg)';
            }, 1000);
        }

        // --- SHOP & ROULETTE ---
        function spin(cost) {
            if(currentRefs < cost) {
                tg.HapticFeedback.notificationOccurred('error');
                return tg.showAlert(`Не хватает рефералов! Нужно: ${cost}`);
            }

            // UI Setup
            const r = document.getElementById('roulette');
            const t = document.getElementById('track');
            r.style.display = 'block';
            
            // Generate Items
            let html = '';
            const items = ['💩', '🪙', '🎟️', '💎', '📦'];
            for(let i=0; i<70; i++) {
                const icon = items[Math.floor(Math.random() * items.length)];
                // 60-й элемент - выигрышный (визуально)
                const isWin = i === 60;
                const style = isWin ? 'border-bottom: 3px solid var(--accent); color: var(--accent);' : '';
                html += `<div class="r-item" style="${style}">${isWin ? '🎁' : icon}</div>`;
            }
            t.innerHTML = html;

            // Start Animation
            setTimeout(() => {
                // Вычисляем сдвиг, чтобы 60-й элемент встал по центру
                // 98px (width+margin) * 60
                const offset = (60 * 98) - (r.offsetWidth / 2) + 45;
                t.style.transform = `translateX(-${offset}px)`;
                
                // Звук (вибрация) во время прокрутки
                let clicker = setInterval(() => tg.HapticFeedback.selectionChanged(), 150);
                setTimeout(() => clearInterval(clicker), 6000);

            }, 50);

            // Finish
            setTimeout(() => {
                confetti({
                    particleCount: 100,
                    spread: 70,
                    origin: { y: 0.6 },
                    colors: ['#fbbf24', '#ffffff']
                });
                tg.HapticFeedback.notificationOccurred('success');
                
                // Отправка данных боту
                setTimeout(() => {
                    tg.sendData(JSON.stringify({ action: 'open_case' }));
                    tg.close();
                }, 1500);
            }, 6000);
        }

        function buyRandom(cost) {
            if(currentRefs < cost) return tg.showAlert(`Не хватает рефералов!`);
            tg.showConfirm("Купить моментально?", (ok) => {
                if(ok) {
                    tg.sendData(JSON.stringify({ action: 'buy_random' }));
                    tg.close();
                }
            });
        }

        function invite() {
            const link = `https://t.me/Bulldroprom_bot?start=${user.id}`;
            tg.openTelegramLink(`https://t.me/share/url?url=${encodeURIComponent(link)}&text=Забирай+промокоды!`);
        }

        // --- NAVIGATION ---
        function nav(screen, el) {
            document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
            document.getElementById('screen-' + screen).classList.add('active');
            
            document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
            el.classList.add('active');
            tg.HapticFeedback.selectionChanged();
        }

        // 3D Card Effect
        document.querySelectorAll('.card').forEach(card => {
            card.addEventListener('mousemove', (e) => {
                const rect = card.getBoundingClientRect();
                const x = e.clientX - rect.left;
                const y = e.clientY - rect.top;
                card.style.transform = `perspective(1000px) rotateX(${(y - rect.height/2)/10}deg) rotateY(${-(x - rect.width/2)/10}deg)`;
            });
            card.addEventListener('mouseleave', () => {
                card.style.transform = 'perspective(1000px) rotateX(0) rotateY(0)';
            });
        });
    </script>
</body>
</html>



