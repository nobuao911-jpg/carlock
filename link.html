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
  flex-direction:column; gap:10px; display:none;
}
.panel.show { display:flex; }
.row { display:flex; align-items:center; justify-content:space-between; }
.lbl { font-size:.85rem; color:#aaa; }
.val { font-size:.85rem; color:#fff; font-weight:bold; }

/* RSSIバー */
#rssiWrap { display:flex; flex-direction:column; gap:4px; }
#rssiBar { width:100%; height:12px; background:#333; border-radius:6px; overflow:hidden; }
#rssiLevel { height:100%; width:0%; border-radius:6px; transition:width .5s, background .5s; }

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
  content:""; position:absolute; width:20px; height:20px;
  left:4px; bottom:4px; background:#fff; border-radius:50%; transition:.3s;
}
input:checked + .sl { background:#2196f3; }
input:checked + .sl:before { transform:translateX(24px); }

#autoMsg { text-align:center; font-size:.82rem; color:#888; }

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
  color:#4f4; height:90px; overflow-y:auto;
}
</style>
</head>
<body>

<h1>🚗 CarLock BLE</h1>
<div id="st">未接続</div>

<div class="panel" id="autoPanel">
  <!-- オートロック ON/OFF -->
  <div class="row">
    <span class="lbl">🔄 オートロック (RSSI)</span>
    <label class="sw">
      <input type="checkbox" id="chkAuto" onchange="toggleAuto(this.checked)">
      <span class="sl"></span>
    </label>
  </div>

  <!-- RSSI表示 -->
  <div id="rssiWrap">
    <div class="row">
      <span class="lbl">📶 電波強度</span>
      <span class="val" id="rssiTxt">-- dBm</span>
    </div>
    <div id="rssiBar"><div id="rssiLevel"></div></div>
  </div>

  <!-- 解錠しきい値スライダー -->
  <div class="srow">
    <div class="slbl">
      <span>🔓 解錠する強度（近い）</span>
      <span id="ulVal">-65 dBm</span>
    </div>
    <input type="range" id="ulSlider" min="-95" max="-40" value="-65"
      oninput="onSliderChange()">
  </div>

  <!-- 施錠しきい値スライダー -->
  <div class="srow">
    <div class="slbl">
      <span>🔒 施錠する強度（遠い）</span>
      <span id="lkVal">-80 dBm</span>
    </div>
    <input type="range" id="lkSlider" min="-95" max="-40" value="-80"
      oninput="onSliderChange()">
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
let isLocked = true, autoMode = false;
let sliderTimer = null;

function addLog(m) {
  const d = document.getElementById('log');
  d.innerHTML += m + '<br>'; d.scrollTop = d.scrollHeight;
}
function setSt(msg, bg) {
  const s = document.getElementById('st');
  s.textContent = msg; s.style.background = bg||'#222';
}
function setUI(on) {
  ['bLock','bUnlock','bHazard'].forEach(id =>
    document.getElementById(id).disabled = !on);
  document.getElementById('bConnect').disabled = on;
  document.getElementById('autoPanel').className = 'panel'+(on?' show':'');
}

// RSSI表示更新
function showRSSI(rssi) {
  document.getElementById('rssiTxt').textContent = rssi + ' dBm';
  const pct = Math.max(0, Math.min(100, (rssi+100)/60*100));
  const bar = document.getElementById('rssiLevel');
  bar.style.width = pct + '%';
  bar.style.background = pct>60?'#4f4':pct>30?'#fa0':'#f44';
}

// スライダー変更時 → ESP32に送信（500ms後に送信してチャタリング防止）
function onSliderChange() {
  const ul = parseInt(document.getElementById('ulSlider').value);
  const lk = parseInt(document.getElementById('lkSlider').value);
  // 解錠>施錠になるように制限
  if (ul <= lk) {
    document.getElementById('ulSlider').value = lk + 5;
  }
  document.getElementById('ulVal').textContent =
    document.getElementById('ulSlider').value + ' dBm';
  document.getElementById('lkVal').textContent = lk + ' dBm';

  if (sliderTimer) clearTimeout(sliderTimer);
  sliderTimer = setTimeout(() => {
    sendCmd('SET_UNLOCK_RSSI:' + document.getElementById('ulSlider').value);
    sendCmd('SET_LOCK_RSSI:' + lk);
  }, 500);
}

// オートロック ON/OFF
function toggleAuto(on) {
  autoMode = on;
  sendCmd(on ? 'AUTO_ON' : 'AUTO_OFF');
  const msg = document.getElementById('autoMsg');
  if (on) {
    msg.textContent = '🔄 RSSI監視中...';
    msg.style.color = '#2196f3';
  } else {
    msg.textContent = 'オートロック OFF';
    msg.style.color = '#888';
  }
}

async function doConnect() {
  if (!navigator.bluetooth) {
    setSt('Chromeで開いてください','#b71c1c'); return;
  }
  try {
    setSt('スキャン中...','#1565c0');
    bleDevice = await navigator.bluetooth.requestDevice({
      filters:[{name:'CarLock-BLE'}],
      optionalServices:[SVC]
    });
    setSt('接続中...','#1565c0');
    const srv = await bleDevice.gatt.connect();
    const svc = await srv.getPrimaryService(SVC);
    ch = await svc.getCharacteristic(CHAR);
    await ch.startNotifications();

    ch.addEventListener('characteristicvaluechanged', e => {
      const v = new TextDecoder().decode(e.target.value);
      addLog('← ' + v);

      if (v === 'LOCKED')     { setSt('🔒 施錠中','#b71c1c'); isLocked=true; }
      if (v === 'UNLOCKED')   { setSt('🔓 解錠中','#1b5e20'); isLocked=false; }
      if (v === 'HAZARD_ON')  setSt('⚠️ ハザード中','#e65100');
      if (v === 'HAZARD_OFF') setSt(isLocked?'🔒 施錠中':'🔓 解錠中',
                                     isLocked?'#b71c1c':'#1b5e20');
      if (v === 'AUTO:ON')  {
        document.getElementById('autoMsg').textContent = '🔄 RSSI監視中...';
        document.getElementById('autoMsg').style.color = '#2196f3';
      }
      // RSSIを受信
      if (v.startsWith('RSSI:')) {
        const rssi = parseInt(v.split(':')[1]);
        showRSSI(rssi);
        const ul = parseInt(document.getElementById('ulSlider').value);
        const lk = parseInt(document.getElementById('lkSlider').value);
        const msg = document.getElementById('autoMsg');
        if (autoMode) {
          if (rssi > ul)      { msg.textContent='📶 近い '+rssi+'dBm → 解錠判定中'; msg.style.color='#4f4'; }
          else if (rssi < lk) { msg.textContent='📶 遠い '+rssi+'dBm → 施錠判定中'; msg.style.color='#f44'; }
          else                { msg.textContent='📶 中間 '+rssi+'dBm → 待機中';     msg.style.color='#aaa'; }
        }
      }
    });

    bleDevice.addEventListener('gattserverdisconnected', () => {
      setSt('切断されました','#333');
      setUI(false); ch = null;
      document.getElementById('chkAuto').checked = false;
      autoMode = false;
    });

    setSt('✅ 接続済み','#1b5e20');
    setUI(true);
    addLog('接続完了');
    setTimeout(() => sendCmd('STATUS'), 600);
  } catch(e) {
    setSt('接続失敗','#b71c1c');
    addLog('エラー: '+e.message);
  }
}

async function sendCmd(cmd) {
  if (!ch) return;
  try {
    await ch.writeValue(new TextEncoder().encode(cmd));
    if (!cmd.startsWith('SET_')) addLog('→ '+cmd);
  } catch(e) { addLog('送信エラー: '+e.message); }
}
</script>
</body>
</html>
