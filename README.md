[index.html](https://github.com/user-attachments/files/29009683/index.html)
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
<title>CarLock BLE</title>
<style>
* { box-sizing:border-box; margin:0; padding:0; }
body {
  font-family:sans-serif; background:#111; color:#fff;
  min-height:100vh; display:flex; flex-direction:column;
  align-items:center; padding:20px; gap:12px;
  padding-top:30px;
}
h1 { font-size:1.3rem; color:#7eb8ff; }
#st {
  font-size:1rem; padding:10px 24px; border-radius:30px;
  font-weight:bold; background:#222; text-align:center;
  width:100%; max-width:320px;
}
.panel {
  width:100%; max-width:320px; background:#1a1a1a;
  border-radius:14px; padding:14px 16px;
  display:flex; flex-direction:column; gap:10px;
}
.row { display:flex; align-items:center; justify-content:space-between; }
.lbl { font-size:.85rem; color:#aaa; }
.val { font-size:.85rem; color:#fff; font-weight:bold; }

/* RSSIバー */
#rssiBar { width:100%; height:10px; background:#333; border-radius:5px; overflow:hidden; }
#rssiLevel { height:100%; width:0%; border-radius:5px; transition:width .6s,background .6s; }

/* スライダー */
.srow { display:flex; flex-direction:column; gap:4px; }
.slbl { font-size:.75rem; color:#888; display:flex; justify-content:space-between; }
input[type=range] { width:100%; accent-color:#2196f3; cursor:pointer; }

/* トグル */
.sw { position:relative; width:52px; height:28px; flex-shrink:0; }
.sw input { opacity:0; width:0; height:0; }
.sl {
  position:absolute; inset:0; background:#444;
  border-radius:28px; cursor:pointer; transition:.3s;
}
.sl:before {
  content:""; position:absolute;
  width:20px; height:20px; left:4px; bottom:4px;
  background:#fff; border-radius:50%; transition:.3s;
}
input:checked + .sl { background:#2196f3; }
input:checked + .sl:before { transform:translateX(24px); }

#autoMsg { text-align:center; font-size:.82rem; color:#888; min-height:1.2em; }

.btn {
  width:100%; max-width:320px; padding:20px;
  font-size:1.35rem; font-weight:bold; border:none;
  border-radius:16px; cursor:pointer; transition:opacity .15s;
}
.btn:active { opacity:.65; }
.btn:disabled { opacity:.28; cursor:not-allowed; }
#bConnect { background:#2196f3; color:#fff; }
#bLock    { background:#e53935; color:#fff; }
#bUnlock  { background:#43a047; color:#fff; }
#bHazard  { background:#fb8c00; color:#fff; }

#log {
  width:100%; max-width:320px; background:#1a1a1a;
  border-radius:10px; padding:10px 12px;
  font-size:.73rem; font-family:monospace;
  color:#4f4; height:80px; overflow-y:auto;
}
</style>
</head>
<body>

<h1>🚗 CarLock BLE</h1>
<div id="st">未接続</div>

<!-- オートロックパネル（接続後に表示） -->
<div class="panel" id="autoPanel" style="display:none">
  <div class="row">
    <span class="lbl">🔄 オートロック</span>
    <label class="sw">
      <input type="checkbox" id="autoChk" onchange="toggleAuto(this.checked)">
      <span class="sl"></span>
    </label>
  </div>
  <div class="row">
    <span class="lbl">電波強度 (RSSI)</span>
    <span class="val" id="rssiTxt">-- dBm</span>
  </div>
  <div id="rssiBar"><div id="rssiLevel"></div></div>
  <div class="srow">
    <div class="slbl">
      <span>🔓 解錠する距離（近）</span>
      <span id="ulVal">-65 dBm</span>
    </div>
    <input type="range" id="ulThresh" min="-95" max="-45" value="-65"
      oninput="document.getElementById('ulVal').textContent=this.value+' dBm'">
  </div>
  <div class="srow">
    <div class="slbl">
      <span>🔒 施錠する距離（遠）</span>
      <span id="lkVal">-80 dBm</span>
    </div>
    <input type="range" id="lkThresh" min="-95" max="-45" value="-80"
      oninput="document.getElementById('lkVal').textContent=this.value+' dBm'">
  </div>
  <div id="autoMsg">オートロック OFF</div>
</div>

<button class="btn" id="bConnect" onclick="doConnect()">🔵 接続する</button>
<button class="btn" id="bLock"    onclick="sendCmd('LOCK')"   disabled>🔒 ロック</button>
<button class="btn" id="bUnlock"  onclick="sendCmd('UNLOCK')" disabled>🔓 アンロック</button>
<button class="btn" id="bHazard"  onclick="sendCmd('HAZARD')" disabled>⚠️ ハザード5秒</button>
<div id="log"></div>

<script>
const SVC  = '0000ffe0-0000-1000-8000-00805f9b34fb';
const CHAR = '0000ffe1-0000-1000-8000-00805f9b34fb';
let ch = null, bleDevice = null;
let autoMode = false, isLocked = true;
let autoTimer = null;
// 直近RSSIサンプル（平均化用）
let rssiSamples = [];

function addLog(m) {
  const d = document.getElementById('log');
  d.innerHTML += m + '<br>';
  d.scrollTop = d.scrollHeight;
}
function setSt(msg, bg) {
  const s = document.getElementById('st');
  s.textContent = msg; s.style.background = bg || '#222';
}
function setUI(connected) {
  ['bLock','bUnlock','bHazard'].forEach(id =>
    document.getElementById(id).disabled = !connected);
  document.getElementById('bConnect').disabled = connected;
  document.getElementById('autoPanel').style.display = connected ? 'flex' : 'none';
}

/* ── RSSI 取得（Web Bluetooth 標準API）────────────────── */
async function fetchRSSI() {
  if (!bleDevice || !bleDevice.gatt.connected) return null;
  try {
    // Chrome 87+ で利用可能な標準API
    const rssi = await bleDevice.gatt.getPrimaryService(SVC)
      .then(() => null); // プレースホルダ
    // 実際のRSSI取得: watchAdvertisements API
    return null;
  } catch(e) { return null; }
}

/* Chrome では readRSSI が非公開のため
   watchAdvertisements() でアドバタイズパケットのRSSIを取得する方式 */
async function startRSSIWatch() {
  if (!bleDevice) return;
  try {
    await bleDevice.watchAdvertisements();
    bleDevice.addEventListener('advertisementreceived', e => {
      const rssi = e.rssi;
      if (rssi == null) return;
      updateRSSI(rssi);
    });
    addLog('RSSI監視開始');
  } catch(e) {
    addLog('RSSI取得: ' + e.message);
    addLog('※ RSSIはChrome フラグ要有効化');
    // フォールバック: タイマーで疑似距離監視
    autoTimer = setInterval(checkFallback, 2000);
  }
}

function updateRSSI(rssi) {
  // 移動平均（5サンプル）
  rssiSamples.push(rssi);
  if (rssiSamples.length > 5) rssiSamples.shift();
  const avg = Math.round(rssiSamples.reduce((a,b)=>a+b,0)/rssiSamples.length);

  document.getElementById('rssiTxt').textContent = avg + ' dBm';
  const pct = Math.max(0, Math.min(100, (avg + 100) / 60 * 100));
  const bar = document.getElementById('rssiLevel');
  bar.style.width = pct + '%';
  bar.style.background = pct > 60 ? '#4f4' : pct > 30 ? '#fa0' : '#f44';

  if (autoMode) checkAutoLock(avg);
}

function checkAutoLock(rssi) {
  const ul = parseInt(document.getElementById('ulThresh').value);
  const lk = parseInt(document.getElementById('lkThresh').value);
  const msg = document.getElementById('autoMsg');

  if (rssi >= ul && isLocked) {
    msg.textContent = '🔓 近い → 自動解錠！';
    msg.style.color = '#4f4';
    sendCmd('UNLOCK');
  } else if (rssi < lk && !isLocked) {
    msg.textContent = '🔒 遠い → 自動施錠！';
    msg.style.color = '#f44';
    sendCmd('LOCK');
  } else {
    msg.textContent = '🔄 監視中 ' + rssi + ' dBm';
    msg.style.color = '#aaa';
  }
}

// watchAdvertisements が使えない場合の接続維持確認
function checkFallback() {
  if (!bleDevice || !bleDevice.gatt.connected) return;
  document.getElementById('autoMsg').textContent =
    '⚠️ RSSI取得不可。手動で操作してください。';
  document.getElementById('autoMsg').style.color = '#fa0';
}

function toggleAuto(on) {
  autoMode = on;
  const msg = document.getElementById('autoMsg');
  if (on) {
    rssiSamples = [];
    msg.textContent = '🔄 オートロック ON - 監視中...';
    msg.style.color = '#2196f3';
    startRSSIWatch();
    addLog('オートロック ON');
  } else {
    msg.textContent = 'オートロック OFF';
    msg.style.color = '#888';
    if (autoTimer) { clearInterval(autoTimer); autoTimer = null; }
    if (bleDevice) try { bleDevice.unwatchAdvertisements(); } catch(e) {}
    addLog('オートロック OFF');
  }
}

async function doConnect() {
  if (!navigator.bluetooth) {
    setSt('Chromeで開いてください', '#b71c1c'); return;
  }
  try {
    setSt('スキャン中...', '#1565c0');
    bleDevice = await navigator.bluetooth.requestDevice({
      filters: [{ name: 'CarLock-BLE' }],
      optionalServices: [SVC]
    });
    setSt('接続中...', '#1565c0');
    const srv = await bleDevice.gatt.connect();
    const svc = await srv.getPrimaryService(SVC);
    ch = await svc.getCharacteristic(CHAR);
    await ch.startNotifications();
    ch.addEventListener('characteristicvaluechanged', e => {
      const v = new TextDecoder().decode(e.target.value);
      addLog('← ' + v);
      if (v === 'LOCKED')     { setSt('🔒 施錠中', '#b71c1c');  isLocked = true; }
      if (v === 'UNLOCKED')   { setSt('🔓 解錠中', '#1b5e20');  isLocked = false; }
      if (v === 'HAZARD_ON')  setSt('⚠️ ハザード中', '#e65100');
      if (v === 'HAZARD_OFF') setSt('✅ 完了', '#1b5e20');
    });
    bleDevice.addEventListener('gattserverdisconnected', () => {
      setSt('切断されました', '#333');
      setUI(false); ch = null;
      if (autoTimer) { clearInterval(autoTimer); autoTimer = null; }
      document.getElementById('autoChk').checked = false;
      autoMode = false;
    });
    setSt('✅ 接続済み', '#1b5e20');
    setUI(true);
    addLog('接続完了');
    setTimeout(() => sendCmd('STATUS'), 500);
  } catch(e) {
    setSt('接続失敗', '#b71c1c');
    addLog('エラー: ' + e.message);
  }
}

async function sendCmd(cmd) {
  if (!ch) return;
  try {
    await ch.writeValue(new TextEncoder().encode(cmd));
    addLog('→ ' + cmd);
  } catch(e) { addLog('送信エラー: ' + e.message); }
}
</script>
</body>
</html>
