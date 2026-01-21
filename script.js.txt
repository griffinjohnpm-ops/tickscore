// --- Configuration & State ---
let config = {
    stocks: localStorage.getItem('tickr_stocks') || 'AAPL, MSFT, GOOGL',
    sports: localStorage.getItem('tickr_sports') || 'NY Knicks, Georgia Bulldogs',
    zip: localStorage.getItem('tickr_zip') || '10001'
};

// --- Initialization ---
document.addEventListener('DOMContentLoaded', () => {
    // Setup event listeners
    document.getElementById('btn-settings').onclick = () => toggleModal(true);
    document.getElementById('btn-close').onclick = () => toggleModal(false);
    document.getElementById('btn-save').onclick = saveAndRender;
    document.getElementById('btn-fullscreen').onclick = toggleFullScreen;

    // Load saved values into inputs
    document.getElementById('set-stocks').value = config.stocks;
    document.getElementById('set-sports').value = config.sports;
    document.getElementById('set-zip').value = config.zip;

    // Initial Render
    renderTicker();
});

// --- Main Rendering Function ---
async function renderTicker() {
    const track = document.getElementById('scroll-track');
    // Temporarily stop animation to avoid glitches while updating
    track.style.animation = 'none';
    track.offsetHeight; // Trigger reflow
    track.innerHTML = ''; // Clear existing content

    // 1. Generate Stock Modules
    const stockSymbols = config.stocks.split(',').map(s => s.trim().toUpperCase());
    for (const symbol of stockSymbols) {
        if (symbol) {
            const data = await getMockStockData(symbol);
            track.appendChild(createStockModule(data));
        }
    }

    // 2. Generate Sports Modules
    const teams = config.sports.split(',').map(s => s.trim());
    // Group into matchups for demo purposes (2 teams per module)
    for (let i = 0; i < teams.length; i += 2) {
        const team1 = teams[i];
        const team2 = teams[i+1] || 'OPPONENT'; // Handle odd numbers
        if (team1) {
            const data = await getMockSportsData(team1, team2);
            track.appendChild(createSportsModule(data));
        }
    }
    
    // Duplicate content to ensure a smooth, infinite-looking loop
    track.innerHTML += track.innerHTML;

    // Restart animation
    setTimeout(() => {
        track.style.animation = '';
    }, 100);
}

// --- Module Creation Helpers (HTML Generation) ---

function createStockModule(data) {
    const mod = document.createElement('div');
    mod.className = 'module stock-module';
    const colorClass = data.isUp ? 'led-green' : 'led-red';
    const arrow = data.isUp ? '▲' : '▼';
    // Determine icon based on symbol for demo
    let iconId = '#icon-microsoft';
    if (data.symbol === 'AAPL') iconId = '#icon-apple';

    mod.innerHTML = `
        <div class="stock-logo-container">
            <svg class="stock-icon" style="fill:${data.iconColor}">
                <use href="${iconId}"></use>
            </svg>
        </div>
        <div class="stock-data">
            <div class="stock-symbol">${data.symbol}</div>
            <div class="stock-change ${colorClass}">${arrow} ${data.changePct}</div>
            <div class="stock-price">$${data.price}</div>
        </div>
    `;
    return mod;
}

function createSportsModule(data) {
    const mod = document.createElement('div');
    mod.className = 'module sports-module';
    mod.innerHTML = `
        <div class="matchup">
            <div class="team">
                <svg class="team-logo"><use href="#icon-${data.sportType}"></use></svg>
                <div class="team-name">${data.team1.code}</div>
            </div>
            <div class="vs">VS</div>
            <div class="team">
                <svg class="team-logo"><use href="#icon-${data.sportType}"></use></svg>
                <div class="team-name">${data.team2.code}</div>
            </div>
        </div>
        <div class="scores">
            <div class="score-line"><span class="led-red">${data.team1.score}</span></div>
            <div class="score-line"><span class="led-white">${data.team2.score}</span></div>
        </div>
        <div class="game-status">
            <div>${data.status}</div>
            <div>${data.time}</div>
        </div>
    `;
    return mod;
}

// --- Mock Data Generators (Replace with real APIs later) ---
// These functions generate data structured exactly like the image props.

function getMockStockData(symbol) {
    // Simulate API call delay
    return new Promise(resolve => {
        const isUp = Math.random() > 0.5;
        const price = (Math.random() * 300 + 100).toFixed(2);
        const change = (Math.random() * 5).toFixed(2);
        resolve({
            symbol: symbol,
            price: price,
            changePct: change + '%',
            isUp: isUp,
            iconColor: symbol === 'AAPL' ? '#ff3333' : '#00f3ff' // Match image colors
        });
    });
}

function getMockSportsData(team1Name, team2Name) {
    return new Promise(resolve => {
        //Simple logic to guess sport type for icon
        const type = team1Name.toLowerCase().includes('knicks') ? 'bball' : 'football';
        resolve({
            sportType: type,
            team1: { code: team1Name.substring(0,3).toUpperCase(), score: Math.floor(Math.random() * 110) },
            team2: { code: team2Name.substring(0,3).toUpperCase(), score: Math.floor(Math.random() * 110) },
            status: '4TH QTR',
            time: '2:45 REM'
        });
    });
}

// --- UI Utility Functions ---
function toggleModal(show) {
    const modal = document.getElementById('settings-modal');
    modal.classList.toggle('hidden', !show);
}

function saveAndRender() {
    // Save inputs to localStorage
    config.stocks = document.getElementById('set-stocks').value;
    config.sports = document.getElementById('set-sports').value;
    config.zip = document.getElementById('set-zip').value;
    
    localStorage.setItem('tickr_stocks', config.stocks);
    localStorage.setItem('tickr_sports', config.sports);
    localStorage.setItem('tickr_zip', config.zip);

    toggleModal(false);
    renderTicker();
}

function toggleFullScreen() {
    if (!document.fullscreenElement) {
        document.documentElement.requestFullscreen();
    } else {
        if (document.exitFullscreen) document.exitFullscreen();
    }
}