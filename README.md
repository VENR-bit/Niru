# Niru
trip for Niru Villege
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Niru Village Trip Map</title>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.css" />
<script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.js"></script>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { font-family: 'Segoe UI', system-ui, -apple-system, sans-serif; background: #0f172a; color: #e2e8f0; }

  #map { width: 100%; height: 100vh; z-index: 1; }

  /* Sidebar */
  .sidebar {
    position: fixed; top: 0; left: 0; width: 370px; height: 100vh;
    background: linear-gradient(180deg, #0f172a 0%, #1e293b 100%);
    z-index: 1000; overflow-y: auto; border-right: 1px solid #334155;
    transition: transform 0.35s cubic-bezier(0.4, 0, 0.2, 1);
    box-shadow: 4px 0 24px rgba(0,0,0,0.4);
  }
  .sidebar.collapsed { transform: translateX(-370px); }

  .sidebar-header {
    padding: 28px 24px 20px;
    background: linear-gradient(135deg, #1e3a5f 0%, #0f172a 100%);
    border-bottom: 1px solid #334155;
  }
  .sidebar-header h1 { font-size: 20px; font-weight: 700; color: #f8fafc; margin-bottom: 4px; letter-spacing: -0.3px; }
  .sidebar-header p { font-size: 13px; color: #94a3b8; line-height: 1.5; }

  .trip-meta {
    display: flex; gap: 12px; margin-top: 14px; flex-wrap: wrap;
  }
  .meta-chip {
    font-size: 11px; padding: 4px 10px; border-radius: 20px;
    background: rgba(56, 189, 248, 0.12); color: #7dd3fc; border: 1px solid rgba(56, 189, 248, 0.2);
  }

  /* Day tabs */
  .day-tabs {
    display: flex; padding: 0; border-bottom: 1px solid #334155;
  }
  .day-tab {
    flex: 1; padding: 12px 16px; text-align: center; font-size: 13px; font-weight: 600;
    cursor: pointer; border: none; background: transparent; color: #64748b;
    transition: all 0.2s; border-bottom: 2px solid transparent;
  }
  .day-tab:hover { color: #cbd5e1; background: rgba(255,255,255,0.03); }
  .day-tab.active { color: #38bdf8; border-bottom-color: #38bdf8; background: rgba(56, 189, 248, 0.06); }

  /* Location cards */
  .locations { padding: 12px 16px 80px; }

  .location-card {
    padding: 14px 16px; margin-bottom: 8px; border-radius: 10px;
    background: rgba(255,255,255,0.03); border: 1px solid transparent;
    cursor: pointer; transition: all 0.2s;
  }
  .location-card:hover { background: rgba(56, 189, 248, 0.06); border-color: rgba(56, 189, 248, 0.15); }
  .location-card.active { background: rgba(56, 189, 248, 0.1); border-color: #38bdf8; }

  .card-top { display: flex; align-items: center; gap: 10px; margin-bottom: 6px; }
  .card-icon {
    width: 32px; height: 32px; border-radius: 8px; display: flex; align-items: center;
    justify-content: center; font-size: 16px; flex-shrink: 0;
  }
  .card-title { font-size: 14px; font-weight: 600; color: #f1f5f9; }
  .card-subtitle { font-size: 11px; color: #64748b; margin-top: 1px; }
  .card-desc { font-size: 12.5px; color: #94a3b8; line-height: 1.55; margin-top: 4px; }
  .card-tags { display: flex; gap: 6px; margin-top: 8px; flex-wrap: wrap; }
  .card-tag {
    font-size: 10px; padding: 2px 8px; border-radius: 10px;
    background: rgba(255,255,255,0.05); color: #94a3b8;
  }

  /* Toggle button */
  .sidebar-toggle {
    position: fixed; top: 16px; z-index: 1001;
    width: 40px; height: 40px; border-radius: 10px;
    background: #1e293b; border: 1px solid #475569; color: #e2e8f0;
    cursor: pointer; display: flex; align-items: center; justify-content: center;
    font-size: 18px; transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
    box-shadow: 0 2px 12px rgba(0,0,0,0.3);
  }
  .sidebar-toggle.open { left: 382px; }
  .sidebar-toggle.closed { left: 16px; }

  /* Legend */
  .legend {
    position: fixed; bottom: 24px; right: 24px; z-index: 1000;
    background: #1e293b; border: 1px solid #334155; border-radius: 12px;
    padding: 14px 18px; box-shadow: 0 4px 20px rgba(0,0,0,0.4);
  }
  .legend h4 { font-size: 11px; color: #64748b; text-transform: uppercase; letter-spacing: 0.8px; margin-bottom: 8px; }
  .legend-item { display: flex; align-items: center; gap: 8px; margin-bottom: 5px; font-size: 12px; color: #cbd5e1; }
  .legend-dot { width: 10px; height: 10px; border-radius: 50%; flex-shrink: 0; }
  .legend-line { width: 20px; height: 3px; border-radius: 2px; flex-shrink: 0; }

  /* Map custom markers */
  .custom-marker {
    width: 36px; height: 36px; border-radius: 50%; display: flex;
    align-items: center; justify-content: center; font-size: 16px;
    box-shadow: 0 3px 12px rgba(0,0,0,0.4); border: 2px solid white;
    transition: transform 0.2s;
  }
  .custom-marker:hover { transform: scale(1.15); }

  .leaflet-popup-content-wrapper {
    background: #1e293b !important; border: 1px solid #475569 !important;
    border-radius: 12px !important; box-shadow: 0 8px 32px rgba(0,0,0,0.5) !important;
  }
  .leaflet-popup-tip { background: #1e293b !important; border: 1px solid #475569 !important; }
  .leaflet-popup-content { color: #e2e8f0 !important; font-family: 'Segoe UI', system-ui, sans-serif !important; margin: 14px 16px !important; }
  .popup-title { font-size: 15px; font-weight: 700; color: #f8fafc; margin-bottom: 4px; }
  .popup-chinese { font-size: 12px; color: #7dd3fc; margin-bottom: 8px; }
  .popup-desc { font-size: 12.5px; color: #94a3b8; line-height: 1.5; }
  .popup-detail { font-size: 11.5px; color: #64748b; margin-top: 8px; padding-top: 8px; border-top: 1px solid #334155; }

  /* Distance info */
  .distance-bar {
    position: fixed; bottom: 24px; left: 50%; transform: translateX(-50%); z-index: 1000;
    background: #1e293b; border: 1px solid #334155; border-radius: 12px;
    padding: 10px 20px; display: flex; gap: 24px; align-items: center;
    box-shadow: 0 4px 20px rgba(0,0,0,0.4); font-size: 12px;
  }
  .distance-bar .stat { text-align: center; }
  .distance-bar .stat-val { font-size: 16px; font-weight: 700; color: #38bdf8; }
  .distance-bar .stat-label { color: #64748b; font-size: 10px; text-transform: uppercase; letter-spacing: 0.5px; }

  @media (max-width: 768px) {
    .sidebar { width: 100%; height: 45vh; top: auto; bottom: 0; border-right: none; border-top: 1px solid #334155; }
    .sidebar.collapsed { transform: translateY(45vh); }
    .sidebar-toggle.open { left: 16px; bottom: calc(45vh + 12px); top: auto; }
    .sidebar-toggle.closed { left: 16px; bottom: 16px; top: auto; }
    #map { height: 58vh; }
    .legend { bottom: calc(45vh + 16px); }
    .distance-bar { display: none; }
  }
</style>
</head>
<body>

<div id="map"></div>

<div class="sidebar" id="sidebar">
  <div class="sidebar-header">
    <h1>Niru Village Trip</h1>
    <p>2-Day Spring Itinerary — Shangri-La, Yunnan</p>
    <div class="trip-meta">
      <span class="meta-chip">2 Days</span>
      <span class="meta-chip">2,700 – 3,950 m</span>
      <span class="meta-chip">Spring</span>
      <span class="meta-chip">~CNY 600</span>
    </div>
  </div>

  <div class="day-tabs">
    <button class="day-tab active" data-day="all">All</button>
    <button class="day-tab" data-day="0">Travel</button>
    <button class="day-tab" data-day="1">Day 1</button>
    <button class="day-tab" data-day="2">Day 2</button>
  </div>

  <div class="locations" id="locations"></div>
</div>

<button class="sidebar-toggle open" id="sidebarToggle">◀</button>

<div class="legend">
  <h4>Legend</h4>
  <div class="legend-item"><span class="legend-dot" style="background:#f97316"></span> Transport Hub</div>
  <div class="legend-item"><span class="legend-dot" style="background:#22c55e"></span> Village / Stay</div>
  <div class="legend-item"><span class="legend-dot" style="background:#3b82f6"></span> Nature / Hike</div>
  <div class="legend-item"><span class="legend-dot" style="background:#a855f7"></span> Culture / Temple</div>
  <div class="legend-item"><span class="legend-line" style="background:#f97316; opacity:0.6"></span> Drive Route</div>
  <div class="legend-item"><span class="legend-line" style="background:#3b82f6; opacity:0.6; border-style:dashed"></span> Hiking Trail</div>
</div>

<div class="distance-bar">
  <div class="stat"><div class="stat-val">88 km</div><div class="stat-label">Shangri-La → Niru</div></div>
  <div class="stat"><div class="stat-val">~3.5 hr</div><div class="stat-label">Drive Time</div></div>
  <div class="stat"><div class="stat-val">12 km</div><div class="stat-label">Waterfall Trail</div></div>
  <div class="stat"><div class="stat-val">2,700 m</div><div class="stat-label">Village Alt.</div></div>
</div>

<script>
const locations = [
  {
    id: "shangrila",
    name: "Shangri-La Old Town",
    chinese: "香格里拉古城",
    lat: 27.8297,
    lng: 99.7069,
    day: 0,
    type: "transport",
    color: "#f97316",
    icon: "🏙️",
    elevation: "3,160 m",
    desc: "Your starting point. Spend the night here before departing to acclimatize to the altitude. The old town (Dukezong) has great Tibetan restaurants and cozy hostels.",
    tags: ["Start Point", "Acclimatize", "Old Town"]
  },
  {
    id: "busstation",
    name: "Shangri-La Bus Station",
    chinese: "香格里拉客运站",
    lat: 27.8190,
    lng: 99.7150,
    day: 0,
    type: "transport",
    color: "#f97316",
    icon: "🚌",
    elevation: "3,160 m",
    desc: "If taking the budget route, catch the bus to Luoji here. Departures at 10:30 and 15:00 daily. Fare: CNY 35.",
    tags: ["Bus", "CNY 35", "2.5 hrs to Luoji"]
  },
  {
    id: "luoji",
    name: "Luoji Township",
    chinese: "洛吉乡",
    lat: 27.9500,
    lng: 99.9800,
    day: 0,
    type: "transport",
    color: "#f97316",
    icon: "🛤️",
    elevation: "~2,800 m",
    desc: "Transfer point if taking the bus. From here, hire a local driver for the final 38 km of unpaved mountain road into Niru Village (CNY 250–300).",
    tags: ["Transfer", "Hire Driver", "Unpaved Road"]
  },
  {
    id: "niru",
    name: "Niru Village",
    chinese: "尼汝村",
    lat: 28.0167,
    lng: 100.1333,
    day: 1,
    type: "village",
    color: "#22c55e",
    icon: "🏡",
    elevation: "2,700 m",
    desc: "Your base for both days. A pristine Tibetan village praised by UNESCO as 'No. 1 Village in the World'. Traditional wooden houses, prayer flags, and virtually no light pollution for incredible stargazing.",
    tags: ["Homestay", "CNY 150–200/night", "UNESCO"]
  },
  {
    id: "waterfall_trail",
    name: "Colorful Waterfall Trail",
    chinese: "七彩瀑布步道",
    lat: 28.0350,
    lng: 100.1550,
    day: 1,
    type: "nature",
    color: "#3b82f6",
    icon: "🥾",
    elevation: "2,700 – 2,900 m",
    desc: "A gentle 6 km boardwalk each way through virgin forest along the Niru River. Wildflowers carpet the forest floor in spring. Round trip takes 3–4 hours.",
    tags: ["Easy", "6 km each way", "Boardwalk"]
  },
  {
    id: "waterfall",
    name: "Qicai (Colorful) Waterfall",
    chinese: "七彩瀑布",
    lat: 28.0530,
    lng: 100.1700,
    day: 1,
    type: "nature",
    color: "#3b82f6",
    icon: "💧",
    elevation: "~2,900 m",
    desc: "The trail's stunning finale — 30 m high and stretching 330 m wide. Mineral-rich water creates rainbow-like color bands across the rock face. Rivals Jiuzhaigou in beauty.",
    tags: ["Iconic", "30 m High", "330 m Wide"]
  },
  {
    id: "dingru",
    name: "Dingru Lake",
    chinese: "丁如湖",
    lat: 28.0700,
    lng: 100.1100,
    day: 2,
    type: "nature",
    color: "#3b82f6",
    icon: "🏔️",
    elevation: "~3,500 m",
    desc: "A sacred alpine glacial lake with impossibly clear water, surrounded by old-growth forest. Drinking the water is welcomed, but washing hands in the lake is forbidden. Purple rhododendrons bloom along the descent in spring.",
    tags: ["Sacred Lake", "Challenging", "5–6 hrs RT"]
  },
  {
    id: "nanbao",
    name: "Nanbao Pasture",
    chinese: "南宝牧场",
    lat: 28.0450,
    lng: 100.0900,
    day: 2,
    type: "nature",
    color: "#3b82f6",
    icon: "🌿",
    elevation: "~3,200 m",
    desc: "A vast alpine meadow at the base of snow peaks. Yaks graze freely across the fresh spring grass. A gentler alternative to Dingru Lake — very photogenic.",
    tags: ["Moderate", "3–4 hrs RT", "Yaks"]
  },
  {
    id: "monastery",
    name: "Local Tibetan Monastery",
    chinese: "藏传佛教寺院",
    lat: 28.0180,
    lng: 100.1380,
    day: 2,
    type: "culture",
    color: "#a855f7",
    icon: "🛕",
    elevation: "2,700 m",
    desc: "A small but atmospheric Tibetan Buddhist temple in the village. Locals come daily to pray and spin the prayer wheels. Walk clockwise around the site as a sign of respect.",
    tags: ["Buddhist", "Prayer Wheels", "Respectful Visit"]
  }
];

// Init map
const map = L.map('map', {
  center: [27.95, 99.95],
  zoom: 10,
  zoomControl: false
});

L.control.zoom({ position: 'topright' }).addTo(map);

// Tile layer — terrain style
L.tileLayer('https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png', {
  maxZoom: 17,
  attribution: '&copy; OpenTopoMap contributors'
}).addTo(map);

// Drive route (Shangri-La → Luoji → Niru)
const driveRoute = [
  [27.8297, 99.7069],
  [27.8450, 99.7400],
  [27.8700, 99.7900],
  [27.9000, 99.8500],
  [27.9200, 99.9100],
  [27.9500, 99.9800],
  [27.9700, 100.0200],
  [27.9900, 100.0700],
  [28.0050, 100.1000],
  [28.0167, 100.1333]
];

const driveLine = L.polyline(driveRoute, {
  color: '#f97316', weight: 3.5, opacity: 0.7, dashArray: '8, 8',
  className: 'drive-route'
}).addTo(map);

// Hiking trail (Niru → Waterfall)
const waterfallTrail = [
  [28.0167, 100.1333],
  [28.0250, 100.1420],
  [28.0350, 100.1550],
  [28.0440, 100.1620],
  [28.0530, 100.1700]
];

L.polyline(waterfallTrail, {
  color: '#3b82f6', weight: 3, opacity: 0.6, dashArray: '5, 10',
}).addTo(map);

// Hiking trail (Niru → Dingru Lake)
const dingruTrail = [
  [28.0167, 100.1333],
  [28.0300, 100.1250],
  [28.0450, 100.1180],
  [28.0580, 100.1130],
  [28.0700, 100.1100]
];

L.polyline(dingruTrail, {
  color: '#3b82f6', weight: 3, opacity: 0.5, dashArray: '5, 10',
}).addTo(map);

// Hiking trail (Niru → Nanbao)
const nanbaoTrail = [
  [28.0167, 100.1333],
  [28.0250, 100.1150],
  [28.0350, 100.1000],
  [28.0450, 100.0900]
];

L.polyline(nanbaoTrail, {
  color: '#3b82f6', weight: 3, opacity: 0.5, dashArray: '5, 10',
}).addTo(map);

// Markers
const markers = {};
locations.forEach(loc => {
  const markerHtml = `<div class="custom-marker" style="background:${loc.color}">${loc.icon}</div>`;
  const icon = L.divIcon({ html: markerHtml, className: '', iconSize: [36, 36], iconAnchor: [18, 18] });

  const popup = `
    <div class="popup-title">${loc.name}</div>
    <div class="popup-chinese">${loc.chinese}</div>
    <div class="popup-desc">${loc.desc}</div>
    <div class="popup-detail">
      <strong>Elevation:</strong> ${loc.elevation}<br>
      ${loc.tags.map(t => `<span style="display:inline-block;background:rgba(56,189,248,0.12);color:#7dd3fc;padding:2px 7px;border-radius:8px;font-size:10px;margin:3px 3px 0 0;">${t}</span>`).join('')}
    </div>
  `;

  const marker = L.marker([loc.lat, loc.lng], { icon }).addTo(map).bindPopup(popup, { maxWidth: 280 });
  markers[loc.id] = marker;
});

// Sidebar card rendering
function renderCards(dayFilter) {
  const container = document.getElementById('locations');
  const filtered = dayFilter === 'all' ? locations : locations.filter(l => l.day === parseInt(dayFilter));

  container.innerHTML = filtered.map(loc => `
    <div class="location-card" data-id="${loc.id}">
      <div class="card-top">
        <div class="card-icon" style="background:${loc.color}22; color:${loc.color}">${loc.icon}</div>
        <div>
          <div class="card-title">${loc.name}</div>
          <div class="card-subtitle">${loc.chinese} · ${loc.elevation}</div>
        </div>
      </div>
      <div class="card-desc">${loc.desc}</div>
      <div class="card-tags">${loc.tags.map(t => `<span class="card-tag">${t}</span>`).join('')}</div>
    </div>
  `).join('');

  // Card click events
  container.querySelectorAll('.location-card').forEach(card => {
    card.addEventListener('click', () => {
      container.querySelectorAll('.location-card').forEach(c => c.classList.remove('active'));
      card.classList.add('active');
      const loc = locations.find(l => l.id === card.dataset.id);
      map.flyTo([loc.lat, loc.lng], 13, { duration: 1.2 });
      markers[loc.id].openPopup();
    });
  });
}

// Day tab switching
document.querySelectorAll('.day-tab').forEach(tab => {
  tab.addEventListener('click', () => {
    document.querySelectorAll('.day-tab').forEach(t => t.classList.remove('active'));
    tab.classList.add('active');
    renderCards(tab.dataset.day);

    if (tab.dataset.day === 'all' || tab.dataset.day === '0') {
      map.flyToBounds(driveLine.getBounds().pad(0.2), { duration: 1 });
    } else if (tab.dataset.day === '1') {
      map.flyTo([28.035, 100.15], 12, { duration: 1 });
    } else if (tab.dataset.day === '2') {
      map.flyTo([28.045, 100.12], 12, { duration: 1 });
    }
  });
});

// Sidebar toggle
const sidebar = document.getElementById('sidebar');
const toggle = document.getElementById('sidebarToggle');
let sidebarOpen = true;

toggle.addEventListener('click', () => {
  sidebarOpen = !sidebarOpen;
  sidebar.classList.toggle('collapsed');
  toggle.classList.toggle('open', sidebarOpen);
  toggle.classList.toggle('closed', !sidebarOpen);
  toggle.innerHTML = sidebarOpen ? '◀' : '▶';
  setTimeout(() => map.invalidateSize(), 350);
});

// Initial render
renderCards('all');

// Fit all markers
const allLatLngs = locations.map(l => [l.lat, l.lng]);
map.fitBounds(allLatLngs, { padding: [60, 60] });
</script>
</body>
</html>
