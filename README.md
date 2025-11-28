<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Planet Crafter Web</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        
        body {
            background-color: #0a1a33;
            color: #eee;
            overflow: hidden;
            height: 100vh;
            display: flex;
            flex-direction: column;
        }
        
        #game-container {
            position: relative;
            flex: 1;
            overflow: hidden;
            background-color: #0a1a33;
        }
        
        #game-canvas {
            display: block;
            background-color: #0a1a33;
            position: absolute;
            top: 0;
            left: 0;
        }
        
        #ui-panel {
            background-color: rgba(15, 25, 45, 0.9);
            padding: 15px;
            display: flex;
            justify-content: space-between;
            border-top: 1px solid #333;
        }
        
        #resource-panel, #planet-stats, #building-panel {
            background-color: rgba(25, 35, 60, 0.8);
            border-radius: 8px;
            padding: 10px;
            min-width: 200px;
            border: 1px solid #333;
        }
        
        .resource-item {
            display: flex;
            justify-content: space-between;
            margin: 5px 0;
            padding: 5px;
            border-radius: 4px;
        }
        
        .resource-item.positive {
            background-color: rgba(0, 200, 100, 0.2);
        }
        
        .resource-item.negative {
            background-color: rgba(200, 50, 50, 0.2);
        }
        
        .resource-name {
            font-weight: bold;
            color: #ccc;
        }
        
        .resource-value {
            color: #fff;
        }
        
        .resource-value.negative {
            color: #ff6b6b;
        }
        
        .resource-bar {
            height: 10px;
            background-color: #333;
            border-radius: 5px;
            margin: 5px 0;
            overflow: hidden;
        }
        
        .resource-bar-fill {
            height: 100%;
            background-color: #4CAF50;
            transition: width 0.5s ease;
        }
        
        .planet-stat {
            margin: 8px 0;
            padding: 8px;
            border-radius: 4px;
            background-color: rgba(25, 35, 60, 0.6);
        }
        
        .planet-stat-label {
            display: inline-block;
            width: 100px;
            font-weight: bold;
            color: #ccc;
        }
        
        .planet-stat-value {
            display: inline-block;
            color: #fff;
        }
        
        .planet-stat-value.temp-heat {
            color: #ff6b6b;
        }
        
        .planet-stat-value.temp-cold {
            color: #6b8cff;
        }
        
        .building-option {
            display: flex;
            flex-direction: column;
            align-items: center;
            cursor: pointer;
            padding: 10px;
            margin: 5px;
            border-radius: 8px;
            background-color: rgba(25, 35, 60, 0.7);
            border: 1px solid #444;
            transition: all 0.3s ease;
            width: 120px;
            text-align: center;
        }
        
        .building-option:hover {
            background-color: rgba(40, 55, 85, 0.9);
            transform: translateY(-2px);
        }
        
        .building-option.selected {
            border: 2px solid #ffcc00;
            box-shadow: 0 0 10px rgba(255, 204, 0, 0.5);
        }
        
        .building-icon {
            width: 40px;
            height: 40px;
            margin-bottom: 5px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 20px;
            color: white;
        }
        
        .building-name {
            font-size: 12px;
            font-weight: bold;
            margin-bottom: 5px;
        }
        
        .building-cost {
            font-size: 10px;
            color: #aaa;
        }
        
        .building-cost span {
            margin-right: 5px;
        }
        
        .building-cost .positive {
            color: #4CAF50;
        }
        
        .building-cost .negative {
            color: #ff6b6b;
        }
        
        #controls {
            display: flex;
            justify-content: center;
            margin-top: 10px;
            gap: 15px;
        }
        
        .control-btn {
            background-color: #2c3e50;
            color: white;
            border: none;
            padding: 8px 15px;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        
        .control-btn:hover {
            background-color: #34495e;
        }
        
        .game-info {
            position: absolute;
            top: 10px;
            left: 10px;
            background-color: rgba(0, 0, 0, 0.7);
            padding: 10px;
            border-radius: 5px;
            font-size: 14px;
            color: #eee;
            pointer-events: none;
        }
        
        .sun-moon {
            position: absolute;
            width: 50px;
            height: 50px;
            border-radius: 50%;
            pointer-events: none;
            z-index: 10;
        }
        
        .sun {
            background-color: #FFD700;
            box-shadow: 0 0 30px #FFD700;
        }
        
        .moon {
            background-color: #E6E6E6;
            box-shadow: 0 0 20px #E6E6E6;
        }
        
        #loading-screen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: #0a1a33;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 100;
            color: white;
        }
        
        #loading-text {
            font-size: 24px;
            margin-top: 20px;
        }
        
        .progress-bar {
            width: 300px;
            height: 10px;
            background-color: #333;
            margin-top: 20px;
            border-radius: 5px;
            overflow: hidden;
        }
        
        .progress-fill {
            height: 100%;
            width: 0%;
            background-color: #4CAF50;
            transition: width 0.3s;
        }
    </style>
