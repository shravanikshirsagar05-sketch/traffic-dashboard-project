# traffic-dashboard-project
Learning project - building a responsive web dashboard
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>NiyantranaX - Traffic Management</title>

<!-- Leaflet + Chart.js -->
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>

<style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    html,body { height:100%; }
    body {
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        background: #0a0f1c;
        color: #e6eef8;
        overflow-x: hidden;
        -webkit-font-smoothing:antialiased;
    }

    /* Sidebar */
    .sidebar {
        width: 260px;
        height: 100vh;
        background: #111423;
        position: fixed;
        left: 0;
        top: 0;
        z-index: 1200;
        padding: 22px 14px;
        border-right: 1px solid rgba(255,255,255,0.03);
    }
    .logo { padding-bottom:18px; border-bottom:1px solid rgba(255,255,255,0.03); margin-bottom:18px; }
    .logo h2 { color:#2fb0ff; font-size:20px; margin-left:6px; font-weight:700; }
    .logo .subtitle { font-size:12px; color:#9aa6b3; margin-top:6px; }

    .nav-menu { list-style:none; margin-top:18px; padding-left:0; }
    .nav-item { margin-bottom:8px; }
    .nav-link {
        display:flex; align-items:center; gap:10px;
        padding:10px 12px; border-radius:8px;
        color:#b9c7d6; text-decoration:none; cursor:pointer;
    }
    .nav-link:hover, .nav-link.active { background:#111827; color:#2fb0ff; border-left:4px solid #2fb0ff; padding-left:8px; }
    .nav-icon { font-size:18px; width:26px; display:inline-block; }

    /* Main content */
    .main-content { margin-left:260px; min-height:100vh; padding-bottom:60px; transition:margin-left .2s ease; }
    .header {
        padding:22px 28px; border-bottom:1px solid rgba(255,255,255,0.03);
        display:flex; justify-content:space-between; align-items:center;
    }
    .header h1 { font-size:28px; color:#fff; font-weight:800; }
    .header .subtitle { color:#9aa6b3; margin-top:6px; font-size:13px; }

    .time { color:#9aa6b3; font-size:13px; }

    /* Cards */
    .card {
        background:#0f1720;
        border-radius:12px; border:1px solid rgba(255,255,255,0.03);
        padding:18px; box-shadow: 0 6px 18px rgba(2,6,23,0.45);
        margin-bottom:18px;
    }

    .metrics-grid { display:grid; grid-template-columns: repeat(auto-fit, minmax(220px,1fr)); gap:18px; margin:16px 0; }
    .metric-card { padding:18px; border-radius:10px; background:linear-gradient(180deg,#0f1820,#101522); border:1px solid rgba(255,255,255,0.03); }
    .metric-title { color:#9aa6b3; font-weight:600; margin-bottom:8px; }
    .metric-value { font-weight:800; font-size:28px; }

    /* Map container */
    .map-container {
        position:relative;
        min-height:480px;
        height: calc(60vh);
        max-height:760px;
        border-radius:10px;
        overflow:hidden;
    }
    #map, #mapDetailed { width:100%; height:100%; display:block; }

    .map-legend {
        position:absolute; bottom:18px; right:18px;
        background:rgba(10,14,20,0.85); padding:10px; border-radius:8px;
        border:1px solid rgba(255,255,255,0.03); z-index:1500; color:#c9d9ea; font-size:13px;
    }
    .legend-item { display:flex; gap:8px; align-items:center; margin-bottom:6px; }
    .legend-dot { width:12px; height:12px; border-radius:50%; }

    /* Signal Control redesign */
    .signal-grid { display:grid; grid-template-columns: 360px 1fr 280px; gap:18px; align-items:start; }
    .signal-panel { padding:18px; border-radius:10px; background:#0f1720; border:1px solid rgba(255,255,255,0.03); }
    .signal-panel h3 { font-size:18px; margin-bottom:12px; color:#fff; }
    .select, select { width:100%; padding:10px 12px; border-radius:8px; border:1px solid rgba(255,255,255,0.04); background:#09101a; color:#e6eef8; }
    .status-badge { display:inline-block; padding:8px 12px; border-radius:8px; font-weight:700; color:#fff; margin-top:10px; }
    .status-optimal { background:#18b26b; }
    .status-warning { background:#ee8b34; }
    .status-congested { background:#f06363; }

    .signal-controls { display:flex; gap:10px; align-items:center; margin-top:12px; flex-wrap:wrap; }
    .btn { padding:10px 14px; border-radius:8px; font-weight:700; border:none; cursor:pointer; box-shadow:0 6px 18px rgba(2,6,23,0.25); }
    .btn-green { background:#1dbf7a; color:#04211a; }
    .btn-red { background:#ff6b6b; color:#2b0d0d; }
    .btn-blue { background:#3aa0ff; color:#072233; }
    .btn-muted { background:#111827; color:#c9d9ea; border:1px solid rgba(255,255,255,0.02); }

    /* Analytics grid */
    .analytics-grid { display:grid; grid-template-columns: repeat(2, 1fr); gap:18px; margin-top:10px; }
    .chart-card { padding:14px; border-radius:10px; background:#0f1720; border:1px solid rgba(255,255,255,0.03); min-height:290px; }

    .intersection-table { width:100%; border-collapse:collapse; margin-top:12px; color:#d6e6f6; font-size:14px; }
    .intersection-table th, .intersection-table td { padding:12px; border-bottom:1px solid rgba(255,255,255,0.03); text-align:left;}
    .intersection-table th { color:#9aa6b3; background:transparent; font-weight:700; position:sticky; top:0; }

    /* Safety Reports Styling - Fixed to match the image design */
    .safety-reports-container {
        max-width: 800px;
        margin: 0 auto;
        padding: 20px 28px;
    }
    
    .safety-form {
        background: #1a1f2e;
        border-radius: 12px;
        padding: 24px;
        margin-bottom: 24px;
        border: 1px solid rgba(255,255,255,0.05);
    }
    
    .form-group {
        margin-bottom: 20px;
    }
    
    .form-label {
        color: #9aa6b3;
        font-weight: 600;
        display: block;
        margin-bottom: 8px;
        font-size: 14px;
    }
    
    .form-input, .form-select, .form-textarea {
        width: 100%;
        padding: 12px 16px;
        border-radius: 8px;
        border: 1px solid rgba(255,255,255,0.1);
        background: #0f1720;
        color: #e6eef8;
        font-size: 14px;
        transition: border-color 0.3s ease;
    }
    
    .form-input:focus, .form-select:focus, .form-textarea:focus {
        outline: none;
        border-color: #2fb0ff;
    }
    
    .form-textarea {
        min-height: 100px;
        resize: vertical;
    }
    
    .severity-buttons {
        display: flex;
        gap: 12px;
        margin-top: 8px;
    }
    
    .severity-btn {
        flex: 1;
        padding: 12px 16px;
        border-radius: 8px;
        border: 1px solid rgba(255,255,255,0.1);
        background: transparent;
        color: #9aa6b3;
        cursor: pointer;
        font-weight: 600;
        font-size: 14px;
        transition: all 0.3s ease;
        display: flex;
        align-items: center;
        justify-content: center;
        gap: 8px;
    }
    
    .severity-btn:hover {
        background: rgba(255,255,255,0.05);
    }
    
    .severity-btn.selected.low {
        background: #22c55e;
        color: #ffffff;
        border-color: #22c55e;
    }
    
    .severity-btn.selected.medium {
        background: #f59e0b;
        color: #ffffff;
        border-color: #f59e0b;
    }
    
    .severity-btn.selected.high {
        background: #ef4444;
        color: #ffffff;
        border-color: #ef4444;
    }
    
    .submit-btn {
        width: 100%;
        padding: 14px;
        border-radius: 8px;
        border: none;
        background: #2fb0ff;
        color: #ffffff;
        font-weight: 700;
        font-size: 16px;
        cursor: pointer;
        transition: background-color 0.3s ease;
    }
    
    .submit-btn:hover {
        background: #1e90ff;
    }
    
    .clear-btn {
        background: #374151;
        color: #d1d5db;
        margin-left: 12px;
        padding: 14px 24px;
        border: none;
        border-radius: 8px;
        font-weight: 600;
        cursor: pointer;
        transition: background-color 0.3s ease;
    }
    
    .clear-btn:hover {
        background: #4b5563;
    }
    
    .recent-reports {
        background: #1a1f2e;
        border-radius: 12px;
        padding: 24px;
        border: 1px solid rgba(255,255,255,0.05);
    }
    
    .reports-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 20px;
    }
    
    .reports-title {
        color: #ffffff;
        font-size: 18px;
        font-weight: 700;
        margin: 0;
    }
    
    .reports-subtitle {
        color: #9aa6b3;
        font-size: 13px;
        margin-top: 4px;
    }
    
    .refresh-btn {
        padding: 8px 16px;
        border-radius: 6px;
        border: none;
        background: #374151;
        color: #d1d5db;
        font-weight: 600;
        cursor: pointer;
        font-size: 14px;
        transition: background-color 0.3s ease;
    }
    
    .refresh-btn:hover {
        background: #4b5563;
    }
    
    .reports-list {
        max-height: 400px;
        overflow-y: auto;
    }
    
    .report-item {
        display: flex;
        gap: 12px;
        padding: 16px;
        background: #0f1720;
        border-radius: 8px;
        margin-bottom: 12px;
        border-left: 4px solid #2fb0ff;
    }
    
    .report-severity-dot {
        width: 12px;
        height: 12px;
        border-radius: 50%;
        margin-top: 4px;
        flex-shrink: 0;
    }
    
    .report-content {
        flex: 1;
    }
    
    .report-title {
        color: #ffffff;
        font-weight: 700;
        margin-bottom: 4px;
        font-size: 14px;
    }
    
    .report-description {
        color: #9aa6b3;
        font-size: 13px;
        line-height: 1.4;
        margin-bottom: 8px;
    }
    
    .report-meta {
        color: #6b7280;
        font-size: 12px;
    }

    /* User type selector in sidebar bottom */
    .user-type-selector {
        position: absolute;
        bottom: 20px;
        left: 14px;
        right: 14px;
        z-index: 1300;
    }
    
    .user-select {
        width: 100%;
        padding: 10px 12px;
        border-radius: 8px;
        border: 1px solid rgba(255,255,255,0.1);
        background: #0f1720;
        color: #e6eef8;
        font-size: 14px;
        font-weight: 600;
    }
    
    .user-select:focus {
        outline: none;
        border-color: #2fb0ff;
    }

    /* small devices */
    @media (max-width: 980px) {
        .signal-grid { grid-template-columns: 1fr; }
        .analytics-grid { grid-template-columns:1fr; }
        .main-content { margin-left:0; padding-left:16px; padding-right:16px; }
        .sidebar { position:relative; width:100%; height:auto; }
        .safety-reports-container { padding: 16px; }
    }

    /* utility: admin and citizen visibility */
    .admin-only { display: initial; }
    .citizen-only { display: none; }
    
    /* Ensure pages are properly positioned */
    .page {
        margin: 16px 28px;
    }

    /* Loading Screen Animation */
    .loading-screen {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background: linear-gradient(135deg, #0a0f1c 0%, #111423 50%, #0f1720 100%);
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        z-index: 9999;
        transition: opacity 0.8s ease-out;
    }

    .loading-screen.fade-out {
        opacity: 0;
        pointer-events: none;
    }

    .logo-container {
        position: relative;
        margin-bottom: 40px;
    }

    .nx-logo {
        font-family: 'Georgia', serif;
        font-size: 180px;
        font-weight: 900;
        color: #2fb0ff;
        text-shadow: 0 0 30px rgba(47, 176, 255, 0.5), 0 0 60px rgba(47, 176, 255, 0.3);
        animation: logoReveal 2s ease-out forwards;
        opacity: 0;
        transform: scale(0.3);
    }

    @keyframes logoReveal {
        0% {
            opacity: 0;
            transform: scale(0.3);
        }
        50% {
            opacity: 0.7;
            transform: scale(1.1);
        }
        100% {
            opacity: 1;
            transform: scale(1);
        }
    }

    .logo-glow {
        position: absolute;
        top: 50%;
        left: 50%;
        width: 200px;
        height: 200px;
        background: radial-gradient(circle, rgba(47, 176, 255, 0.3) 0%, transparent 70%);
        transform: translate(-50%, -50%);
        animation: glow 2s ease-in-out infinite alternate;
        border-radius: 50%;
    }

    @keyframes glow {
        from { transform: translate(-50%, -50%) scale(0.8); opacity: 0.5; }
        to { transform: translate(-50%, -50%) scale(1.2); opacity: 0.8; }
    }

    .loading-title {
        font-size: 24px;
        color: #9aa6b3;
        font-weight: 600;
        margin-top: 20px;
        animation: titleSlide 1.5s ease-out 1s forwards;
        opacity: 0;
        transform: translateY(30px);
    }

    @keyframes titleSlide {
        to {
            opacity: 1;
            transform: translateY(0);
        }
    }

    .loading-subtitle {
        font-size: 14px;
        color: #6b7280;
        margin-top: 10px;
        animation: titleSlide 1.5s ease-out 1.5s forwards;
        opacity: 0;
        transform: translateY(30px);
    }

    .loading-dots {
        margin-top: 30px;
        display: flex;
        gap: 8px;
        animation: titleSlide 1.5s ease-out 2s forwards;
        opacity: 0;
    }

    .dot {
        width: 8px;
        height: 8px;
        background: #2fb0ff;
        border-radius: 50%;
        animation: dotPulse 1.5s infinite ease-in-out;
    }

    .dot:nth-child(2) { animation-delay: 0.2s; }
    .dot:nth-child(3) { animation-delay: 0.4s; }

    @keyframes dotPulse {
        0%, 80%, 100% { transform: scale(0.8); opacity: 0.5; }
        40% { transform: scale(1.2); opacity: 1; }
    }
</style>
</head>
<body>

<!-- Loading Screen -->
<div class="loading-screen" id="loadingScreen">
    <div class="logo-container">
        <div class="logo-glow"></div>
        <div class="nx-logo">NX</div>
    </div>
    <div class="loading-title">NiyantranaX</div>
    <div class="loading-subtitle">Traffic Management</div>
    <div class="loading-dots">
        <div class="dot"></div>
        <div class="dot"></div>
        <div class="dot"></div>
    </div>
</div>

<!-- Sidebar -->
<div class="sidebar" id="sidebar">
    <div class="logo">
        <h2>NiyantranaX</h2>
        <div class="subtitle">Traffic Management</div>
    </div>
    <ul class="nav-menu">
        <li class="nav-item">
            <a class="nav-link active" onclick="showPage('overview', event)">
                <span class="nav-icon">📊</span>Dashboard
            </a>
        </li>
        <li class="nav-item">
            <a class="nav-link" onclick="showPage('traffic-map', event)">
                <span class="nav-icon">🗺️</span>Traffic Map
            </a>
        </li>
        <li class="nav-item admin-only">
            <a class="nav-link" onclick="showPage('analytics', event)">
                <span class="nav-icon">📈</span>Analytics
            </a>
        </li>
        <li class="nav-item admin-only">
            <a class="nav-link" onclick="showPage('signal-control', event)">
                <span class="nav-icon">🚦</span>Signal Control
            </a>
        </li>
        <li class="nav-item admin-only">
            <a class="nav-link" onclick="showPage('alerts', event)">
                <span class="nav-icon">⚠️</span>Alerts
            </a>
        </li>
        <li class="nav-item admin-only">
            <a class="nav-link" onclick="showPage('reports', event)">
                <span class="nav-icon">📋</span>Reports
            </a>
        </li>

        <!-- Safety Reports nav visible only in citizen view -->
        <li class="nav-item citizen-only">
            <a class="nav-link" onclick="showPage('safety-reports', event)">
                <span class="nav-icon">🚨</span>Safety Reports
            </a>
        </li>
    </ul>
    
    <!-- User Type Selector moved to sidebar bottom -->
    <div class="user-type-selector">
        <select id="userType" class="user-select" onchange="switchUserType()">
            <option value="admin">Admin (RTO Controller)</option>
            <option value="citizen">Citizen View</option>
        </select>
    </div>
</div>

<!-- Main content -->
<div class="main-content" id="mainContent">
    <div class="header">
        <div>
            <h1>Traffic Control Center</h1>
            <div class="subtitle">Real-time intersection monitoring and control - Odisha</div>
        </div>
        <div style="display:flex; gap:18px; align-items:center;">
            <div class="time" id="currentTime"></div>
        </div>
    </div>

    <!-- Overview -->
    <div id="overview" class="card page active">
        <div class="metrics-grid">
            <div class="metric-card">
                <div class="metric-title">Traffic Efficiency</div>
                <div class="metric-value" id="trafficEfficiency">--%</div>
                <div style="color:#8ad9a6; font-size:13px; margin-top:6px;">↑ +2.4% from last hour</div>
            </div>
            <div class="metric-card">
                <div class="metric-title">Active Signals</div>
                <div class="metric-value"><span id="activeSignals">--</span>/50</div>
                <div style="color:#9aa6b3; font-size:13px; margin-top:6px;" id="maintenanceText">All systems operational</div>
            </div>
            <div class="metric-card">
                <div class="metric-title">Avg Wait Time</div>
                <div class="metric-value" id="avgWaitTime">-- min</div>
                <div style="color:#9aa6b3; font-size:13px; margin-top:6px;">↓ -10.2% reduction</div>
            </div>
            <div class="metric-card">
                <div class="metric-title">System Status</div>
                <div id="systemStatus" class="metric-value">--</div>
                <div style="color:#9aa6b3; font-size:13px; margin-top:6px;" id="systemStatusText">All systems operational</div>
            </div>
        </div>

        <div class="card">
            <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:12px;">
                <div>
                    <h3 style="margin-bottom:4px;">Real-time Traffic Map</h3>
                    <div style="color:#98aabf; font-size:13px;">Live traffic conditions across major Odisha intersections</div>
                </div>
                <div><button class="btn btn-muted" onclick="refreshMap()">🔄 Refresh</button></div>
            </div>
            <div class="map-container" id="overviewMapWrap"><div id="map"></div>
                <div class="map-legend">
                    <div class="legend-item"><div class="legend-dot" style="background:#18b26b"></div><div>Optimal Flow</div></div>
                    <div class="legend-item"><div class="legend-dot" style="background:#ee8b34"></div><div>Moderate Traffic</div></div>
                    <div class="legend-item"><div class="legend-dot" style="background:#f06363"></div><div>Heavy Congestion</div></div>
                </div>
            </div>
        </div>
    </div>

    <!-- Traffic Map Page (always available) -->
    <div id="traffic-map" class="card page" style="display:none;">
        <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:12px;">
            <div>
                <h3 style="margin-bottom:4px;">Interactive Traffic Map</h3>
                <div style="color:#98aabf; font-size:13px;">Detailed view of all intersections in Odisha</div>
            </div>
            <div><button class="btn btn-muted" onclick="refreshDetailedMap()">🔄 Refresh</button></div>
        </div>
        <div class="map-container" id="detailedMapWrap"><div id="mapDetailed"></div>
            <div class="map-legend">
                <div class="legend-item"><div class="legend-dot" style="background:#18b26b"></div><div>Optimal Flow</div></div>
                <div class="legend-item"><div class="legend-dot" style="background:#ee8b34"></div><div>Moderate Traffic</div></div>
                <div class="legend-item"><div class="legend-dot" style="background:#f06363"></div><div>Heavy Congestion</div></div>
            </div>
        </div>
    </div>

    <!-- Analytics (admin only) -->
    <div id="analytics" class="card page admin-only" style="display:none;">
        <div style="display:flex; justify-content:space-between; align-items:center; gap:12px; margin-bottom:10px;">
            <div>
                <h2 style="font-size:20px;color:#fff;margin-bottom:6px;">Analytics</h2>
                <div style="color:#9aa6b3;">Detailed charts and intersection performance</div>
            </div>
            <div><button class="btn btn-muted" onclick="exportStats()">Generate Report</button></div>
        </div>

        <div class="analytics-grid">
            <div class="chart-card"><h4 style="color:#fff;margin-bottom:8px;">Average Waiting Time</h4><canvas id="waitingTimeChart"></canvas></div>
            <div class="chart-card"><h4 style="color:#fff;margin-bottom:8px;">Queue Length Analysis</h4><canvas id="queueLengthChart"></canvas></div>
            <div class="chart-card"><h4 style="color:#fff;margin-bottom:8px;">Traffic Flow Trends</h4><canvas id="trafficFlowChart"></canvas></div>
            <div class="chart-card"><h4 style="color:#fff;margin-bottom:8px;">Intersection Performance</h4><canvas id="performanceChart"></canvas></div>
        </div>

        <div class="card" style="margin-top:16px;">
            <div style="display:flex; justify-content:space-between; align-items:center;">
                <div style="font-weight:800; font-size:16px; color:#fff;">Intersection Statistics</div>
                <div><button class="btn btn-muted" onclick="exportStats()">📥 Export CSV</button></div>
            </div>
            <div style="margin-top:12px; overflow:auto;">
                <table class="intersection-table" id="intersectionTable">
                    <thead>
                        <tr><th>Intersection</th><th>Location</th><th>Avg Wait (s)</th><th>Queue</th><th>Status</th></tr>
                    </thead>
                    <tbody id="intersectionTableBody"></tbody>
                </table>
            </div>
        </div>
    </div>

    <!-- Signal Control (admin only) -->
    <div id="signal-control" class="card page admin-only" style="display:none;">
        <div style="margin-bottom:12px; display:flex; justify-content:space-between; align-items:center;">
            <div><h2 style="margin:0;color:#fff;">Manual Signal Control</h2><div style="color:#98aabf;font-size:13px;">Override automatic signals for emergency or maintenance</div></div>
            <div style="color:#98aabf;font-size:13px;">Note: demo actions only</div>
        </div>
        <div class="signal-grid">
            <div class="signal-panel">
                <h3>Select Intersection</h3>
                <select id="intersectionSelect" class="select"></select>
                <div style="margin-top:12px; color:#9aa6b3;">Current Status:</div>
                <div id="selectedStatus" class="status-badge status-optimal">Optimal</div>
            </div>

            <div class="signal-panel">
                <h3>Manual Override</h3>
                <div class="signal-controls">
                    <button class="btn btn-green" onclick="forceGreen()">Force Green</button>
                    <button class="btn btn-red" onclick="forceRed()">Force Red</button>
                    <button class="btn btn-blue" onclick="setAutoMode()">Auto Mode</button>
                    <div style="margin-left:auto; display:flex; gap:8px; align-items:center;">
                        <div style="color:#9aa6b3;">Duration (s):</div>
                        <input id="overrideDuration" type="number" min="5" value="30" style="width:84px;padding:8px;border-radius:8px;border:1px solid rgba(255,255,255,0.03);background:#08101a;color:#e6eef8;">
                    </div>
                </div>
                <div class="signal-help" style="color:#98aabf; font-size:13px; margin-top:8px;">Force commands temporarily override signal timings.</div>
            </div>

            <div class="signal-panel">
                <h3>Emergency Controls</h3>
                <div style="display:flex;flex-direction:column;gap:12px;margin-top:8px;">
                    <button class="btn btn-red" onclick="emergencyOverride()">🚨 Emergency Override</button>
                    <button class="btn btn-blue" onclick="maintenanceMode()">🔧 Maintenance Mode</button>
                </div>
                <div style="margin-top:12px;color:#9aa6b3;">Emergency override prioritizes emergency vehicle passage.</div>
            </div>
        </div>
    </div>

    <!-- Alerts (admin only) -->
    <div id="alerts" class="card page admin-only" style="display:none;">
        <h3>Active Alerts</h3>
        <div id="alertsPanel" style="margin-top:12px;"></div>
    </div>

    <!-- Reports (admin only) -->
    <div id="reports" class="card page admin-only" style="display:none;">
        <div style="display:flex; justify-content:space-between; align-items:center;">
            <div><h3 style="margin:0;color:#fff;">Traffic Reports</h3><div style="color:#98aabf;font-size:13px;">Daily and weekly summaries</div></div>
            <div><button class="btn btn-muted" onclick="generateReport()">Generate New Report</button></div>
        </div>

        <div style="display:flex; gap:16px; margin-top:12px; flex-wrap:wrap;">
            <div style="flex:1; min-width:360px;">
                <h4 style="color:#fff;">Daily Traffic Summary</h4>
                <canvas id="reportsDailyChart" style="width:100%;height:220px;"></canvas>
            </div>
            <div style="flex:1; min-width:360px;">
                <h4 style="color:#fff;">Weekly Performance</h4>
                <canvas id="reportsWeeklyChart" style="width:100%;height:220px;"></canvas>
            </div>
        </div>
    </div>

    <!-- Safety Reports (citizen only) - Fixed to match image design exactly -->
    <div id="safety-reports" class="page citizen-only" style="display:none;">
        <div class="safety-reports-container">
            <!-- Safety Report Form -->
            <div class="safety-form">
                <div style="margin-bottom: 20px;">
                    <h3 style="color: #ffffff; margin: 0; margin-bottom: 8px;">Report Safety Issues</h3>
                    <p style="color: #9aa6b3; font-size: 13px; margin: 0;">Help fellow travelers by reporting road conditions and hazards</p>
                </div>
                
                <div class="form-group">
                    <label class="form-label">Type of Issue</label>
                    <select id="issueType" class="form-select">
                        <option value="">Select issue type...</option>
                        <option value="🚗 Traffic Accident">🚗 Traffic Accident</option>
                        <option value="🔧 Vehicle Breakdown">🔧 Vehicle Breakdown</option>
                        <option value="🚧 Road Block/Construction">🚧 Road Block/Construction</option>
                        <option value="🌧️ Weather Hazard">🌧️ Weather Hazard</option>
                        <option value="🚦 Heavy Traffic/Congestion">🚦 Heavy Traffic/Congestion</option>
                        <option value="🚑 Emergency Vehicle">🚑 Emergency Vehicle</option>
                        <option value="❗ Other Hazard">❗ Other Hazard</option>
                    </select>
                </div>
                
                <div class="form-group">
                    <label class="form-label">Location</label>
                    <input id="issueLocation" type="text" class="form-input" placeholder="Enter location or intersection name">
                </div>
                
                <div class="form-group">
                    <label class="form-label">Severity Level</label>
                    <div class="severity-buttons">
                        <button type="button" id="sevLow" class="severity-btn" onclick="selectSeverity('low', this)">
                            🟢 Low
                        </button>
                        <button type="button" id="sevMed" class="severity-btn" onclick="selectSeverity('medium', this)">
                            🟡 Medium
                        </button>
                        <button type="button" id="sevHigh" class="severity-btn" onclick="selectSeverity('high', this)">
                            🔴 High
                        </button>
                    </div>
                </div>
                
                <div class="form-group">
                    <label class="form-label">Description</label>
                    <textarea id="issueDescription" class="form-textarea" placeholder="Provide details about the situation to help other travelers..."></textarea>
                </div>
                
                <div style="display: flex; gap: 12px;">
                    <button class="submit-btn" onclick="submitSafetyReport()">Submit Safety Report</button>
                    <button class="clear-btn" onclick="clearSafetyForm()">Clear</button>
                </div>
            </div>
            
            <!-- Recent Safety Reports -->
            <div class="recent-reports">
                <div class="reports-header">
                    <div>
                        <h4 class="reports-title">Recent Safety Reports</h4>
                        <p class="reports-subtitle">Community-reported road conditions and hazards</p>
                    </div>
                    <button class="refresh-btn" onclick="refreshReports()">🔄 Refresh</button>
                </div>
                
                <div class="reports-list" id="safetyReportsList">
                    <!-- Reports will be populated here -->
                </div>
            </div>
        </div>
    </div>

</div>

<script>
/* ============================
   Demo data & state
   ============================ */
const intersections = [
    { id:'ODI-001', name:'Patia Chowk', lat:20.2961, lng:85.8245, avgWait:45, queue:12, status:'optimal' },
    { id:'ODI-002', name:'Bisra Chowk', lat:20.2656, lng:85.8402, avgWait:77, queue:8, status:'optimal' },
    { id:'ODI-003', name:'Trishakti madir Chowk', lat:20.2510, lng:85.8316, avgWait:90, queue:15, status:'congested' },
    { id:'ODI-004', name:'Madhupatna Chowk', lat:20.3270, lng:85.8330, avgWait:38, queue:6, status:'optimal' },
    { id:'ODI-005', name:'Panposh Chowk', lat:20.3140, lng:85.8530, avgWait:60, queue:10, status:'moderate' }
];

let alerts = [
    { id:1, title:'Signal Controller Offline (ODI-003)', severity:'high', desc:'Controller not responding', time: new Date(Date.now()-1000*60*55).toISOString() },
    { id:2, title:'Planned Maintenance ODI-002', severity:'medium', desc:'Maintenance window 10:00-12:00', time: new Date(Date.now()-1000*60*60*6).toISOString() }
];

// Safety reports stored in memory instead of localStorage
let safetyReports = [
    {
        id: 1,
        type: '🚧 Road Block/Construction',
        location: 'Patia Chowk',
        severity: 'medium',
        descr: 'Road construction work blocking the left lane. Traffic is being diverted to the right lane causing delays.',
        time: new Date(Date.now() - 1000 * 60 * 30).toISOString()
    },
    {
        id: 2,
        type: '🚗 Traffic Accident',
        location: 'Madhupatna Chowk',
        severity: 'high',
        descr: 'Minor collision between two vehicles. Emergency services on site. Avoid this route if possible.',
        time: new Date(Date.now() - 1000 * 60 * 120).toISOString()
    },
    {
        id: 3,
        type: '🌧️ Weather Hazard',
        location: 'Bisra Chowk',
        severity: 'low',
        descr: 'Road surface wet due to recent rainfall. Drive carefully and maintain safe distance.',
        time: new Date(Date.now() - 1000 * 60 * 60 * 2).toISOString()
    }
];

/* ============================
   Page nav & admin/citizen toggle
   ============================ */
const pages = document.querySelectorAll('.page');
const navLinks = document.querySelectorAll('.nav-link');
function showPage(id, e) {
    pages.forEach(p => p.style.display = 'none');
    const page = document.getElementById(id);
    if (page) page.style.display = 'block';
    navLinks.forEach(n => n.classList.remove('active'));
    if (e && e.currentTarget) e.currentTarget.classList.add('active');
    if (id === 'traffic-map' || id === 'overview') {
        setTimeout(()=> { if (map) map.invalidateSize(); if (mapDetailed) mapDetailed.invalidateSize(); }, 220);
    }
}

function switchUserType() {
    const mode = document.getElementById('userType').value;
    if (mode === 'citizen') {
        document.querySelectorAll('.admin-only').forEach(el => el.style.display = 'none');
        document.querySelectorAll('.citizen-only').forEach(el => el.style.display = 'block');
        // Clear all nav active states first
        document.querySelectorAll('.nav-link').forEach(link => link.classList.remove('active'));
        // Find and activate the overview nav link (dashboard)
        const overviewNavLink = document.querySelector('.nav-link[onclick*="overview"]');
        if (overviewNavLink) overviewNavLink.classList.add('active');
        showPage('overview');
    } else {
        document.querySelectorAll('.admin-only').forEach(el => el.style.display = 'block');
        document.querySelectorAll('.citizen-only').forEach(el => el.style.display = 'none');
        // Clear all nav active states first
        document.querySelectorAll('.nav-link').forEach(link => link.classList.remove('active'));
        // Find and activate the overview nav link
        const overviewNavLink = document.querySelector('.nav-link[onclick*="overview"]');
        if (overviewNavLink) overviewNavLink.classList.add('active');
        showPage('overview');
    }
}

/* ============================
   Leaflet maps
   ============================ */
let map, mapDetailed, markers = [];
function initMaps() {
    map = L.map('map', { zoomControl: true }).setView([20.2961,85.8245], 12);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 19 }).addTo(map);

    mapDetailed = L.map('mapDetailed', { zoomControl: true }).setView([20.2961,85.8245], 12);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 19 }).addTo(mapDetailed);

    drawIntersections();
}

function drawIntersections() {
    markers.forEach(m => {
        try { if (map.hasLayer(m.marker)) map.removeLayer(m.marker); } catch(e){}
        try { if (mapDetailed.hasLayer(m.marker2)) mapDetailed.removeLayer(m.marker2); } catch(e){}
    });
    markers = [];

    intersections.forEach(i => {
        const color = i.status === 'optimal' ? '#18b26b' : (i.status === 'moderate' ? '#ee8b34' : '#f06363');
        const popup = `<strong>${i.name}</strong><br/>Avg Wait: ${i.avgWait}s<br/>Queue: ${i.queue}`;
        const circle = L.circleMarker([i.lat, i.lng], { radius: Math.max(6, Math.min(20, Math.round(i.queue/1.5))), color, fillColor: color, fillOpacity: 0.8 })
            .bindPopup(popup)
            .addTo(map);
        const circle2 = L.circleMarker([i.lat, i.lng], { radius: Math.max(6, Math.min(20, Math.round(i.queue/1.5))), color, fillColor: color, fillOpacity: 0.8 })
            .bindPopup(popup + '<br/><em>Detailed</em>')
            .addTo(mapDetailed);
        markers.push({ marker: circle, marker2: circle2 });
    });
}

function refreshMap() {
    intersections.forEach(i => {
        i.avgWait = Math.max(8, Math.round(i.avgWait + (Math.random()-0.5)*8));
        i.queue = Math.max(1, Math.round(i.queue + Math.round((Math.random()-0.5)*4)));
        i.status = i.queue > 12 ? 'congested' : (i.queue > 8 ? 'moderate' : 'optimal');
    });
    drawIntersections(); updateIntersectionTable(); refreshMetricsFromIntersections();
}
function refreshDetailedMap() {
    refreshMap();
    const latlngs = intersections.map(i => [i.lat, i.lng]);
    if (latlngs.length) mapDetailed.fitBounds(latlngs, { padding:[40,40] });
}

window.addEventListener('resize', ()=> {
    setTimeout(()=> { if (map) map.invalidateSize(); if (mapDetailed) mapDetailed.invalidateSize(); }, 120);
});

/* ============================
   Charts
   ============================ */
let waitingTimeChart, queueLengthChart, trafficFlowChart, performanceChart, reportsDailyChart, reportsWeeklyChart;
function initCharts() {
    const wtCtx = document.getElementById('waitingTimeChart').getContext('2d');
    waitingTimeChart = new Chart(wtCtx, {
        type: 'line',
        data: { labels: ['6 AM','9 AM','12 PM','3 PM','6 PM','9 PM'], datasets:[{ label:'Avg Wait (s)', data:[45,77,52,64,88,44], tension:0.35, borderWidth:3, borderColor:'#3aa0ff', pointBackgroundColor:'#fff', fill:false }] },
        options: { responsive:true, plugins:{ legend:{ display:true } }, scales:{ x:{ ticks:{ color:'#9aa6b3' } }, y:{ ticks:{ color:'#9aa6b3' } } } }
    });

    const qCtx = document.getElementById('queueLengthChart').getContext('2d');
    const qData = intersections.map(i => i.queue);
    const qColors = intersections.map(i => i.status === 'congested' ? '#f06363' : '#18b26b');
    queueLengthChart = new Chart(qCtx, {
        type: 'bar',
        data: { labels: intersections.map(i=>i.id), datasets:[{ label:'Queue (vehicles)', data:qData, backgroundColor:qColors }] },
        options: { responsive:true, plugins:{ legend:{ display:true } }, scales:{ x:{ ticks:{ color:'#9aa6b3' } }, y:{ ticks:{ color:'#9aa6b3' } } } }
    });

    const tfCtx = document.getElementById('trafficFlowChart').getContext('2d');
    const counts = [intersections.filter(i=>i.status==='optimal').length, intersections.filter(i=>i.status==='moderate').length, intersections.filter(i=>i.status==='congested').length];
    trafficFlowChart = new Chart(tfCtx, {
        type:'doughnut',
        data:{ labels:['Optimal','Warning','Congested'], datasets:[{ data:counts, backgroundColor:['#18b26b','#ee8b34','#f06363'], hoverOffset:8 }] },
        options:{ responsive:true, plugins:{ legend:{ position:'top' } } }
    });

    const pfCtx = document.getElementById('performanceChart').getContext('2d');
    performanceChart = new Chart(pfCtx, {
        type:'radar',
        data:{ labels:['Queue Management','Response Time','Emergency Handling','Signal Uptime','Throughput'], datasets:[{ label:'Performance', data:[84,86,88,92,80], borderColor:'#2fb0ff', backgroundColor:'rgba(47,176,255,0.12)', pointBackgroundColor:'#2fb0ff' }] },
        options:{ responsive:true, scales:{ r:{ ticks:{ color:'#9aa6b3' } } } }
    });

    const rdCtx = document.getElementById('reportsDailyChart').getContext('2d');
    reportsDailyChart = new Chart(rdCtx, {
        type:'line',
        data:{ labels:['Mon','Tue','Wed','Thu','Fri','Sat','Sun'], datasets:[{ label:'Efficiency %', data:[88,92,85,90,87,75,68], borderColor:'#2fbf6e', backgroundColor:'rgba(47,191,110,0.06)', tension:0.3, fill:true }] },
        options:{ responsive:true, scales:{ x:{ ticks:{ color:'#9aa6b3' } }, y:{ ticks:{ color:'#9aa6b3' } } } }
    });
    const rwCtx = document.getElementById('reportsWeeklyChart').getContext('2d');
    reportsWeeklyChart = new Chart(rwCtx, {
        type:'bar',
        data:{ labels:['Week 1','Week 2','Week 3','Week 4'], datasets:[{ label:'Avg Commute Time (min)', data:[25.3,23.7,22.0,22.8], backgroundColor:['#3aa0ff','#3aa0ff','#3aa0ff','#3aa0ff'] }] },
        options:{ responsive:true, scales:{ x:{ ticks:{ color:'#9aa6b3' } }, y:{ ticks:{ color:'#9aa6b3' } } } }
    });
}

/* ============================
   Intersection table & metrics
   ============================ */
function updateIntersectionTable() {
    const tbody = document.getElementById('intersectionTableBody');
    tbody.innerHTML = '';
    intersections.forEach(i => {
        const statusClass = i.status === 'optimal' ? 'status-optimal' : (i.status === 'moderate' ? 'status-warning' : 'status-congested');
        const tr = document.createElement('tr');
        tr.innerHTML = `<td>${i.id} - ${i.name}</td><td>${i.lat.toFixed(4)}, ${i.lng.toFixed(4)}</td><td>${i.avgWait}</td><td>${i.queue}</td><td><span class="status-badge ${statusClass}" style="padding:6px 8px;font-weight:700;font-size:13px;">${i.status}</span></td>`;
        tbody.appendChild(tr);
    });
}

function refreshMetricsFromIntersections() {
    const score = Math.round((intersections.reduce((acc,i)=>acc + (i.status==='optimal' ? 1 : (i.status==='moderate' ? 0.75 : 0.4)),0) / intersections.length) * 10 + 10);
    document.getElementById('trafficEfficiency').innerText = `${score}%`;
    document.getElementById('avgWaitTime').innerText = `${Math.round(intersections.reduce((s,i)=>s+i.avgWait,0)/intersections.length/60)} min`;
    
    const congestedCount = intersections.filter(i=>i.status==='congested').length;
    const activeSignals = 50 - congestedCount;
    
    document.getElementById('activeSignals').innerText = `${activeSignals}`;
    
    // Update maintenance text based on congested signals
    const maintenanceElement = document.getElementById('maintenanceText');
    if (congestedCount === 0) {
        maintenanceElement.innerText = 'All systems operational';
        maintenanceElement.style.color = '#8ad9a6';
    } else {
        maintenanceElement.innerText = `${congestedCount} signal${congestedCount > 1 ? 's' : ''} in maintenance`;
        maintenanceElement.style.color = '#9aa6b3';
    }
    
    // Update system status and its subtitle text
    const hasCongestion = intersections.some(i=>i.status==='congested');
    const systemStatusElement = document.getElementById('systemStatus');
    const systemStatusTextElement = document.getElementById('systemStatusText');
    
    if (hasCongestion) {
        systemStatusElement.innerText = 'Degraded';
        systemStatusTextElement.innerText = 'Traffic congestion detected';
        systemStatusTextElement.style.color = '#ee8b34';
    } else {
        systemStatusElement.innerText = 'Optimal';
        systemStatusTextElement.innerText = 'All systems operational';
        systemStatusTextElement.style.color = '#8ad9a6';
    }

    if (queueLengthChart) {
        queueLengthChart.data.datasets[0].data = intersections.map(i=>i.queue);
        queueLengthChart.data.datasets[0].backgroundColor = intersections.map(i => i.status==='congested' ? '#f06363' : '#18b26b');
        queueLengthChart.update();
    }
    if (trafficFlowChart) {
        trafficFlowChart.data.datasets[0].data = [
            intersections.filter(i=>i.status==='optimal').length,
            intersections.filter(i=>i.status==='moderate').length,
            intersections.filter(i=>i.status==='congested').length
        ];
        trafficFlowChart.update();
    }
}

/* ============================
   Alerts rendering
   ============================ */
function renderAlerts() {
    const panel = document.getElementById('alertsPanel');
    panel.innerHTML = '';
    if (!alerts.length) panel.innerHTML = '<div style="color:#9aa6b3;padding:12px;">No active alerts</div>';
    alerts.forEach(a => {
        const el = document.createElement('div');
        el.className = 'card';
        el.style.marginBottom = '10px';
        el.innerHTML = `<div style="display:flex;justify-content:space-between;align-items:center;">
            <div><strong style="color:#fff">${a.title}</strong><div style="color:#9aa6b3;font-size:13px;margin-top:6px">${a.desc}</div></div>
            <div style="color:#9aa6b3;font-size:13px">${timeAgo(a.time)}</div>
        </div>`;
        panel.appendChild(el);
    });
}

/* ============================
   Signal control demo actions
   ============================ */
function populateIntersectionSelect() {
    const sel = document.getElementById('intersectionSelect');
    sel.innerHTML = '';
    intersections.forEach(i => {
        const op = document.createElement('option'); op.value = i.id; op.textContent = `${i.id} - ${i.name}`;
        sel.appendChild(op);
    });
    sel.addEventListener('change', ()=> updateSelectedStatus(sel.value));
    updateSelectedStatus(sel.value || intersections[0].id);
}
function updateSelectedStatus(id) {
    const i = intersections.find(x => x.id === id) || intersections[0];
    const el = document.getElementById('selectedStatus');
    el.className = 'status-badge ' + (i.status==='optimal' ? 'status-optimal' : (i.status==='moderate' ? 'status-warning' : 'status-congested'));
    el.innerText = `${i.status.charAt(0).toUpperCase() + i.status.slice(1)} (${i.queue})`;
}
function forceGreen() {
    const sel = document.getElementById('intersectionSelect');
    const id = sel.value; const dur = Number(document.getElementById('overrideDuration').value) || 30;
    const i = intersections.find(x=>x.id===id);
    if (i) { i.queue = Math.max(1, i.queue - Math.round(Math.random()*8)); i.avgWait = Math.max(5, i.avgWait - Math.round(Math.random()*12)); i.status='optimal'; }
    drawIntersections(); updateIntersectionTable(); refreshMetricsFromIntersections(); updateSelectedStatus(id);
    alerts.unshift({ id:Date.now(), title:`Force Green at ${id}`, severity:'low', desc:`Applied for ${dur}s`, time:new Date().toISOString() });
    renderAlerts();
    alert(`Force Green applied to ${id} for ${dur}s (demo).`);
}
function forceRed() {
    const sel = document.getElementById('intersectionSelect'); const id = sel.value;
    const i = intersections.find(x=>x.id===id);
    if (i) { i.queue += Math.round(Math.random()*6); i.avgWait += Math.round(Math.random()*10); i.status='congested'; }
    drawIntersections(); updateIntersectionTable(); refreshMetricsFromIntersections(); updateSelectedStatus(id);
    alerts.unshift({ id:Date.now(), title:`Force Red at ${id}`, severity:'high', desc:`Applied immediately`, time:new Date().toISOString() });
    renderAlerts();
    alert(`Force Red applied to ${id} (demo).`);
}
function setAutoMode() {
    intersections.forEach(i => { if (i.queue < 12) i.status='optimal'; else if (i.queue < 16) i.status='moderate'; else i.status='congested'; });
    drawIntersections(); updateIntersectionTable(); refreshMetricsFromIntersections();
    alert('Auto mode restored (demo).');
}
function emergencyOverride() {
    intersections.forEach(i => { if (i.status==='congested') { i.queue = Math.max(2, i.queue - 12); i.avgWait = Math.max(5, i.avgWait - 18); i.status='moderate'; }});
    drawIntersections(); updateIntersectionTable(); refreshMetricsFromIntersections();
    alerts.unshift({ id:Date.now(), title:'Emergency Override executed', severity:'high', desc:'Emergency override reduced congested queues', time:new Date().toISOString() });
    renderAlerts();
    alert('Emergency override executed (demo).');
}
function maintenanceMode() {
    alerts.unshift({ id:Date.now(), title:'Maintenance Mode enabled', severity:'medium', desc:'Selected signals placed into maintenance', time:new Date().toISOString() });
    renderAlerts();
    alert('Maintenance mode enabled (demo).');
}

/* ============================
   Safety reports (citizen) - using memory storage
   ============================ */
let selectedSeverity = null;
function selectSeverity(level, el) {
    selectedSeverity = level;
    document.querySelectorAll('#sevLow,#sevMed,#sevHigh').forEach(b => b.classList.remove('selected','low','medium','high'));
    if (el) el.classList.add('selected', level);
}
function submitSafetyReport() {
    const type = document.getElementById('issueType').value;
    const location = document.getElementById('issueLocation').value.trim();
    const descr = document.getElementById('issueDescription').value.trim();
    if (!type || !location || !selectedSeverity) { 
        alert('Please select issue type, location and severity.'); 
        return; 
    }
    const now = new Date();
    const timestamp = now.toISOString();
    const newReport = { id: Date.now(), type, location, severity: selectedSeverity, descr, time: timestamp };
    safetyReports.unshift(newReport);
    clearSafetyForm();
    refreshReports();
    alert('Safety report submitted - thank you!');
}

function refreshReports() {
    const container = document.getElementById('safetyReportsList');
    container.innerHTML = '';
    if (!safetyReports.length) { 
        container.innerHTML = '<div style="color:#9aa6b3; padding:12px; text-align:center;">No reports yet</div>'; 
        return; 
    }
    safetyReports.forEach(r => {
        const item = document.createElement('div');
        item.className = 'report-item';
        const sevColor = r.severity === 'low' ? '#22c55e' : (r.severity === 'medium' ? '#f59e0b' : '#ef4444');
        const timeStr = formatTimestamp(r.time);
        item.innerHTML = `
            <div class="report-severity-dot" style="background:${sevColor};"></div>
            <div class="report-content">
                <div class="report-title">${escapeHtml(r.type)} — ${escapeHtml(r.location)}</div>
                <div class="report-description">${r.descr ? escapeHtml(r.descr) : '<em>No additional details provided</em>'}</div>
                <div class="report-meta">${timeStr}</div>
            </div>
        `;
        container.appendChild(item);
    });
}

function formatTimestamp(iso) {
    const d = new Date(iso);
    const options = { year:'numeric', month:'short', day:'numeric', hour:'numeric', minute:'2-digit' };
    return `Reported on ${d.toLocaleString('en-IN', options)}`;
}

function clearSafetyForm() {
    document.getElementById('issueType').value = '';
    document.getElementById('issueLocation').value = '';
    document.getElementById('issueDescription').value = '';
    selectedSeverity = null;
    document.querySelectorAll('#sevLow,#sevMed,#sevHigh').forEach(b => b.classList.remove('selected','low','medium','high'));
}

function escapeHtml(text) {
    return (text || '').replace(/[&<>"'`]/g, function(m){ 
        return {'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;','`':'&#96;'}[m]; 
    });
}

/* ============================
   Export CSV & reports
   ============================ */
function exportStats() {
    const headers = ['id','name','lat','lng','avgWait','queue','status'];
    const rows = intersections.map(i => headers.map(h=>String(i[h]||'')).map(v=>`"${v.replace(/"/g,'""')}"`).join(','));
    const csv = [headers.join(','), ...rows].join('\n');
    const blob = new Blob([csv], { type: 'text/csv' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a'); a.href = url; a.download = 'intersection_stats.csv'; document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
}
function generateReport() {
    reportsDailyChart.data.datasets[0].data = reportsDailyChart.data.datasets[0].data.map(v => Math.max(60, Math.round(v + (Math.random()-0.5)*6)));
    reportsDailyChart.update();
    reportsWeeklyChart.data.datasets[0].data = reportsWeeklyChart.data.datasets[0].data.map(v => Math.max(20, Math.round(v + (Math.random()-0.5)*2)));
    reportsWeeklyChart.update();
    alert('New report generated (demo).');
}

/* ============================
   Helpers
   ============================ */
function timeAgo(iso) {
    const s = Math.floor((Date.now() - new Date(iso))/1000);
    if (s < 60) return `${s}s ago`;
    if (s < 3600) return `${Math.floor(s/60)}m ago`;
    if (s < 86400) return `${Math.floor(s/3600)}h ago`;
    return `${Math.floor(s/86400)}d ago`;
}

/* ============================
   Initialization
   ============================ */
window.addEventListener('load', () => {
    // Show loading screen for 3.5 seconds, then fade out
    setTimeout(() => {
        const loadingScreen = document.getElementById('loadingScreen');
        loadingScreen.classList.add('fade-out');
        
        // Remove loading screen from DOM after fade completes
        setTimeout(() => {
            loadingScreen.style.display = 'none';
        }, 800);
    }, 3500);

    initMaps();
    initCharts();
    populateIntersectionSelect();
    updateIntersectionTable();
    refreshMetricsFromIntersections();
    renderAlerts();
    refreshReports();

    // clock
    const el = document.getElementById('currentTime');
    (function tick() {
        const now = new Date();
        el.innerText = now.toLocaleTimeString('en-IN', { hour12:true });
        setTimeout(tick, 1000);
    })();

    // periodic demo updates
    setInterval(()=> {
        intersections.forEach(i => {
            i.avgWait = Math.max(20, i.avgWait + Math.round((Math.random()-0.5)*10));
            i.queue = Math.max(1, i.queue + Math.round((Math.random()-0.5)*4));
            i.status = i.queue > 12 ? 'congested' : (i.queue > 8 ? 'moderate' : 'optimal');
        });
        drawIntersections(); updateIntersectionTable(); refreshMetricsFromIntersections();
        if (waitingTimeChart) {
            waitingTimeChart.data.datasets[0].data = waitingTimeChart.data.datasets[0].data.map(v => Math.max(30, Math.round(v + (Math.random()-0.5)*6)));
            waitingTimeChart.update();
        }
    }, 15000);

    // ensure maps rendered properly on load
    setTimeout(()=> { if (map) map.invalidateSize(); if (mapDetailed) mapDetailed.invalidateSize(); }, 400);

    // default mode = admin, but let's check if we need to switch to citizen
    const userType = document.getElementById('userType').value;
    if (userType === 'citizen') {
        switchUserType();
    }
});
</script>
</body>
</html>
