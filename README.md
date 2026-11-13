# hydro-ai-lab
Hydro AI 液壓智慧實驗室


<!DOCTYPE html>

<html lang="zh-Hant">

<head>

<meta charset="UTF-8">

<meta name="viewport" content="width=device-width, initial-scale=1">

<meta name="theme-color" content="#101827">

<title>Hydro AI V2 液壓智慧實驗室</title>

<style>

*{box-sizing:border-box}

body{margin:0;background:#101827;color:#edf4ff;font-family:Arial,sans-serif}

header{padding:18px;background:#172438;border-bottom:1px solid #30435b}

h1{font-size:21px;margin:0}

header p{font-size:12px;color:#9db1ca;margin:7px 0 0}

main{padding:14px;max-width:900px;margin:auto}

.panel{background:#1b2a40;border:1px solid #354b66;border-radius:12px;padding:14px;margin-bottom:14px}

h2{font-size:16px;margin:0 0 12px}

button{border:0;border-radius:8px;padding:12px;background:#2878d0;color:white;font-weight:bold}

button:active{opacity:.7}

.controls{display:grid;grid-template-columns:repeat(3,1fr);gap:8px}

input{width:100%;padding:10px;border-radius:7px;border:1px solid #53677e;background:#101d30;color:white}

label{font-size:12px;color:#b9c9dc;display:block;margin-bottom:6px}

.fields{display:grid;grid-template-columns:1fr 1fr;gap:12px}

.metric{background:#101d30;border-radius:8px;padding:12px}

.metric strong{display:block;font-size:21px;color:#59c5ff;margin-top:5px}

svg{width:100%;height:auto;background:#f4f7fa;border-radius:8px}

.pipe{fill:none;stroke:#8797a8;stroke-width:5}

.active{stroke:#08a8e8;stroke-dasharray:10 7;animation:flow .7s linear infinite}

@keyframes flow{to{stroke-dashoffset:-34}}

.symbol{stroke:#27384c;stroke-width:2;fill:white}

.status{color:#70d8ff;font-weight:bold}

.note{font-size:12px;color:#aebed0;line-height:1.6}

</style>

</head>

<body>

<header>

<h1>Hydro AI V2</h1>

<p>液壓智慧實驗室｜2D 動態油路模擬</p>

</header>

<main>

<section class="panel">

<h2>液壓系統工作台</h2>

<div class="controls">

<button onclick="setMode('extend')">油缸伸出</button>

<button onclick="setMode('neutral')">中位停止</button>

<button onclick="setMode('retract')">油缸縮回</button>

</div>

<p>目前狀態：<span class="status" id="status">中位停止</span></p>

<svg viewBox="0 0 340 260" role="img" aria-label="雙動油缸及方向閥液壓迴路">

<!-- 油缸 -->

<rect x="205" y="30" width="105" height="35" rx="4" class="symbol"/>

<rect id="piston" x="212" y="35" width="12" height="25" fill="#2878d0"/>

<line id="rod" x1="224" y1="47" x2="260" y2="47" stroke="#27384c" stroke-width="5"/>

<text x="205" y="22" font-size="10" fill="#27384c">雙動油缸</text>

<!-- 方向閥 -->

<rect x="115" y="105" width="55" height="55" class="symbol"/>

<rect x="170" y="105" width="55" height="55" class="symbol"/>

<path d="M122 150 L160 115 M153 115 L160 115 L160 122" stroke="#27384c" fill="none" stroke-width="2"/>

<path d="M178 115 L215 150 M208 150 L215 150 L215 143" stroke="#27384c" fill="none" stroke-width="2"/>

<text x="132" y="176" font-size="10" fill="#27384c">4/3 方向閥</text>

<!-- 泵浦和油箱 -->

<circle cx="42" cy="125" r="18" class="symbol"/>

<path d="M34 134 L50 125 L34 116 Z" fill="#2878d0"/>

<text x="28" y="155" font-size="10" fill="#27384c">泵浦</text>

<path d="M20 205 L64 205 L58 225 L26 225 Z" class="symbol"/>

<text x="30" y="240" font-size="10" fill="#27384c">油箱</text>

<!-- 管線 -->

<path id="pLine" class="pipe" d="M60 125 H115"/>

<path id="tLine" class="pipe" d="M115 150 H80 V215 H64"/>

<path id="aLine" class="pipe" d="M142 105 V80 H205"/>

<path id="bLine" class="pipe" d="M198 105 V90 H285 V65"/>

<text x="75" y="116" font-size="11" fill="#27384c">P</text>

<text x="84" y="169" font-size="11" fill="#27384c">T</text>

<text x="150" y="78" font-size="11" fill="#27384c">A</text>

<text x="270" y="84" font-size="11" fill="#27384c">B</text>

</svg>

<p class="note">藍色虛線代表目前設定的模擬油流路徑。此為教學用簡化示意，尚未包含完整閥芯中位機能與負載模型。</p>

</section>

<section class="panel">

<h2>模擬參數</h2>

<div class="fields">

<div><label>系統壓力（bar）</label><input id="pressure" type="number" value="100" min="0"></div>

<div><label>泵浦流量（L/min）</label><input id="flow" type="number" value="20" min="0"></div>

<div><label>油缸缸徑（mm）</label><input id="bore" type="number" value="80" min="1"></div>

<div><label>活塞桿直徑（mm）</label><input id="rodSize" type="number" value="40" min="0"></div>

</div>

<button style="width:100%;margin-top:12px" onclick="calculate()">重新計算</button>

</section>

<section class="panel">

<h2>運算結果</h2>

<div class="fields">

<div class="metric">伸出速度<strong id="extendSpeed">—</strong></div>

<div class="metric">縮回速度<strong id="retractSpeed">—</strong></div>

<div class="metric">系統壓力<strong id="pressureValue">100 bar</strong></div>

<div class="metric">泵浦流量<strong id="flowValue">20 L/min</strong></div>

</div>

</section>

<section class="panel">

<h2>手動故障測試</h2>

<label>選擇模擬故障</label>

<select id="fault" style="width:100%;padding:10px;border-radius:7px">

<option value="none">正常狀態</option>

<option value="leak">模擬內漏</option>

<option value="lowflow">模擬流量不足</option>

<option value="stuck">模擬方向閥卡滯</option>

</select>

<button style="width:100%;margin-top:10px" onclick="diagnose()">執行故障分析</button>

<p id="diagnosis" class="note">等待故障測試。</p>

</section>

</main>

<script>

let mode='neutral';

function setMode(m){

 mode=m;

 document.getElementById('status').textContent=

 m==='extend'?'油缸伸出':m==='retract'?'油缸縮回':'中位停止';

 ['pLine','tLine','aLine','bLine'].forEach(id=>{

  document.getElementById(id).setAttribute('class','pipe');

 });

 if(m==='extend'){

  ['pLine','aLine','bLine','tLine'].forEach(id=>

   document.getElementById(id).setAttribute('class','pipe active'));

  document.getElementById('piston').setAttribute('x','245');

  document.getElementById('rod').setAttribute('x1','257');

 }else if(m==='retract'){

  ['pLine','aLine','bLine','tLine'].forEach(id=>

   document.getElementById(id).setAttribute('class','pipe active'));

  document.getElementById('piston').setAttribute('x','212');

  document.getElementById('rod').setAttribute('x1','224');

 }else{

  document.getElementById('piston').setAttribute('x','212');

  document.getElementById('rod').setAttribute('x1','224');

 }

}

function calculate(){

 const q=Number(document.getElementById('flow').value);

 const d=Number(document.getElementById('bore').value);

 const r=Number(document.getElementById('rodSize').value);

 const p=Number(document.getElementById('pressure').value);

 if(![q,d,r,p].every(Number.isFinite)||q<0||d<=0||r<0||r>=d||p<0){

  alert('請確認參數：流量、壓力不可為負數，桿徑須小於缸徑。');

  return;

 }

 const area=Math.PI*d*d/4;

 const annulus=Math.PI*(d*d-r*r)/4;

 const ext=q*1000000/60/area;

 const ret=q*1000000/60/annulus;

 document.getElementById('extendSpeed').textContent=(ext/1000).toFixed(3)+' m/s';

 document.getElementById('retractSpeed').textContent=(ret/1000).toFixed(3)+' m/s';

 document.getElementById('pressureValue').textContent=p+' bar';

 document.getElementById('flowValue').textContent=q+' L/min';

}

function diagnose(){

 const f=document.getElementById('fault').value;

 const messages={

 none:'目前未設定故障。這不代表實體設備已確認正常。',

 leak:'模擬內漏：可進一步檢查油缸密封、閥內洩及保壓回路。',

 lowflow:'模擬流量不足：可檢查泵浦供油、吸油條件、濾芯及節流位置。',

 stuck:'模擬方向閥卡滯：可檢查閥芯動作、控制訊號及先導油路。'

 };

 document.getElementById('diagnosis').textContent=messages[f];

}

calculate();

</script>

</body>

</html>