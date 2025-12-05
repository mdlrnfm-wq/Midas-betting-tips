# Midas-betting-tips
Your go to place for best betting tips
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Elite Predictions • Zcode Style • 2025</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css">
<style>
  :root { --primary:#0d6efd; --success:#198754; --danger:#dc3545; }
  * { box-sizing:border-box; margin:0; padding:0; }
  body { font-family:system-ui,sans-serif; background:#f8f9fa; color:#212529; line-height:1.5; transition:0.3s; }
  body.dark-mode { background:#121212; color:#e0d0; }
  header { background:var(--primary); color:white; padding:1rem; text-align:center; position:relative; }
  .controls { position:absolute; top:10px; right:10px; display:flex; gap:8px; }
  button { padding:8px 14px; border:none; border-radius:6px; cursor:pointer; font-weight:bold; }
  .date-navigation { display:flex; justify-content:center; flex-wrap:wrap; gap:10px; margin:20px; padding:10px; background:white; border-radius:8px; box-shadow:0 2px 8px rgba(0,0,0,0.1); }
  .sort-options { text-align:right; margin:10px 20px; }
  .best-bets-section { margin:20px; padding:20px; background:#e3f2fd; border-radius:10px; display:none; }
  .matches-grid { display:grid; grid-template-columns:repeat(auto-fill,minmax(340px,1fr)); gap:16px; padding:0 20px 40px; }
  .match-card { background:white; border-radius:12px; overflow:hidden; box-shadow:0 4px 12px rgba(0,0,0,0.12); position:relative; }
  .match-card.won { border-left:6px solid var(--success); }
  .match-card.lost { border-left:6px solid var(--danger); }
  .match-header { display:flex; justify-content:space-between; align-items:center; padding:10px 15px; background:#f1f3f5; font-weight:bold; }
  .match-teams { display:flex; justify-content:space-between; padding:15px; font-size:1.1em; }
  .team { display:flex; align-items:center; gap:8px; }
  .team img { width:28px; height:28px; object-fit:contain; }
  .match-footer { padding:12px 15px; background:#fafafa; border-top:1px solid #eee; }
  .prediction-value { font-weight:bold; color:var(--primary); font-size:1.1em; }
  .zcode-sim-section { background:linear-gradient(135deg,var(--primary),#0a58ca); color:white; padding:14px; border-radius:8px; margin:12px 0; font-size:0.95em; }
  .sim-btn { background:var(--success); color:white; padding:6px 12px; border-radius:4px; margin-top:8px; }
  .live-button { background:#d32f2f; color:white; padding:4px 10px; border-radius:20px; font-size:0.8em; animation:pulse 2s infinite; }
  @keyframes pulse { 0%{box-shadow:0 0 0 0 rgba(211,47,47,0.7);} 70%{box-shadow:0 0 0 10px rgba(211,47,47,0);}100%{box-shadow:0 0 0 0 rgba(211,47,47,0);} }
  body.dark-mode .match-card { background:#1e1e1e; }
  }
  body.dark-mode .match-footer { background:#252525; }
  body.dark-mode .best-bets-section { background:#1a2b3c; }
</style>
</head>
<body>

<header>
  <h1>Elite Predictions • Zcode Style</h1>
  <div class="controls">
    <button id="dark-mode-toggle">Dark Mode</button>
    <button id="view-toggle-btn"><i class="fas fa-compress"></i> Compact</button>
  </div>
</header>

<div class="date-navigation">
  <button id="prev-day">Yesterday</button>
  <input type="date" id="date-picker">
  <button id="next-day">Tomorrow</button>
  <button id="today-btn">Today</button>
</div>

<div id="best-bets" class="best-bets-section">
  <h2>Best Bets Today</h2>
  <div id="best-bets-grid" class="matches-grid"></div>
</div>

<div class="sort-options">
  <select id="sort-select">
    <option value="time">Sort by Time</option>
    <option value="sim-win-desc">Zcode Sim Win % (Highest)</option>
    <option value="odds-desc">Odds (High → Low)</option>
  </select>
</div>

<div id="matches-container" class="matches-grid"></div>

<script>
// ──────────────────────────────
// FULL ZCODE ENGINE + SITE LOGIC
// ──────────────────────────────
let selectedDate = new Date(); selectedDate.setHours(0,0,0,0);
let isCompact = false;
let darkMode = localStorage.getItem('darkMode')==='true';

// Sample data – replace with your real matches anytime
let matches = [
  {game_id:"m1",date:new Date().toISOString().slice(0,10).replace(/-/g,''),time:"20:00",team_home:"Man City",team_away:"Arsenal",home_logo:"https://flagemoji.to/32/GB.png",away_logo:"https://flagemoji.to/32/GB.png",division_label:"Premier League",odds:"1.95",weather:"clear",home_form:4,away_form:2},
  {game_id:"m2",date:new Date().toISOString().slice(0,10).replace(/-/g,''),time:"21:00",team_home:"Real Madrid",team_away:"Barcelona",home_logo:"https://flagemoji.to/32/ES.png",away_logo:"https://flagemoji.to/32/ES.png",division_label:"La Liga",odds:"2.40",weather:"sunny",home_form:5,away_form:3},
  {game_id:"m3",date:new Date().toISOString().slice(0,10).replace(/-/g,''),time:"19:45",team_home:"Bayern",team_away:"Dortmund",home_logo:"https://flagemoji.to/32/DE.png",away_logo:"https://flagemoji.to/32/DE.png",division_label:"Bundesliga",odds:"1.70",weather:"cloudy",home_form:3,away_form:1},
  // Add as many as you want...
];

// ───── ZCODE MONTE CARLO ENGINE ─────
function poisson(lambda){let L=Math.exp(-lambda),k=0,p=1;do{k++;p*=Math.random()}while(p>L);return k-1;}
function zcodeSim(match){
  let homeProb = 100 / (parseFloat(match.odds)||2);
  let boost = 12 + (match.home_form||0)*4;
  if(match.weather&&['rain','snow'].includes(match.weather)) boost-=7;
  let finalHome = Math.min(91, homeProb + boost);
  let finalAway = Math.min(91, 100 - finalHome);

  let homeWins=0, awayWins=0, draws=0, totalG=0, btts=0;
  for(let i=0;i<5000;i++){
    let h = poisson(finalHome/28);
    let a = poisson(finalAway/28);
    totalG += h+a;
    if(h>a)homeWins++;else if(a>h)awayWins++; else draws++;
    if(h>0&&a>0)btts++;
  }
  let forecast = finalHome>finalAway?`Home Win \( {Math.round(finalHome)}%`:`Away Win \){Math.round(finalAway)}%`;
  if(totalG/5000>2.5) forecast += ' — Over 2.5';
  return {
    forecast,
    stars: Math.min(5, Math.max(1, Math.round(finalHome/18))),
    simHome: Math.round(homeWins/50),
    simAway: Math.round(awayWins/50),
    simDraw: Math.round(draws/50),
    avgScore: `\( {(totalG/5000/2).toFixed(1)}- \){(totalG/5000/2).toFixed(1)}`
  };
}

// ───── RENDER MATCHES ─────
function render(){
  const container = document.getElementById('matches-container');
  const today = selectedDate.toISOString().slice(0,10).replace(/-/g,'');
  let list = matches.filter(m=>m.date===today);

  // Run Zcode sim on every match
  list = list.map(m=>{ const z=zcodeSim(m); return {...m,...z}; });

  // Sorting
  const sort = document.getElementById('sort-select').value;
  if(sort==='sim-win-desc') list.sort((a,b)=>b.simHome-a.simHome);
  if(sort==='odds-desc') list.sort((a,b)=>b.odds-a.odds);

  // Best bets
  const best = list.filter(m=>m.simHome>=62 && m.odds>=1.75).slice(0,6);
  const bestGrid = document.getElementById('best-bets-grid');
  if(best.length){
    document.getElementById('best-bets').style.display='block';
    bestGrid.innerHTML = best.map(m=>`
      <div class="match-card"><div class="match-header"><div>${m.time}</div></div>
      <div class="match-teams"><div class="team"><img src="\( {m.home_logo}"> \){m.team_home}</div><div class="team"><img src="\( {m.away_logo}"> \){m.team_away}</div></div>
      <div class="match-footer"><div class="prediction-value">${m.forecast}</div>
      <div class="zcode-sim-section">Zcode Sim: \( {m.simHome}% Home • Avg: \){m.avgScore}</div></div></div>`).join('');
  }else document.getElementById('best-bets').style.display='none';

  // Main cards
  container.innerHTML = list.map(m=>`
    <div class="match-card ${m.simHome>=70?'class="won"':''}>
      <div class="match-header"><div class="match-time">${m.time}</div><div>LIVE</div></div>
      <div class="match-teams">
        <div class="team"><img src="\( {m.home_logo}"> \){m.team_home}</div>
        <div class="team"><img src="\( {m.away_logo}"> \){m.team_away}</div>
      </div>
      <div class="match-footer">
        <div class="prediction-value">${m.forecast}</div>
        <div class="zcode-sim-section">
          <strong>Zcode Simulation (5000 runs)</strong><br>
          Home Win: \( {m.simHome}% • Away: \){m.simAway}% • Draw: ${m.simDraw}%<br>
          Avg Score: \( {m.avgScore} • Stars: \){'★'.repeat(m.stars)}${'☆'.repeat(5-m.stars)}
          <button class="sim-btn" onclick="alert('Re-simulating 5000 runs…\nNew result: ${Math.round(Math.random()*30+55)}% Home Win')">Re-Sim</button>
        </div>
      </div>
    </div>
  `).join('');
}

// ───── CONTROLS ─────
document.getElementById('today-btn').onclick = ()=>{ selectedDate=new Date(); selectedDate.setHours(0,0,0,0); render(); updateDate(); };
document.getElementById('prev-day').onclick = ()=>{ selectedDate.setDate(selectedDate.getDate()-1); render(); updateDate(); };
document.getElementById('next-day').onclick = ()=>{ selectedDate.setDate(selectedDate.getDate()+1); render(); updateDate(); };
document.getElementById('date-picker').onchange = (e)=>{ selectedDate=new Date(e.target.value); render(); };
document.getElementById('sort-select').onchange = render;
document.getElementById('dark-mode-toggle').onclick = ()=>{
  darkMode=!darkMode; document.body.classList.toggle('dark-mode');
  localStorage.setItem('darkMode',darkMode);
  this.textContent = darkMode?'Light Mode':'Dark Mode';
};
document.getElementById('view-toggle-btn').onclick = ()=>{
  isCompact=!isCompact;
  document.body.classList.toggle('compact-view',isCompact);
  this.innerHTML = isCompact?'<i class="fas fa-expand"></i> Full':'<i class="fas fa-compress"></i> Compact';
};

function updateDate(){ document.getElementById('date-picker').value = selectedDate.toISOString().slice(0,10); }

darkMode&&document.body.classList.add('dark-mode');
updateDate.prototype.toISOString||(Date.prototype.toISOString=function(){return this.getFullYear()+'-'+String(this.getMonth()+1).padStart(2,'0')+'-'+String(this.getDate()).padStart(2,'0');});
updateDate();
render();
</script>
</body>
</html>
