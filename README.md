<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>PACE 校園運動分析站</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: -apple-system, sans-serif; background: #0f172a; color: #f8fafc; display: flex; height: 100vh; }
    #sidebar { width: 340px; background: #1e293b; padding: 20px; display: flex; flex-direction: column; gap: 14px; border-right: 1px solid #334155; }
    .badge { background: #38bdf8; color: #0f172a; padding: 3px 8px; border-radius: 4px; font-weight: bold; font-size: 12px; display: inline-block; width: fit-content; }
    h1 { font-size: 18px; margin-top: 4px; }
    .card { background: #0f172a; padding: 12px; border-radius: 8px; border: 1px solid #334155; }
    .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; text-align: center; }
    .metric { font-size: 18px; font-weight: bold; color: #38bdf8; font-family: monospace; }
    .label { font-size: 11px; color: #94a3b8; }
    button { width: 100%; padding: 10px; border: none; border-radius: 6px; font-weight: bold; cursor: pointer; margin-top: 6px; }
    .btn-main { background: #38bdf8; color: #0f172a; }
    .btn-sub { background: #334155; color: white; }
    #map { flex: 1; height: 100%; }
    @media (max-width: 768px) { body { flex-direction: column-reverse; } #sidebar { width: 100%; height: 50vh; } #map { height: 50vh; } }
  </style>
</head>
<body>
  <div id="sidebar">
    <div>
      <span class="badge">逢甲大學 · 精密系統二</span>
      <h1>PACE 校園運動監測</h1>
    </div>
    <div class="card">
      <div class="grid">
        <div><div class="metric" id="dist">0.00</div><div class="label">距離 (km)</div></div>
        <div><div class="metric" id="time">00:00</div><div class="label">時間</div></div>
        <div><div class="metric" id="speed">0.0</div><div class="label">速度 (km/h)</div></div>
        <div><div class="metric" id="cal">0</div><div class="label">熱量 (kcal)</div></div>
      </div>
      <button class="btn-main" onclick="startSim()">體驗示範運動 (室內測試)</button>
      <button class="btn-sub" onclick="downloadJSON()">下載軌跡 JSON</button>
    </div>
    <div style="font-size: 12px; color: #94a3b8;" id="status">狀態：等待操作</div>
  </div>
  <div id="map"></div>

  <script>
    const map = L.map('map').setView([24.1788, 120.6466], 17);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 19 }).addTo(map);

    let pts = [];
    let line = L.polyline([], { color: '#f97316', weight: 4 }).addTo(map);

    function startSim() {
      document.getElementById('status').innerText = '示範運動中...';
      let sec = 0;
      pts = [];
      line.setLatLngs([]);
      
      const timer = setInterval(() => {
        sec += 1;
        document.getElementById('time').innerText = `00:${sec < 10 ? '0' + sec : sec}`;
        const lat = 24.1788 + 0.0006 * Math.sin(sec * 0.3);
        const lng = 120.6466 + 0.0008 * Math.cos(sec * 0.3);
        pts.push({ lat, lng, time: new Date().toISOString() });
        line.addLatLng([lat, lng]);
        map.panTo([lat, lng]);

        const km = (sec * 0.015).toFixed(2);
        document.getElementById('dist').innerText = km;
        document.getElementById('speed').innerText = '5.4';
        document.getElementById('cal').innerText = (sec * 0.8).toFixed(0);

        if (sec >= 30) {
          clearInterval(timer);
          document.getElementById('status').innerText = `運動完成！共記錄 ${pts.length} 個點位`;
        }
      }, 500);
    }

    function downloadJSON() {
      if (pts.length === 0) { alert('請先點擊示範運動累積數據！'); return; }
      const blob = new Blob([JSON.stringify({ userId: 'User', points: pts }, null, 2)], { type: 'application/json' });
      const a = document.createElement('a');
      a.href = URL.createObjectURL(blob);
      a.download = 'my_track.json';
      a.click();
    }
  </script>
</body>
</html>