</head>
<body>
    <div id="loading-screen">
        <h1>PLANET CRAFTER</h1>
        <div class="progress-bar">
            <div class="progress-fill" id="progress-fill"></div>
        </div>
        <div id="loading-text">Initializing terraforming equipment...</div>
    </div>

    <div id="game-container">
        <canvas id="game-canvas"></canvas>
        
        <div class="sun-moon" id="sun-moon"></div>
        
        <div class="game-info" id="game-info">
            Click on building types at the bottom to select<br>
            Click on terrain to place buildings<br>
            Click on resource nodes to collect them<br>
            SPACE: Pause/Resume | UP/DOWN: Adjust speed
        </div>
    </div>

    <div id="ui-panel">
        <div id="resource-panel">
            <h3>Resources</h3>
            <div class="resource-item" id="resource-iron">
                <span class="resource-name">Iron</span>
                <span class="resource-value" id="resource-iron-value">0</span>
            </div>
            <div class="resource-item" id="resource-copper">
                <span class="resource-name">Copper</span>
                <span class="resource-value" id="resource-copper-value">0</span>
            </div>
            <div class="resource-item" id="resource-stone">
                <span class="resource-name">Stone</span>
                <span class="resource-value" id="resource-stone-value">0</span>
            </div>
            <div class="resource-item" id="resource-water">
                <span class="resource-name">Water</span>
                <span class="resource-value" id="resource-water-value">0</span>
            </div>
            <div class="resource-item" id="resource-energy">
                <span class="resource-name">Energy</span>
                <span class="resource-value" id="resource-energy-value">0</span>
            </div>
            <div class="resource-item" id="resource-plants">
                <span class="resource-name">Plants</span>
                <span class="resource-value" id="resource-plants-value">0</span>
            </div>
        </div>

        <div id="planet-stats">
            <h3>Planet Status</h3>
            <div class="planet-stat">
                <span class="planet-stat-label">Atmosphere</span>
                <span class="planet-stat-value" id="planet-atmosphere">0.00</span>
                <div class="resource-bar">
                    <div class="resource-bar-fill" id="atmosphere-bar" style="width: 0%"></div>
                </div>
            </div>
            <div class="planet-stat">
                <span class="planet-stat-label">Temperature</span>
                <span class="planet-stat-value temp-cold" id="planet-temperature">-30.0°C</span>
                <div class="resource-bar">
                    <div class="resource-bar-fill" id="temperature-bar" style="width: 0%"></div>
                </div>
            </div>
            <div class="planet-stat">
                <span class="planet-stat-label">Oxygen</span>
                <span class="planet-stat-value" id="planet-oxygen">0.00</span>
                <div class="resource-bar">
                    <div class="resource-bar-fill" id="oxygen-bar" style="width: 0%"></div>
                </div>
            </div>
        </div>

        <div id="building-panel">
            <h3>Buildings</h3>
            <div class="building-option" id="building-solar-panel">
                <div class="building-icon">☀️</div>
                <div class="building-name">Solar Panel</div>
                <div class="building-cost">
                    <span>Iron: <span class="positive">10</span></span>
                    <span>Copper: <span class="positive">5</span></span>
                </div>
            </div>
            <div class="building-option" id="building-mining-station">
                <div class="building-icon">⛏️</div>
                <div class="building-name">Mining Station</div>
                <div class="building-cost">
                    <span>Iron: <span class="positive">20</span></span>
                    <span>Stone: <span class="positive">10</span></span>
                </div>
            </div>
            <div class="building-option" id="building-greenhouse">
                <div class="building-icon">🌿</div>
                <div class="building-name">Greenhouse</div>
                <div class="building-cost">
                    <span>Iron: <span class="positive">15</span></span>
                    <span>Glass: <span class="positive">10</span></span>
                </div>
            </div>
            <div class="building-option" id="building-atmosphere-processor">
                <div class="building-icon">💨</div>
                <div class="building-name">Atmosphere Processor</div>
                <div class="building-cost">
                    <span>Iron: <span class="positive">30</span></span>
                    <span>Copper: <span class="positive">20</span></span>
                    <span>Stone: <span class="positive">15</span></span>
                </div>
            </div>
        </div>
    </div>

    <div id="controls">
        <button class="control-btn" id="pause-btn">Pause</button>
        <button class="control-btn" id="speed-up-btn">Speed Up</button>
        <button class="control-btn" id="speed-down-btn">Slow Down</button>
        <button class="control-btn" id="reset-btn">Reset</button>
    </div>

    <script>
        // Game state
        const gameState = {
            // Planet parameters
            atmosphere: 0.2,
            temperature: -30,
            oxygen: 0.1,
            dayTime: 0,
            dayLength: 2000,
            gameSpeed: 1,
            paused: false,
            
            // Resources
            resources: {
                iron: 100,
                copper: 50,
                stone: 150,
                water: 20,
                energy: 100,
                plants: 0
            },
            
            // Buildings
            buildings: [],
            selectedBuilding: null,
            
            // Terrain
            terrain: [],
            
            // Surface resources
            surfaceResources: [],
            
            // Particles
            particles: [],
            
            // Game elements
            canvas: null,
            ctx: null,
            sunMoon: null,
            
            // UI elements
            uiElements: {}
        };

        // Building types definition
        const buildingTypes = {
            "solar-panel": {
                cost: { iron: 10, copper: 5 },
                energy: 5,
                icon: "☀️",
                name: "Solar Panel",
                description: "Generates energy from sunlight"
            },
            "mining-station": {
                cost: { iron: 20, stone: 10 },
                energy: -2,
                icon: "⛏️",
                name: "Mining Station",
                description: "Extracts resources from the ground"
            },
            "greenhouse": {
                cost: { iron: 15, glass: 10 },
                energy: -3,
                oxygen: 0.01,
                plants: 0.005,
                icon: "🌿",
                name: "Greenhouse",
                description: "Produces oxygen and plants"
            },
            "atmosphere-processor": {
                cost: { iron: 30, copper: 20, stone: 15 },
                energy: -5,
                atmosphere: 0.02,
                icon: "💨",
                name: "Atmosphere Processor",
                description: "Increases atmospheric pressure"
            }
        };

        // Initialize the game
        function initGame() {
            // Get the canvas and context
            gameState.canvas = document.getElementById('game-canvas');
            gameState.ctx = gameState.canvas.getContext('2d');
            
            // Set canvas size
            resizeCanvas();
            window.addEventListener('resize', resizeCanvas);
            
            // Get UI elements
            gameState.uiElements = {
                resourcePanel: document.getElementById('resource-panel'),
                planetStats: document.getElementById('planet-stats'),
                buildingPanel: document.getElementById('building-panel'),
                sunMoon: document.getElementById('sun-moon'),
                gameInfo: document.getElementById('game-info'),
                pauseBtn: document.getElementById('pause-btn'),
                speedUpBtn: document.getElementById('speed-up-btn'),
                speedDownBtn: document.getElementById('speed-down-btn'),
                resetBtn: document.getElementById('reset-btn')
            };
            
            // Initialize terrain
            for (let i = 0; i < 50; i++) {
                gameState.terrain.push(300 + Math.random() * 100);
            }
            
            // Initialize surface resources
            for (let i = 0; i < 30; i++) {
                const x = Math.random() * gameState.canvas.width;
                const resourceType = ['iron', 'copper', 'stone', 'water'][Math.floor(Math.random() * 4)];
                const amount = 5 + Math.floor(Math.random() * 15);
                gameState.surfaceResources.push({
                    x: x,
                    type: resourceType,
                    amount: amount,
                    size: 8 + Math.random() * 5,
                    collected: false
                });
            }
            
            // Initialize UI
            updateUI();
            
            // Set up event listeners
            setupEventListeners();
            
            // Hide loading screen
            document.getElementById('loading-screen').style.display = 'none';
            
            // Start game loop
            requestAnimationFrame(gameLoop);
        }

        // Resize canvas to fit window
        function resizeCanvas() {
            gameState.canvas.width = window.innerWidth;
            gameState.canvas.height = window.innerHeight - 150; // Account for UI
            gameState.sunMoon.style.width = '50px';
            gameState.sunMoon.style.height = '50px';
        }

        // Setup event listeners
        function setupEventListeners() {
            // Building selection
            Object.keys(buildingTypes).forEach(type => {
                const element = document.getElementById(`building-${type}`);
                element.addEventListener('click', () => {
                    gameState.selectedBuilding = type;
                    updateBuildingSelection();
                });
            });
            
            // UI buttons
            gameState.uiElements.pauseBtn.addEventListener('click', () => {
                gameState.paused = !gameState.paused;
                gameState.uiElements.pauseBtn.textContent = gameState.paused ? 'Play' : 'Pause';
            });
            
            gameState.uiElements.speedUpBtn.addEventListener('click', () => {
                gameState.gameSpeed = Math.min(5, gameState.gameSpeed + 0.5);
            });
            
            gameState.uiElements.speedDownBtn.addEventListener('click', () => {
                gameState.gameSpeed = Math.max(0.1, gameState.gameSpeed - 0.5);
            });
            
            gameState.uiElements.resetBtn.addEventListener('click', () => {
                if (confirm('Are you sure you want to reset the planet? All progress will be lost.')) {
                    resetGame();
                }
            });
            
            // Mouse events
            gameState.canvas.addEventListener('click', (e) => {
                const rect = gameState.canvas.getBoundingClientRect();
                const x = e.clientX - rect.left;
                const y = e.clientY - rect.top;
                
                if (gameState.selectedBuilding) {
                    // Try to place a building
                    placeBuilding(x, y);
                } else {
                    // Try to collect a resource
                    collectResource(x, y);
                }
            });
            
            // Keyboard controls
            document.addEventListener('keydown', (e) => {
                if (e.code === 'Space') {
                    e.preventDefault();
                    gameState.paused = !gameState.paused;
                    gameState.uiElements.pauseBtn.textContent = gameState.paused ? 'Play' : 'Pause';
                } else if (e.code === 'ArrowUp') {
                    gameState.gameSpeed = Math.min(5, gameState.gameSpeed + 0.5);
                } else if (e.code === 'ArrowDown') {
                    gameState.gameSpeed = Math.max(0.1, gameState.gameSpeed - 0.5);
                }
            });
        }

        // Update building selection UI
        function updateBuildingSelection() {
            // Remove selection from all buildings
            Object.keys(buildingTypes).forEach(type => {
                const element = document.getElementById(`building-${type}`);
                element.classList.remove('selected');
            });
            
            // Add selection to the selected building
            if (gameState.selectedBuilding) {
                const element = document.getElementById(`building-${gameState.selectedBuilding}`);
                element.classList.add('selected');
            }
        }

        // Place a building at coordinates
        function placeBuilding(x, y) {
            if (!gameState.selectedBuilding) return;
            
            const buildingType = buildingTypes[gameState.selectedBuilding];
            
            // Check if we have enough resources
            let canAfford = true;
            for (const resource in buildingType.cost) {
                if (gameState.resources[resource] < buildingType.cost[resource]) {
                    canAfford = false;
                    break;
                }
            }
            
            if (!canAfford) {
                showNotification("Not enough resources!", 2000);
                return;
            }
            
            // Check if placement is valid (on terrain)
            const terrainIndex = Math.floor(x / (gameState.canvas.width / gameState.terrain.length));
            if (terrainIndex < 0 || terrainIndex >= gameState.terrain.length) return;
            
            const terrainHeight = gameState.terrain[terrainIndex];
            const terrainY = gameState.canvas.height - terrainHeight;
            
            // Place building above terrain
            const buildingY = terrainY - 50;
            
            // Deduct resources
            for (const resource in buildingType.cost) {
                gameState.resources[resource] -= buildingType.cost[resource];
            }
            
            // Add building
            gameState.buildings.push({
                type: gameState.selectedBuilding,
                x: x,
                y: buildingY,
                created: Date.now()
            });
            
            // Add visual effect
            createParticles(x, buildingY, 10, 150, 150, 150, 3, 1);
            
            // Clear selection
            gameState.selectedBuilding = null;
            updateBuildingSelection();
            updateUI();
        }

        // Collect a resource
        function collectResource(x, y) {
            for (let i = 0; i < gameState.surfaceResources.length; i++) {
                const resource = gameState.surfaceResources[i];
                if (resource.collected) continue;
                
                const terrainIndex = Math.floor(resource.x / (gameState.canvas.width / gameState.terrain.length));
                if (terrainIndex < 0 || terrainIndex >= gameState.terrain.length) continue;
                
                const terrainHeight = gameState.terrain[terrainIndex];
                const resourceY = gameState.canvas.height - terrainHeight - 10;
                
                // Calculate distance
                const distance = Math.sqrt(
                    Math.pow(x - resource.x, 2) + 
                    Math.pow(y - resourceY, 2)
                );
                
                if (distance < resource.size + 15) {
                    // Collect resource
                    gameState.resources[resource.type] += resource.amount;
                    resource.collected = true;
                    
                    // Create visual effect
                    createParticles(resource.x, resourceY, 15, 255, 255, 0, 2, 1);
                    
                    // Add new resource
                    const newResourceType = ['iron', 'copper', 'stone', 'water'][Math.floor(Math.random() * 4)];
                    const newAmount = 5 + Math.floor(Math.random() * 15);
                    const newX = Math.random() * gameState.canvas.width;
                    
                    gameState.surfaceResources.push({
                        x: newX,
                        type: newResourceType,
                        amount: newAmount,
                        size: 8 + Math.random() * 5,
                        collected: false
                    });
                    
                    updateUI();
                    return true;
                }
            }
            return false;
        }

        // Create particle effects
        function createParticles(x, y, count, r, g, b, size, speed) {
            for (let i = 0; i < count; i++) {
                gameState.particles.push({
                    x: x,
                    y: y,
                    vx: (Math.random() - 0.5) * speed,
                    vy: (Math.random() - 0.5) * speed,
                    r: r,
                    g: g,
                    b: b,
                    size: size,
                    life: 30,
                    maxLife: 30
                });
            }
        }

        // Update game state
        function updateGameState() {
            if (gameState.paused) return;
            
            // Update day/night cycle
            gameState.dayTime = (gameState.dayTime + 0.1 * gameState.gameSpeed) % 100;
            
            // Calculate building effects
            let energyChange = 0;
            let atmosphereChange = 0;
            let oxygenChange = 0;
            let plantsChange = 0;
            
            for (const building of gameState.buildings) {
                const type = buildingTypes[building.type];
                if (type.energy !== undefined) energyChange += type.energy;
                if (type.atmosphere !== undefined) atmosphereChange += type.atmosphere;
                if (type.oxygen !== undefined) oxygenChange += type.oxygen;
                if (type.plants !== undefined) plantsChange += type.plants;
            }
            
            // Update resources
            gameState.resources.energy += energyChange * (gameState.gameSpeed / 2);
            gameState.atmosphere = Math.min(1.0, Math.max(0.0, gameState.atmosphere + atmosphereChange * 0.01 * (gameState.gameSpeed / 2)));
            gameState.oxygen = Math.min(1.0, Math.max(0.0, gameState.oxygen + oxygenChange * 0.01 * (gameState.gameSpeed / 2)));
            gameState.resources.plants += plantsChange * (gameState.gameSpeed / 2);
            
            // Update temperature based on atmosphere and oxygen
            gameState.temperature = -30 + (gameState.atmosphere * 50) + (gameState.oxygen * 20);
            
            // Update particles
            for (let i = gameState.particles.length - 1; i >= 0; i--) {
                const p = gameState.particles[i];
                p.x += p.vx;
                p.y += p.vy;
                p.life--;
                p.size *= 0.98;
                
                if (p.life <= 0) {
                    gameState.particles.splice(i, 1);
                }
            }
            
            // Add random particles for visual effect
            if (Math.random() < 0.05 * gameState.gameSpeed) {
                createParticles(
                    Math.random() * gameState.canvas.width,
                    Math.random() * (gameState.canvas.height / 3),
                    1,
                    Math.floor(Math.random() * 100) + 150,
                    Math.floor(Math.random() * 100) + 150,
                    Math.floor(Math.random() * 100) + 150,
                    2,
                    0.5
                );
            }
        }

        // Draw the game
        function drawGame() {
            const ctx = gameState.ctx;
            const canvas = gameState.canvas;
            
            // Clear canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Draw sky based on day/night cycle
            if (gameState.dayTime < 50 || gameState.dayTime > 95) {
                ctx.fillStyle = '#0a1a33';
            } else {
                ctx.fillStyle = '#1a3a66';
            }
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // Draw stars at night
            if (gameState.dayTime < 50 || gameState.dayTime > 95) {
                ctx.fillStyle = '#ffffff';
                for (let i = 0; i < 100; i++) {
                    const x = (i * 137.5) % canvas.width;
                    const y = (i * 73.2) % (canvas.height / 3);
                    const size = Math.random() * 1.5;
                    ctx.beginPath();
                    ctx.arc(x, y, size, 0, Math.PI * 2);
                    ctx.fill();
                }
            }
            
            // Draw sun/moon
            const sunX = (gameState.dayTime / 100) * canvas.width;
            const sunY = canvas.height / 4;
            
            gameState.sunMoon.style.display = 'block';
            gameState.sunMoon.style.left = `${sunX - 25}px`;
            gameState.sunMoon.style.top = `${sunY - 25}px`;
            
            if (gameState.dayTime < 50) {
                gameState.sunMoon.className = 'sun-moon moon';
            } else {
                gameState.sunMoon.className = 'sun-moon sun';
            }
            
            // Draw terrain
            const segmentWidth = canvas.width / gameState.terrain.length;
            ctx.strokeStyle = '#3a5a40';
            ctx.lineWidth = 3;
            ctx.beginPath();
            
            for (let i = 0; i < gameState.terrain.length; i++) {
                const x = i * segmentWidth;
                const y = canvas.height - gameState.terrain[i];
                
                if (i === 0) {
                    ctx.moveTo(x, y);
                } else {
                    ctx.lineTo(x, y);
                }
            }
            
            // Connect to bottom
            ctx.lineTo(canvas.width, canvas.height);
            ctx.lineTo(0, canvas.height);
            ctx.closePath();
            
            // Fill with grass color
            ctx.fillStyle = '#2a4a30';
            ctx.fill();
            
            // Draw surface resources
            for (const resource of gameState.surfaceResources) {
                if (resource.collected) continue;
                
                const terrainIndex = Math.floor(resource.x / segmentWidth);
                if (terrainIndex < 0 || terrainIndex >= gameState.terrain.length) continue;
                
                const terrainHeight = gameState.terrain[terrainIndex];
                const x = resource.x;
                const y = canvas.height - terrainHeight - 10;
                
                // Color based on resource type
                let color;
                switch (resource.type) {
                    case 'iron': color = '#888'; break;
                    case 'copper': color = '#daa520'; break;
                    case 'stone': color = '#8b4513'; break;
                    case 'water': color = '#4682b4'; break;
                    default: color = '#fff';
                }
                
                ctx.fillStyle = color;
                ctx.beginPath();
                ctx.arc(x, y, resource.size, 0, Math.PI * 2);
                ctx.fill();
                
                // Add shine effect
                ctx.fillStyle = 'rgba(255, 255, 255, 0.3)';
                ctx.beginPath();
                ctx.arc(x - resource.size/3, y - resource.size/3, resource.size/3, 0, Math.PI * 2);
                ctx.fill();
            }
            
            // Draw buildings
            for (const building of gameState.buildings) {
                const terrainIndex = Math.floor(building.x / segmentWidth);
                if (terrainIndex < 0 || terrainIndex >= gameState.terrain.length) continue;
                
                const terrainHeight = gameState.terrain[terrainIndex];
                const x = building.x;
                const y = canvas.height - terrainHeight - 40;
                
                // Building base
                ctx.fillStyle = '#8b4513';
                ctx.fillRect(x - 20, y, 40, 40);
                
                // Building body
                ctx.fillStyle = '#aaaaaa';
                ctx.fillRect(x - 15, y + 5, 30, 30);
                
                // Building icon
                ctx.font = '20px Arial';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillStyle = '#ffffff';
                ctx.fillText(buildingTypes[building.type].icon, x, y + 20);
            }
            
            // Draw particles
            for (const particle of gameState.particles) {
                ctx.fillStyle = `rgba(${particle.r}, ${particle.g}, ${particle.b}, ${particle.life / particle.maxLife})`;
                ctx.beginPath();
                ctx.arc(particle.x, particle.y, particle.size, 0, Math.PI * 2);
                ctx.fill();
            }
        }

        // Update UI elements
        function updateUI() {
            // Update resource values
            for (const resource in gameState.resources) {
                const valueElement = document.getElementById(`resource-${resource}-value`);
                valueElement.textContent = Math.floor(gameState.resources[resource]);
                
                // Update color based on value
                const itemElement = document.getElementById(`resource-${resource}`);
                if (resource === 'energy' && gameState.resources.energy < 0) {
                    itemElement.className = 'resource-item negative';
                } else {
                    itemElement.className = 'resource-item positive';
                }
            }
            
            // Update planet stats
            document.getElementById('planet-atmosphere').textContent = gameState.atmosphere.toFixed(2);
            document.getElementById('atmosphere-bar').style.width = `${gameState.atmosphere * 100}%`;
            
            document.getElementById('planet-oxygen').textContent = gameState.oxygen.toFixed(2);
            document.getElementById('oxygen-bar').style.width = `${gameState.oxygen * 100}%`;
            
            const tempElement = document.getElementById('planet-temperature');
            tempElement.textContent = gameState.temperature.toFixed(1) + '°C';
            tempElement.className = gameState.temperature > 0 ? 'planet-stat-value temp-heat' : 'planet-stat-value temp-cold';
            
            // Temperature bar
            const tempPercent = Math.max(0, Math.min(100, (gameState.temperature + 30) / 80 * 100));
            document.getElementById('temperature-bar').style.width = `${tempPercent}%`;
            
            // Update day/night indicator
            document.getElementById('game-info').innerHTML = `
                Click on building types at the bottom to select<br>
                Click on terrain to place buildings<br>
                Click on resource nodes to collect them<br>
                SPACE: Pause/Resume | UP/DOWN: Adjust speed<br>
                Day: ${gameState.dayTime.toFixed(1)}% | Speed: ${gameState.gameSpeed.toFixed(1)}x
            `;
        }

        // Reset game
        function resetGame() {
            gameState.atmosphere = 0.2;
            gameState.temperature = -30;
            gameState.oxygen = 0.1;
            gameState.dayTime = 0;
            gameState.gameSpeed = 1;
            gameState.paused = false;
            gameState.uiElements.pauseBtn.textContent = 'Pause';
            
            gameState.resources = {
                iron: 100,
                copper: 50,
                stone: 150,
                water: 20,
                energy: 100,
                plants: 0
            };
            
            gameState.buildings = [];
            gameState.selectedBuilding = null;
            
            // Reset terrain
            gameState.terrain = [];
            for (let i = 0; i < 50; i++) {
                gameState.terrain.push(300 + Math.random() * 100);
            }
            
            // Reset surface resources
            gameState.surfaceResources = [];
            for (let i = 0; i < 30; i++) {
                const x = Math.random() * gameState.canvas.width;
                const resourceType = ['iron', 'copper', 'stone', 'water'][Math.floor(Math.random() * 4)];
                const amount = 5 + Math.floor(Math.random() * 15);
                gameState.surfaceResources.push({
                    x: x,
                    type: resourceType,
                    amount: amount,
                    size: 8 + Math.random() * 5,
                    collected: false
                });
            }
            
            gameState.particles = [];
            updateUI();
            updateBuildingSelection();
        }

        // Show notification
        function showNotification(message, duration = 3000) {
            const notification = document.createElement('div');
            notification.textContent = message;
            notification.style.position = 'absolute';
            notification.style.top = '50%';
            notification.style.left = '50%';
            notification.style.transform = 'translate(-50%, -50%)';
            notification.style.backgroundColor = 'rgba(0, 0, 0, 0.8)';
            notification.style.color = 'white';
            notification.style.padding = '10px 20px';
            notification.style.borderRadius = '5px';
            notification.style.zIndex = '100';
            notification.style.fontSize = '18px';
            notification.style.pointerEvents = 'none';
            notification.style.transition = 'opacity 0.5s';
            
            document.body.appendChild(notification);
            
            setTimeout(() => {
                notification.style.opacity = '0';
                setTimeout(() => {
                    document.body.removeChild(notification);
                }, 500);
            }, duration);
        }

        // Game loop
        function gameLoop() {
            updateGameState();
            drawGame();
            updateUI();
            requestAnimationFrame(gameLoop);
        }

        // Initialize the game when page loads
        window.addEventListener('load', () => {
            // Simulate loading
            let progress = 0;
            const interval = setInterval(() => {
                progress += 1;
                document.getElementById('progress-fill').style.width = `${progress}%`;
                document.getElementById('loading-text').textContent = `Initializing terraforming equipment... ${progress}%`;
                
                if (progress >= 100) {
                    clearInterval(interval);
                    setTimeout(initGame, 500); // Add a small delay to ensure loading is complete
                }
            }, 50);
        });
    </script>
</body>
</html>
