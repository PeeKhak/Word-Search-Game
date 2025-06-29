<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Word Search Game</title>
    <style>
        body {
            font-family: 'Segoe UI', Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            background: linear-gradient(to bottom, #d9efff, #a3d2ff);
            margin: 0;
            padding: 20px;
            user-select: none;
            touch-action: manipulation; /* Improve touch responsiveness */
        }
        #game-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            background: linear-gradient(145deg, #ffffff, #f5f5f5);
            padding: 25px;
            border-radius: 20px;
            box-shadow: 0 4px 25px rgba(0,0,0,0.15);
            max-width: 95vw;
        }
        #grid-container {
            position: relative;
            overflow: auto;
            max-width: calc(100vw - 40px); /* Fit viewport width */
            max-height: 60vh; /* Minimize vertical scrolling */
            margin-bottom: 20px;
            background: linear-gradient(145deg, #f0f0f0, #e0e0e0);
            border-radius: 12px;
            padding: 10px;
            display: flex;
            justify-content: center;
            touch-action: none; /* Prevent default touch scrolling in grid */
        }
        #grid {
            display: grid;
            gap: 4px;
            background: linear-gradient(145deg, #5a5a5a, #4a4a4a);
            padding: 8px;
            border-radius: 10px;
        }
        .cell {
            display: flex;
            align-items: center;
            justify-content: center;
            background: linear-gradient(145deg, #ffffff, #e6e6e6);
            font-size: 18px;
            text-transform: uppercase;
            cursor: pointer;
            border-radius: 10px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.1);
            transition: transform 0.2s, background-color 0.3s, box-shadow 0.2s;
            animation: cellReveal 0.5s ease-in forwards;
            z-index: 1;
        }
        @keyframes cellReveal {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .cell:hover {
            transform: scale(1.15);
            box-shadow: 0 4px 12px rgba(0,0,0,0.2);
        }
        .cell.selected {
            background: linear-gradient(145deg, #aaffaa, #88ff88);
            transform: scale(1.1);
            box-shadow: 0 4px 12px rgba(0,255,0,0.3);
        }
        .cell.found {
            background: linear-gradient(145deg, #4CAF50, #3d8b40);
            color: white;
            animation: fadeIn 0.5s ease-in;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: scale(0.8); }
            to { opacity: 1; transform: scale(1); }
        }
        #word-list {
            max-width: 250px;
            text-align: center;
        }
        #word-list ul {
            list-style: none;
            padding: 0;
        }
        #word-list li {
            margin: 10px 0;
            font-size: 16px;
            font-weight: 500;
            transition: color 0.3s, transform 0.3s;
        }
        #word-list li.found {
            text-decoration: line-through;
            color: #4CAF50;
            animation: wordFound 0.5s ease-in;
        }
        @keyframes wordFound {
            from { transform: scale(1); }
            to { transform: scale(1.1); }
        }
        #message {
            margin-top: 20px;
            font-size: 1.2em;
            color: #333;
            transition: opacity 0.5s;
        }
        .button {
            padding: 12px 24px;
            font-size: 1em;
            cursor: pointer;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 8px;
            margin: 10px;
            transition: background-color 0.3s, transform 0.2s, box-shadow 0.2s;
        }
        .button:hover {
            background-color: #45a049;
            transform: scale(1.05);
            box-shadow: 0 4px 12px rgba(0,0,0,0.2);
        }
        .mode-button {
            background: linear-gradient(145deg, #2196F3, #1976D2);
        }
        .mode-button:hover {
            background: linear-gradient(145deg, #1976D2, #1565C0);
        }
        .win-animation {
            animation: winPulse 1s infinite;
        }
        @keyframes winPulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }
        #canvas {
            position: absolute;
            top: 0;
            left: 0;
            pointer-events: none;
            z-index: 2;
        }
        #confetti-canvas {
            position: fixed;
            top: 0;
            left: 0;
            pointer-events: none;
            z-index: 4;
        }
        .modal {
            display: none;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: linear-gradient(145deg, #ffffff, #f0f0f0);
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.3);
            z-index: 3;
            max-width: 400px;
            text-align: center;
        }
        .modal button {
            padding: 10px 20px;
            font-size: 16px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        .modal button:hover {
            background-color: #45a049;
        }
        .modal-content {
            font-size: 14px;
            line-height: 1.5;
            margin-bottom: 20px;
        }
        #mode-selection {
            margin: 10px 0;
        }
        #mode-label {
            font-size: 1.2em;
            font-weight: 600;
            margin-bottom: 10px;
            color: #333;
        }
        #credits {
            margin-top: 20px;
            font-size: 0.9em;
            color: #555;
            text-align: center;
        }
        #credits p {
            margin: 5px 0;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <h1 id="game-title">Word Search</h1>
        <div id="mode-selection">
            <div id="mode-label">Choose to play the level</div>
            <button class="button mode-button" onclick="setMode('easy')">Easy</button>
            <button class="button mode-button" onclick="setMode('normal')">Normal</button>
            <button class="button mode-button" onclick="setMode('hard')">Hard</button>
        </div>
        <div id="grid-container" style="display: none;">
            <canvas id="canvas"></canvas>
            <div id="grid"></div>
        </div>
        <div id="message"></div>
        <div>
            <button id="restart-button" class="button" style="display: none;">New Game</button>
            <button id="info-button" class="button">How to Play</button>
        </div>
        <div id="word-list" style="display: none;">
            <h2>Words to Find</h2>
            <ul id="word-list-ul"></ul>
            <div id="credits">
                <p>@credit by PeeKhak</p>
                <p>Thanks for Playing My Game</p>
            </div>
        </div>
    </div>
    <canvas id="confetti-canvas"></canvas>
    <div id="info-modal" class="modal">
        <h3>How to Play</h3>
        <div class="modal-content">
            <p>Welcome to Word Search! Find the hidden words in the grid:</p>
            <ul>
                <li>Choose a mode: Easy (8x8, 10 words), Normal (12x12, 18 words), or Hard (20x20, 30 words) using the buttons.</li>
                <li>On desktop, click and drag to select letters in the grid (horizontally, vertically, or diagonally).</li>
                <li>On mobile, touch and hold a letter, then drag to select words.</li>
                <li>Match words from the list below the grid (forward or backward).</li>
                <li>Found words are highlighted with a green gradient and struck through in the list.</li>
                <li>A colored line (blue, purple, etc.) shows your selection; a vibrant line (green, red, etc.) highlights found words with a glow effect.</li>
                <li>Hear a high-pitched beep for correct words, low-pitched for incorrect.</li>
                <li>Win by finding all words, triggering a confetti burst and pulsing animation!</li>
                <li>Click "New Game" to choose a mode and start with new words.</li>
                <li>Click "How to Play" to view these instructions.</li>
            </ul>
        </div>
        <button id="close-info">Close</button>
    </div>

    <script>
        let GRID_SIZE = 8; // Default to Easy
        const modeSettings = {
            easy: { size: 8, cellSize: 38, name: 'Easy', wordCount: 10 },
            normal: { size: 12, cellSize: 32, name: 'Normal', wordCount: 18 },
            hard: { size: 20, cellSize: 28, name: 'Hard', wordCount: 30 }
        };
        const wordPool = {
            animals: [
                'TIGER', 'LION', 'WOLF', 'BEAR', 'FOX', 'DEER', 'EAGLE', 'HAWK', 'SNAKE', 'PANDA',
                'CHEETAH', 'ZEBRA', 'GIRAFFE', 'RHINO', 'MOUSE', 'ELEPHANT', 'KANGAROO', 'LEOPARD', 'BUFFALO', 'WILDCAT',
                'HYENA', 'OTTER', 'JAGUAR', 'COUGAR', 'LYNX', 'WALRUS', 'SEAL', 'DOLPHIN', 'WHALE', 'SHARK'
            ],
            tools: [
                'HAMMER', 'SCREW', 'WRENCH', 'DRILL', 'SAW', 'NAIL', 'BOLT', 'AXE', 'PLIER', 'CHISEL',
                'SCISSOR', 'LADDER', 'TAPE', 'BRUSH', 'KNIFE', 'SHOVEL', 'RULER', 'SCREWDRIVER', 'HACKSAW', 'CROWBAR',
                'MALLET', 'TROWEL', 'SANDER', 'LEVEL', 'VISE', 'CLAMP', 'GRINDER', 'SHEARS', 'ANVIL', 'DRILLBIT'
            ],
            cities: [
                'PARIS', 'TOKYO', 'LONDON', 'ROME', 'CAIRO', 'SYDNEY', 'LAGOS', 'SEOUL', 'MIAMI', 'DUBAI',
                'BERLIN', 'MOSCOW', 'MUMBAI', 'CHICAGO', 'N YORK', 'BANGKOK', 'TORONTO', 'MADRID', 'BEIJING', 'ATHENS',
                'LISBON', 'DUBLIN', 'NAIROBI', 'DELHI', 'RIYADH', 'HAVANA', 'VIENNA', 'OSLO', 'PRAGUE', 'HANOI'
            ],
            food: [
                'APPLE', 'BANANA', 'CHERRY', 'DATE', 'MANGO', 'GRAPE', 'LEMON', 'PEACH', 'PLUM', 'KIWI',
                'BREAD', 'PASTA', 'PIZZA', 'SUSHI', 'TACO', 'SALAD', 'BURGER', 'STEAK', 'SOUP', 'PASTRY',
                'CEREAL', 'CHEESE', 'YOGURT', 'FRIES', 'CURRY', 'NOODLE', 'SALMON', 'CAKE', 'PIE', 'DONUT'
            ],
            sports: [
                'SOCCER', 'TENNIS', 'GOLF', 'RUGBY', 'CRICKET', 'HOCKEY', 'BOXING', 'JUDO', 'SKIING', 'SWIM',
                'BASKET', 'VOLLEY', 'RACE', 'DIVE', 'ARCHERY', 'BASEBALL', 'CYCLING', 'FENCING', 'SURFING', 'KARATE',
                'WRESTLING', 'ROWING', 'SAILING', 'CLIMBING', 'SKATING', 'GYMNAST', 'SQUASH', 'BADMINTON', 'POLO', 'KAYAK'
            ]
        };
        let grid = [];
        let selectedCells = [];
        let foundWords = [];
        let placedWords = [];
        let currentMode = 'easy';
        const colors = {
            select: ['blue', 'purple', 'orange', 'cyan', 'magenta', '#ff6347', '#00ced1', '#ff69b4', '#7b68ee', '#ff4500'],
            found: ['green', 'red', 'yellow', 'lime', 'pink', '#32cd32', '#ff4500', '#ffd700', '#adff2f', '#ff1493']
        };

        // Sound effects
        function playSound(type) {
            const ctx = new (window.AudioContext || window.webkitAudioContext)();
            const oscillator = ctx.createOscillator();
            oscillator.type = 'sine';
            oscillator.frequency.setValueAtTime(type === 'success' ? 440 : 220, ctx.currentTime);
            oscillator.connect(ctx.destination);
            oscillator.start();
            oscillator.stop(ctx.currentTime + 0.2);
        }

        // Canvas for drawing lines
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        function resizeCanvas() {
            canvas.width = document.getElementById('grid').offsetWidth;
            canvas.height = document.getElementById('grid').offsetHeight;
        }

        function drawLine(cells, colorType = 'select') {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            if (cells.length < 2) return;
            const colorArray = colors[colorType];
            const color = colorArray[Math.floor(Math.random() * colorArray.length)];
            ctx.beginPath();
            ctx.strokeStyle = color;
            ctx.lineWidth = 4;
            ctx.lineCap = 'round';
            ctx.shadowColor = colorType === 'found' ? color : 'rgba(0,0,0,0.3)';
            ctx.shadowBlur = colorType === 'found' ? 10 : 5;
            const firstCell = cells[0];
            const rect = firstCell.getBoundingClientRect();
            const gridRect = document.getElementById('grid').getBoundingClientRect();
            ctx.moveTo(
                rect.left - gridRect.left + rect.width / 2,
                rect.top - gridRect.top + rect.height / 2
            );
            for (let i = 1; i < cells.length; i++) {
                const rect = cells[i].getBoundingClientRect();
                ctx.lineTo(
                    rect.left - gridRect.left + rect.width / 2,
                    rect.top - gridRect.top + rect.height / 2
                );
            }
            ctx.stroke();
            ctx.shadowBlur = 0;
        }

        // Confetti animation
        const confettiCanvas = document.getElementById('confetti-canvas');
        const confettiCtx = confettiCanvas.getContext('2d');
        let particles = [];
        function initConfetti() {
            confettiCanvas.width = window.innerWidth;
            confettiCanvas.height = window.innerHeight;
        }
        function createParticle() {
            return {
                x: Math.random() * confettiCanvas.width,
                y: 0,
                size: Math.random() * 5 + 5,
                speed: Math.random() * 5 + 2,
                color: colors.found[Math.floor(Math.random() * colors.found.length)],
                angle: Math.random() * Math.PI * 2
            };
        }
        function drawConfetti() {
            confettiCtx.clearRect(0, 0, confettiCanvas.width, confettiCanvas.height);
            particles.forEach(p => {
                confettiCtx.fillStyle = p.color;
                confettiCtx.beginPath();
                confettiCtx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
                confettiCtx.fill();
                p.y += p.speed;
                p.x += Math.sin(p.angle) * 2;
                p.angle += 0.1;
            });
            particles = particles.filter(p => p.y < confettiCanvas.height);
            if (particles.length > 0) {
                requestAnimationFrame(drawConfetti);
            }
        }
        function startConfetti() {
            particles = Array(100).fill().map(createParticle);
            drawConfetti();
        }
        window.addEventListener('resize', initConfetti);
        initConfetti();

        function getRandomWords() {
            const pool = Object.values(wordPool).flat();
            const shuffled = pool.sort(() => 0.5 - Math.random());
            const numWords = modeSettings[currentMode].wordCount;
            return shuffled.slice(0, numWords);
        }

        function generateGrid() {
            grid = Array(GRID_SIZE).fill().map(() => Array(GRID_SIZE).fill(''));
            placedWords = [];
            const words = getRandomWords();
            for (const word of words) {
                if (placeWord(word)) {
                    placedWords.push(word);
                }
            }
            for (let i = 0; i < GRID_SIZE; i++) {
                for (let j = 0; j < GRID_SIZE; j++) {
                    if (!grid[i][j]) {
                        grid[i][j] = String.fromCharCode(65 + Math.floor(Math.random() * 26));
                    }
                }
            }
            if (placedWords.length < words.length) {
                generateGrid();
            }
        }

        function placeWord(word) {
            const directions = [
                [0, 1], [1, 0], [1, 1], [-1, -1],
                [0, -1], [-1, 0], [-1, 1], [1, -1]
            ];
            let attempts = 0;
            const maxAttempts = 100;
            while (attempts < maxAttempts) {
                attempts++;
                const dir = directions[Math.floor(Math.random() * directions.length)];
                const row = Math.floor(Math.random() * GRID_SIZE);
                const col = Math.floor(Math.random() * GRID_SIZE);
                if (canPlaceWord(word, row, col, dir)) {
                    for (let i = 0; i < word.length; i++) {
                        const r = row + i * dir[0];
                        const c = col + i * dir[1];
                        grid[r][c] = word[i];
                    }
                    return true;
                }
            }
            return false;
        }

        function canPlaceWord(word, row, col, dir) {
            for (let i = 0; i < word.length; i++) {
                const r = row + i * dir[0];
                const c = col + i * dir[1];
                if (r < 0 || r >= GRID_SIZE || c < 0 || c >= GRID_SIZE) return false;
                if (grid[r][c] && grid[r][c] !== word[i]) return false;
            }
            return true;
        }

        function renderGrid() {
            const gridDiv = document.getElementById('grid');
            gridDiv.style.gridTemplateColumns = `repeat(${GRID_SIZE}, ${modeSettings[currentMode].cellSize}px)`;
            gridDiv.innerHTML = '';
            for (let i = 0; i < GRID_SIZE; i++) {
                for (let j = 0; j < GRID_SIZE; j++) {
                    const cell = document.createElement('div');
                    cell.className = 'cell';
                    cell.style.width = `${modeSettings[currentMode].cellSize}px`;
                    cell.style.height = `${modeSettings[currentMode].cellSize}px`;
                    cell.textContent = grid[i][j];
                    cell.dataset.row = i;
                    cell.dataset.col = j;
                    cell.style.animationDelay = `${(i * GRID_SIZE + j) * 0.02}s`;
                    cell.addEventListener('mousedown', startSelection);
                    cell.addEventListener('touchstart', handleTouchStart);
                    gridDiv.appendChild(cell);
                }
            }
            resizeCanvas();
        }

        function renderWordList() {
            const wordListUl = document.getElementById('word-list-ul');
            wordListUl.innerHTML = '';
            placedWords.forEach(word => {
                const li = document.createElement('li');
                li.textContent = word;
                if (foundWords.includes(word)) {
                    li.className = 'found';
                }
                wordListUl.appendChild(li);
            });
        }

        let isSelecting = false;
        let touchTimer = null;
        const HOLD_DELAY = 100; // Milliseconds to detect a hold

        function startSelection(e) {
            isSelecting = true;
            selectedCells = [];
            toggleCellSelection(e.target);
            document.addEventListener('mousemove', handleMouseMove);
            document.addEventListener('mouseup', endSelection);
        }

        function handleTouchStart(e) {
            e.preventDefault(); // Prevent default touch scrolling
            touchTimer = setTimeout(() => {
                isSelecting = true;
                selectedCells = [];
                toggleCellSelection(e.target);
                document.addEventListener('touchmove', handleTouchMove, { passive: false });
                document.addEventListener('touchend', endSelection);
            }, HOLD_DELAY);
        }

        function handleTouchMove(e) {
            if (!isSelecting) return;
            e.preventDefault(); // Prevent default touch scrolling
            const touch = e.touches[0];
            const element = document.elementFromPoint(touch.clientX, touch.clientY);
            if (element && element.classList.contains('cell')) {
                const lastCell = selectedCells[selectedCells.length - 1];
                const currentCell = element;
                const lastRow = parseInt(lastCell.dataset.row);
                const lastCol = parseInt(lastCell.dataset.col);
                const currRow = parseInt(currentCell.dataset.row);
                const currCol = parseInt(currentCell.dataset.col);
                const dr = currRow - lastRow;
                const dc = currCol - lastCol;
                const isValidDirection = Math.abs(dr) <= 1 && Math.abs(dc) <= 1 && (dr !== 0 || dc !== 0);
                if (isValidDirection && selectedCells.length < 10) {
                    toggleCellSelection(currentCell);
                    drawLine(selectedCells, 'select');
                }
            }
        }

        function handleMouseMove(e) {
            if (!isSelecting) return;
            if (e.target.classList.contains('cell')) {
                const lastCell = selectedCells[selectedCells.length - 1];
                const currentCell = e.target;
                const lastRow = parseInt(lastCell.dataset.row);
                const lastCol = parseInt(lastCell.dataset.col);
                const currRow = parseInt(currentCell.dataset.row);
                const currCol = parseInt(currentCell.dataset.col);
                const dr = currRow - lastRow;
                const dc = currCol - lastCol;
                const isValidDirection = Math.abs(dr) <= 1 && Math.abs(dc) <= 1 && (dr !== 0 || dc !== 0);
                if (isValidDirection && selectedCells.length < 10) {
                    toggleCellSelection(currentCell);
                    drawLine(selectedCells, 'select');
                }
            }
        }

        function endSelection() {
            if (touchTimer) {
                clearTimeout(touchTimer);
                touchTimer = null;
            }
            isSelecting = false;
            document.removeEventListener('mousemove', handleMouseMove);
            document.removeEventListener('mouseup', endSelection);
            document.removeEventListener('touchmove', handleTouchMove);
            document.removeEventListener('touchend', endSelection);
            checkSelectedWord();
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            selectedCells.forEach(cell => cell.classList.remove('selected'));
            selectedCells = [];
        }

        function toggleCellSelection(cell) {
            if (!selectedCells.includes(cell) && cell.classList.contains('cell')) {
                selectedCells.push(cell);
                cell.classList.add('selected');
            }
        }

        function checkSelectedWord() {
            if (selectedCells.length === 0) return;
            const word = selectedCells.map(cell => cell.textContent).join('');
            const reverseWord = word.split('').reverse().join('');
            if (placedWords.includes(word) && !foundWords.includes(word)) {
                foundWords.push(word);
                selectedCells.forEach(cell => cell.classList.add('found'));
                drawLine(selectedCells, 'found');
                setTimeout(() => ctx.clearRect(0, 0, canvas.width, canvas.height), 1000);
                renderWordList();
                playSound('success');
                if (foundWords.length === placedWords.length) {
                    document.getElementById('message').textContent = 'Congratulations! You found all words!';
                    document.getElementById('game-container').classList.add('win-animation');
                    startConfetti();
                }
            } else if (placedWords.includes(reverseWord) && !foundWords.includes(reverseWord)) {
                foundWords.push(reverseWord);
                selectedCells.forEach(cell => cell.classList.add('found'));
                drawLine(selectedCells, 'found');
                setTimeout(() => ctx.clearRect(0, 0, canvas.width, canvas.height), 1000);
                renderWordList();
                playSound('success');
                if (foundWords.length === placedWords.length) {
                    document.getElementById('message').textContent = 'Congratulations! You found all words!';
                    document.getElementById('game-container').classList.add('win-animation');
                    startConfetti();
                }
            } else {
                document.getElementById('message').textContent = 'Not a valid word!';
                playSound('error');
                setTimeout(() => {
                    document.getElementById('message').textContent = '';
                }, 1000);
            }
        }

        function setMode(mode) {
            currentMode = mode;
            document.getElementById('mode-selection').style.display = 'none';
            document.getElementById('grid-container').style.display = 'block';
            document.getElementById('word-list').style.display = 'block';
            document.getElementById('restart-button').style.display = 'inline-block';
            startGame();
        }

        function showInfoModal() {
            document.getElementById('info-modal').style.display = 'block';
        }

        function startGame() {
            GRID_SIZE = modeSettings[currentMode].size;
            document.getElementById('game-title').textContent = `Word Search (${modeSettings[currentMode].name})`;
            foundWords = [];
            document.getElementById('game-container').classList.remove('win-animation');
            generateGrid();
            renderGrid();
            renderWordList();
            document.getElementById('message').textContent = '';
        }

        document.getElementById('restart-button').addEventListener('click', () => {
            document.getElementById('mode-selection').style.display = 'flex';
            document.getElementById('grid-container').style.display = 'none';
            document.getElementById('word-list').style.display = 'none';
            document.getElementById('restart-button').style.display = 'none';
        });
        document.getElementById('info-button').addEventListener('click', showInfoModal);
        document.getElementById('close-info').addEventListener('click', () => {
            document.getElementById('info-modal').style.display = 'none';
        });

        // Show mode selection on initial load
        document.getElementById('mode-selection').style.display = 'flex';
    </script>
</body>
</html>
