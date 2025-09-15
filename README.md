# traffic-dashboard-project
Learning project - building a responsive web dashboard
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TrafficFlow Pro - Odisha</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>
    <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: #0a0f1c;
            color: #ffffff;
            overflow-x: hidden;
        }

        .sidebar {
            width: 260px;
            height: 100vh;
            background: #1a1f2e;
            position: fixed;
            left: 0;
            top: 0;
            z-index: 1000;
            padding: 20px 0;
            border-right: 1px solid #2d3748;
            transition: transform 0.3s ease-in-out;
        }

        .sidebar.hidden {
            transform: translateX(-100%);
        }

        .logo {
            padding: 0 20px 30px;
            border-bottom: 1px solid #2d3748;
            margin-bottom: 30px;
        }

        .logo h2 {
            color: #4299e1;
            font-size: 18px;
        }

        .nav-menu {
            list-style: none;
        }

        .nav-item {
            margin-bottom: 5px;
        }

        .nav-link {
            display: flex;
            align-items: center;
            padding: 12px 20px;
            color: #a0aec0;
            text-decoration: none;
            transition: all 0.3s;
            cursor: pointer;
        }

        .nav-link:hover, .nav-link.active {
            background: #2d3748;
            color: #4299e1;
        }

        .nav-icon {
            margin-right: 12px;
            font-size: 16px;
        }

        .main-content {
            margin-left: 260px;
            min-height: 100vh;
            background: #0f1419;
            transition: margin-left 0.3s ease-in-out;
        }

        .main-content.expanded {
            margin-left: 0;
        }

        .header {
            background: #1a1f2e;
            padding: 20px 30px;
            border-bottom: 1px solid #2d3748;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .time {
            color: #718096;
            font-size: 14px;
        }

        .metrics-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            padding: 30px;
        }

        .metric-card {
            background: linear-gradient(135deg, #1a202c, #2d3748);
            border: 1px solid #2d3748;
            border-radius: 12px;
            padding: 24px;
            position: relative;
            overflow: hidden;
        }

        .metric-card::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            height: 3px;
            background: linear-gradient(90deg, #4299e1, #63b3ed);
        }

        .metric-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
        }

        .metric-title {
            color: #a0aec0;
            font-size: 14px;
            font-weight: 500;
        }

        .metric-icon {
            color: #4299e1;
            font-size: 20px;
        }

        .metric-value {
            font-size: 32px;
            font-weight: 700;
            color: #ffffff;
            margin-bottom: 8px;
        }

        .metric-change {
            font-size: 12px;
            color: #48bb78;
        }

        .metric-change.negative {
            color: #f56565;
        }

        .status-optimal {
            color: #48bb78;
        }

        .status-warning {
            color: #ed8936;
        }

        .status-congested {
            color: #f56565;
        }

        .content-section {
            padding: 0 30px 30px;
        }

        .section-card {
            background: #1a202c;
            border: 1px solid #2d3748;
            border-radius: 12px;
            margin-bottom: 30px;
            overflow: hidden;
        }

        .section-header {
            padding: 20px 24px;
            border-bottom: 1px solid #2d3748;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .section-title {
            font-size: 18px;
            font-weight: 600;
            color: #ffffff;
        }

        .section-subtitle {
            color: #718096;
            font-size: 14px;
            margin-top: 4px;
        }

        .map-container {
            height: 500px;
            position: relative;
        }

        #map, #mapDetailed {
            height: 100%;
            width: 100%;
        }

        .controls-panel {
            background: #2d3748;
            padding: 20px;
            display: none;
        }

        .controls-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
        }

        .control-group {
            background: #1a202c;
            padding: 16px;
            border-radius: 8px;
            border: 1px solid #4a5568;
        }

        .control-label {
            color: #a0aec0;
            font-size: 14px;
            margin-bottom: 8px;
            font-weight: 500;
        }

        .control-row {
            display: flex;
            gap: 10px;
            align-items: center;
            margin-bottom: 12px;
        }

        select, input {
            background: #2d3748;
            border: 1px solid #4a5568;
            color: #ffffff;
            padding: 8px 12px;
            border-radius: 6px;
            font-size: 14px;
        }

        button {
            background: #4299e1;
            border: none;
            color: white;
            padding: 10px 16px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 14px;
            font-weight: 500;
            transition: background 0.3s;
        }

        button:hover {
            background: #3182ce;
        }

        .btn-danger {
            background: #f56565;
        }

        .btn-danger:hover {
            background: #e53e3e;
        }

        .btn-success {
            background: #48bb78;
        }

        .btn-success:hover {
            background: #38a169;
        }

        .analytics-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
            gap: 20px;
        }

        /* Adjusted styling for chart containers */
        .chart-container {
            background: #1a202c;
            padding: 20px;
            border-radius: 12px;
            border: 1px solid #2d3748;
            height: 350px; /* New fixed height for charts */
            display: flex;
            flex-direction: column;
            justify-content: center;
        }
        
        .chart-container canvas {
            max-height: 100%;
            max-width: 100%;
        }

        .intersection-table {
            width: 100%;
            border-collapse: collapse;
            background: #1a202c;
        }

        .intersection-table th,
        .intersection-table td {
            padding: 12px 16px;
            text-align: left;
            border-bottom: 1px solid #2d3748;
        }

        .intersection-table th {
            background: #2d3748;
            font-weight: 600;
            color: #a0aec0;
            font-size: 14px;
        }

        .intersection-table td {
            color: #ffffff;
        }

        .status-dot {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            display: inline-block;
            margin-right: 8px;
        }

        .status-dot.green {
            background: #48bb78;
        }

        .status-dot.yellow {
            background: #ed8936;
        }

        .status-dot.red {
            background: #f56565;
        }

        .page {
            display: none;
        }

        .page.active {
            display: block;
        }

        .user-type-selector {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 1001;
            background: #1a202c;
            padding: 10px;
            border-radius: 8px;
            border: 1px solid #2d3748;
        }

        .alerts-panel {
            background: #1a202c;
            padding: 20px;
        }

        .alert-item {
            display: flex;
            align-items: center;
            padding: 12px;
            background: #2d3748;
            border-radius: 8px;
            margin-bottom: 12px;
            border-left: 4px solid #ed8936;
        }

        .alert-icon {
            color: #ed8936;
            margin-right: 12px;
            font-size: 18px;
        }

        .alert-content {
            flex: 1;
        }

        .alert-title {
            font-weight: 600;
            color: #ffffff;
            margin-bottom: 4px;
        }

        .alert-time {
            color: #718096;
            font-size: 12px;
        }

        .quick-actions {
            padding: 20px;
            background: #1a202c;
        }

        .action-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 15px;
        }

        .action-btn {
            background: linear-gradient(135deg, #4299e1, #63b3ed);
            border: none;
            color: white;
            padding: 16px;
            border-radius: 8px;
            cursor: pointer;
            font-weight: 600;
            transition: transform 0.3s;
            text-align: center;
        }

        .action-btn:hover {
            transform: translateY(-2px);
        }

        @media (max-width: 768px) {
            .sidebar {
                transform: translateX(-100%);
            }

            .main-content {
                margin-left: 0;
            }
        }
    </style>
</head>
<body>
    <div class="user-type-selector">
        <select id="userType" onchange="switchUserType()">
            <option value="admin">Admin (RTO Controller)</option>
            <option value="citizen">Citizen View</option>
        </select>
    </div>

    <div class="sidebar">
        <div class="logo">
            <h2>TrafficFlow Pro</h2>
        </div>
        <ul class="nav-menu">
            <li class="nav-item">
                <a class="nav-link active" onclick="showPage('overview', event)">
                    <span class="nav-icon"> 📊 </span>Overview
                </a>
            </li>
            <li class="nav-item">
                <a class="nav-link" onclick="showPage('traffic-map', event)">
                    <span class="nav-icon"> 🗺️ </span>Traffic Map
                </a>
            </li>
            <li class="nav-item">
                <a class="nav-link" onclick="showPage('analytics', event)">
                    <span class="nav-icon"> 📈 </span>Analytics
                </a>
            </li>
            <li class="nav-item admin-only">
                <a class="nav-link" onclick="showPage('signal-control', event)">
                    <span class="nav-icon"> 🚦 </span>Signal Control
                </a>
            </li>
            <li class="nav-item admin-only">
                <a class="nav-link" onclick="showPage('alerts', event)">
                    <span class="nav-icon"> ⚠️ </span>Alerts
                </a>
            </li>
            <li class="nav-item">
                <a class="nav-link" onclick="showPage('reports', event)">
                    <span class="nav-icon"> 📋 </span>Reports
                </a>
            </li>
        </ul>
    </div>

    <div class="main-content">
        <div class="header">
            <div>
                <h1>Traffic Control Center</h1>
                <p class="section-subtitle">Real-time intersection monitoring and control - Odisha</p>
            </div>
            <div class="time" id="currentTime"></div>
        </div>

        <div id="overview" class="page active">
            <div class="metrics-grid">
                <div class="metric-card">
                    <div class="metric-header">
                        <span class="metric-title">Traffic Efficiency</span>
                        <span class="metric-icon"> 📈 </span>
                    </div>
                    <div class="metric-value" id="trafficEfficiency">88.9%</div>
                    <div class="metric-change">+2.4% from last hour</div>
                </div>

                <div class="metric-card">
                    <div class="metric-header">
                        <span class="metric-title">Active Signals</span>
                        <span class="metric-icon"> 📍 </span>
                    </div>
                    <div class="metric-value"><span id="activeSignals">48</span>/50</div>
                    <div class="metric-change">2 signals in maintenance mode</div>
                </div>

                <div class="metric-card">
                    <div class="metric-header">
                        <span class="metric-title">Avg Commute Time</span>
                        <span class="metric-icon"> ⏱️ </span>
                    </div>
                    <div class="metric-value" id="avgCommute">22.8</div>
                    <div class="metric-change">-10.2% reduction achieved</div>
                </div>

                <div class="metric-card">
                    <div class="metric-header">
                        <span class="metric-title">System Status</span>
                        <span class="metric-icon"> 🔧 </span>
                    </div>
                    <div class="metric-value status-optimal">Optimal</div>
                    <div class="metric-change">All systems operational</div>
                </div>
            </div>

            <div class="content-section">
                <div class="section-card">
                    <div class="section-header">
                        <div>
                            <h3 class="section-title">Real-time Traffic Map</h3>
                            <p class="section-subtitle">Live traffic conditions across Odisha intersections</p>
                        </div>
                    </div>
                    <div class="map-container">
                        <div id="map"></div>
                    </div>
                </div>
            </div>
        </div>

        <div id="traffic-map" class="page">
            <div class="content-section">
                <div class="section-card">
                    <div class="section-header">
                        <div>
                            <h3 class="section-title">Interactive Traffic Map</h3>
                            <p class="section-subtitle">Detailed view of all intersections in Odisha</p>
                        </div>
                    </div>
                    <div class="map-container" style="height: 70vh;">
                        <div id="mapDetailed"></div>
                    </div>
                </div>
            </div>
        </div>

        <div id="analytics" class="page">
            <div class="content-section">
                <div class="analytics-grid">
                    <div class="chart-container">
                        <h4>Average Waiting Time</h4>
                        <canvas id="waitingTimeChart"></canvas>
                    </div>
                    <div class="chart-container">
                        <h4>Queue Length Analysis</h4>
                        <canvas id="queueLengthChart"></canvas>
                    </div>
                    <div class="chart-container">
                        <h4>Traffic Flow Trends</h4>
                        <canvas id="trafficFlowChart"></canvas>
                    </div>
                    <div class="chart-container">
                        <h4>Intersection Performance</h4>
                        <canvas id="performanceChart"></canvas>
                    </div>
                </div>

                <div class="section-card">
                    <div class="section-header">
                        <h3 class="section-title">Intersection Statistics</h3>
                    </div>
                    <table class="intersection-table">
                        <thead>
                            <tr>
                                <th>Intersection</th>
                                <th>Status</th>
                                <th>Avg Wait Time</th>
                                <th>Queue Length</th>
                                <th>Efficiency</th>
                                <th>Last Updated</th>
                            </tr>
                        </thead>
                        <tbody id="intersectionTableBody">
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <div id="signal-control" class="page">
            <div class="content-section">
                <div class="section-card">
                    <div class="section-header">
                        <h3 class="section-title">Manual Signal Control</h3>
                        <p class="section-subtitle">Override automatic signals for emergency or maintenance</p>
                    </div>
                    <div class="controls-panel" style="display: block;">
                        <div class="controls-grid">
                            <div class="control-group">
                                <div class="control-label">Select Intersection</div>
                                <select id="intersectionSelect" onchange="updateSignalStatus()">
                                    <option value="">Choose intersection...</option>
                                </select>
                                <div id="currentSignalStatus" style="margin-top: 10px; padding: 10px; background: #2d3748; border-radius: 6px;">
                                    <strong>Current Status:</strong> <span id="signalStatusText">Select an intersection</span>
                                </div>
                            </div>
                            <div class="control-group">
                                <div class="control-label">Manual Override</div>
                                <div class="control-row">
                                    <button class="btn-success" onclick="forceSignal('green')">Force Green</button>
                                    <button class="btn-danger" onclick="forceSignal('red')">Force Red</button>
                                    <button onclick="resetSignal()">Auto Mode</button>
                                </div>
                                <div class="control-row">
                                    <label>Duration (seconds):</label>
                                    <input type="number" id="signalDuration" value="30" min="5" max="300" style="width: 80px;">
                                </div>
                            </div>
                            <div class="control-group">
                                <div class="control-label">Emergency Controls</div>
                                <div class="control-row">
                                    <button class="btn-danger" onclick="emergencyOverride()"> 🚨 Emergency Override</button>
                                </div>
                                <div class="control-row">
                                    <button onclick="maintenanceMode()"> 🔧 Maintenance Mode</button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div id="alerts" class="page">
            <div class="content-section">
                <div class="section-card">
                    <div class="section-header">
                        <h3 class="section-title"> ⚠️ System Alerts</h3>
                    </div>
                    <div class="alerts-panel">
                        <div class="alert-item">
                            <div class="alert-icon"> 🔴 </div>
                            <div class="alert-content">
                                <div class="alert-title">Heavy traffic detected on Broadway corridor</div>
                                <div class="alert-time">2 min ago</div>
                            </div>
                        </div>
                        <div class="alert-item">
                            <div class="alert-icon"> 🔵 </div>
                            <div class="alert-content">
                                <div class="alert-title">Signal timing optimized for rush hour</div>
                                <div class="alert-time">15 min ago</div>
                            </div>
                        </div>
                        <div class="alert-item">
                            <div class="alert-icon"> 🟢 </div>
                            <div class="alert-content">
                                <div class="alert-title">New efficiency record: 94% at Center St</div>
                                <div class="alert-time">1 hour ago</div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="section-card">
                    <div class="section-header">
                        <h3 class="section-title">Quick Actions</h3>
                    </div>
                    <div class="quick-actions">
                        <div class="action-grid">
                            <button class="action-btn" onclick="emergencyOverride()"> ⚡ Emergency Override</button>
                            <button class="action-btn" onclick="optimizeAll()"> ⚙️ Optimize All Signals</button>
                            <button class="action-btn" onclick="generateReport()"> 📊 Generate Report</button>
                            <button class="action-btn" onclick="maintenanceMode()"> 🔧 Maintenance Mode</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div id="reports" class="page">
            <div class="content-section">
                <div class="section-card">
                    <div class="section-header">
                        <h3 class="section-title">Traffic Reports</h3>
                        <button onclick="generateReport()">Generate New Report</button>
                    </div>
                    <div class="analytics-grid">
                        <div class="chart-container">
                            <h4>Daily Traffic Summary</h4>
                            <canvas id="dailySummaryChart"></canvas>
                        </div>
                        <div class="chart-container">
                            <h4>Weekly Performance</h4>
                            <canvas id="weeklyChart"></canvas>
                        </div>
                    </div>
                    <div class="section-card" style="margin-top: 20px;">
                        <div class="section-header">
                            <h3 class="section-title">Report Logs</h3>
                        </div>
                        <div style="padding: 24px;">
                            <ul style="list-style-type: disc; padding-left: 20px; color: #a0aec0;">
                                <li style="margin-bottom: 8px;">Weekly Report - Sep 1, 2025: Generated successfully.</li>
                                <li style="margin-bottom: 8px;">Daily Report - Aug 30, 2025: Generated successfully.</li>
                                <li style="margin-bottom: 8px;">Congestion Report - Aug 28, 2025: Generated successfully.</li>
                                <li style="margin-bottom: 8px;">Emergency Override Log - Aug 25, 2025: Generated successfully.</li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <script>
        // Global variables
        let map, mapDetailed;
        let intersections = [];
        let trafficData = {};
        let roadSegments = {};
        let currentUserType = 'admin';
        let signalOverrides = {};
        let chartInstances = {};

        // Data for the map and analytics
        const intersectionData = [
            { id: 'bhubaneswar-main', name: 'Bhubaneswar Main', location: [20.2713, 85.8453], status: 'optimal', avgWaitTime: 120, queueLength: 5 },
            { id: 'cuttack-junction', name: 'Cuttack Junction', location: [20.4625, 85.8828], status: 'warning', avgWaitTime: 250, queueLength: 15 },
            { id: 'rourkela-square', name: 'Rourkela Square', location: [22.2583, 84.8760], status: 'congested', avgWaitTime: 400, queueLength: 30 },
            { id: 'puri-signal', name: 'Puri Signal', location: [19.8049, 85.8272], status: 'optimal', avgWaitTime: 90, queueLength: 3 },
            { id: 'konark-highway', name: 'Konark Highway', location: [19.8876, 86.0943], status: 'optimal', avgWaitTime: 110, queueLength: 6 }
        ];

        // Functions for manual signal control (re-added from previous update)
        function updateSignalStatus() {
            const select = document.getElementById('intersectionSelect');
            const selectedId = select.value;
            const statusElement = document.getElementById('signalStatusText');

            if (selectedId) {
                const intersection = intersectionData.find(i => i.id === selectedId);
                const currentStatus = signalOverrides[selectedId] || intersection.status;
                const statusMap = {
                    'optimal': 'Auto-optimized (Green)',
                    'warning': 'Congested (Yellow)',
                    'congested': 'Heavy Traffic (Red)',
                    'green': 'Manual Override (Green)',
                    'red': 'Manual Override (Red)',
                    'maintenance': 'Maintenance Mode'
                };
                statusElement.textContent = statusMap[currentStatus];
                statusElement.className = '';
                statusElement.classList.add('status-' + currentStatus);
            } else {
                statusElement.textContent = 'Select an intersection';
            }
        }

        function forceSignal(color) {
            const select = document.getElementById('intersectionSelect');
            const selectedId = select.value;
            const duration = document.getElementById('signalDuration').value;
            if (selectedId) {
                signalOverrides[selectedId] = color;
                alert(`Forcing ${color} signal at ${selectedId} for ${duration} seconds.`);
                updateSignalStatus();
            } else {
                alert('Please select an intersection first.');
            }
        }

        function resetSignal() {
            const select = document.getElementById('intersectionSelect');
            const selectedId = select.value;
            if (selectedId) {
                delete signalOverrides[selectedId];
                alert(`Signal at ${selectedId} reset to automatic mode.`);
                updateSignalStatus();
            } else {
                alert('Please select an intersection first.');
            }
        }

        function emergencyOverride() {
            alert('🚨 Activating system-wide emergency signal override. All intersections will turn red.');
        }

        function maintenanceMode() {
            alert('🔧 Activating system-wide maintenance mode. All signals will be temporarily suspended.');
        }

        function populateIntersectionSelect() {
            const select = document.getElementById('intersectionSelect');
            intersectionData.forEach(intersection => {
                const option = document.createElement('option');
                option.value = intersection.id;
                option.textContent = intersection.name;
                select.appendChild(option);
            });
        }
        
        // Functions for map initialization and updates (re-added)
        function initMaps() {
            // Main map (Overview)
            map = L.map('map').setView([20.2713, 85.8453], 12);
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
            }).addTo(map);

            // Detailed map (Traffic Map page)
            mapDetailed = L.map('mapDetailed').setView([20.2713, 85.8453], 13);
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
            }).addTo(mapDetailed);
        }

        function updateMaps() {
            // Clear existing markers
            map.eachLayer(layer => {
                if (layer instanceof L.Marker) {
                    map.removeLayer(layer);
                }
            });
            mapDetailed.eachLayer(layer => {
                if (layer instanceof L.Marker) {
                    mapDetailed.removeLayer(layer);
                }
            });

            // Add new markers based on current data
            intersectionData.forEach(intersection => {
                const status = signalOverrides[intersection.id] || intersection.status;
                let color;
                if (status === 'optimal' || status === 'green') color = 'green';
                else if (status === 'warning' || status === 'yellow') color = 'orange';
                else color = 'red';

                const markerHtml = `<div style="background-color: ${color}; width: 2rem; height: 2rem; display: block; left: -1rem; top: -1rem; position: relative; border-radius: 2rem 2rem 0; transform: rotate(45deg); border: 1px solid #FFFFFF"></div>`;
                const customIcon = new L.DivIcon({
                    className: 'my-custom-pin',
                    iconAnchor: [0, 24],
                    popupAnchor: [0, -36],
                    html: markerHtml
                });

                const popupContent = `
                    <b>${intersection.name}</b><br>
                    Status: <span class="status-dot ${color}"></span> ${status}<br>
                    Avg Wait Time: ${intersection.avgWaitTime}s<br>
                    Queue Length: ${intersection.queueLength} cars
                `;

                L.marker(intersection.location, { icon: customIcon }).addTo(map).bindPopup(popupContent);
                L.marker(intersection.location, { icon: customIcon }).addTo(mapDetailed).bindPopup(popupContent);
            });
        }
        
        // Functions for chart initialization and updates (re-added)
        function createCharts() {
            const commonChartOptions = {
                responsive: true,
                maintainAspectRatio: false,
                scales: {
                    x: {
                        ticks: { color: '#a0aec0' },
                        grid: { color: '#2d3748' }
                    },
                    y: {
                        ticks: { color: '#a0aec0' },
                        grid: { color: '#2d3748' }
                    }
                },
                plugins: {
                    legend: {
                        labels: { color: '#a0aec0' }
                    }
                }
            };
            
            const waitingTimeCtx = document.getElementById('waitingTimeChart').getContext('2d');
            chartInstances.waitingTime = new Chart(waitingTimeCtx, {
                type: 'line',
                data: {
                    labels: ['9:00', '9:15', '9:30', '9:45', '10:00'],
                    datasets: [{
                        label: 'Avg Waiting Time (s)',
                        data: [180, 210, 200, 190, 185],
                        borderColor: '#4299e1',
                        tension: 0.1
                    }]
                },
                options: commonChartOptions
            });

            const queueLengthCtx = document.getElementById('queueLengthChart').getContext('2d');
            chartInstances.queueLength = new Chart(queueLengthCtx, {
                type: 'bar',
                data: {
                    labels: ['East', 'West', 'North', 'South'],
                    datasets: [{
                        label: 'Queue Length',
                        data: [25, 30, 20, 18],
                        backgroundColor: '#63b3ed'
                    }]
                },
                options: commonChartOptions
            });

            const trafficFlowCtx = document.getElementById('trafficFlowChart').getContext('2d');
            chartInstances.trafficFlow = new Chart(trafficFlowCtx, {
                type: 'line',
                data: {
                    labels: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'],
                    datasets: [{
                        label: 'Daily Vehicle Count (thousands)',
                        data: [120, 150, 130, 145, 160, 110, 95],
                        borderColor: '#48bb78',
                        tension: 0.1
                    }]
                },
                options: commonChartOptions
            });

            const performanceCtx = document.getElementById('performanceChart').getContext('2d');
            chartInstances.performance = new Chart(performanceCtx, {
                type: 'doughnut',
                data: {
                    labels: ['Optimal', 'Warning', 'Congested'],
                    datasets: [{
                        data: [75, 15, 10],
                        backgroundColor: ['#48bb78', '#ed8936', '#f56565']
                    }]
                },
                options: commonChartOptions
            });
            
            const dailySummaryCtx = document.getElementById('dailySummaryChart').getContext('2d');
            chartInstances.dailySummary = new Chart(dailySummaryCtx, {
                type: 'bar',
                data: {
                    labels: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri'],
                    datasets: [{
                        label: 'Vehicle Count',
                        data: [1500, 1750, 1600, 1900, 2100],
                        backgroundColor: '#4299e1'
                    }]
                },
                options: commonChartOptions
            });

            const weeklyChartCtx = document.getElementById('weeklyChart').getContext('2d');
            chartInstances.weeklyChart = new Chart(weeklyChartCtx, {
                type: 'line',
                data: {
                    labels: ['Week 1', 'Week 2', 'Week 3', 'Week 4'],
                    datasets: [{
                        label: 'Efficiency Score (%)',
                        data: [85, 88, 86, 91],
                        borderColor: '#48bb78',
                        tension: 0.1
                    }]
                },
                options: commonChartOptions
            });
        }

        function updateCharts() {
            // This function would be updated with real-time data
            // For now, it's a placeholder.
        }

        function updateTable() {
            const tableBody = document.getElementById('intersectionTableBody');
            tableBody.innerHTML = '';
            intersectionData.forEach(intersection => {
                const statusClass = (signalOverrides[intersection.id] || intersection.status) === 'congested' || (signalOverrides[intersection.id] === 'red') ? 'red' : ((signalOverrides[intersection.id] || intersection.status) === 'warning' ? 'yellow' : 'green');
                const row = `
                    <tr>
                        <td>${intersection.name}</td>
                        <td><span class="status-dot ${statusClass}"></span> ${signalOverrides[intersection.id] || intersection.status}</td>
                        <td>${intersection.avgWaitTime}s</td>
                        <td>${intersection.queueLength} cars</td>
                        <td>${(Math.random() * 10 + 85).toFixed(1)}%</td>
                        <td>${new Date().toLocaleTimeString()}</td>
                    </tr>
                `;
                tableBody.innerHTML += row;
            });
        }

        function showPage(pageId, event) {
            document.querySelectorAll('.page').forEach(page => {
                page.classList.remove('active');
            });
            document.getElementById(pageId).classList.add('active');
            
            // Remove active class from all links and add to the clicked one
            document.querySelectorAll('.nav-link').forEach(link => {
                link.classList.remove('active');
            });
            event.target.closest('.nav-link').classList.add('active');

            // Resize maps and charts when shown
            if (pageId === 'traffic-map') {
                setTimeout(() => {
                    if (mapDetailed) {
                        mapDetailed.invalidateSize();
                    }
                }, 100);
            } else if (pageId === 'overview') {
                 setTimeout(() => {
                    if (map) {
                        map.invalidateSize();
                    }
                }, 100);
            } else if (pageId === 'analytics' || pageId === 'reports') {
                 setTimeout(() => {
                    for (const chartId in chartInstances) {
                        if (chartInstances[chartId]) {
                            chartInstances[chartId].resize();
                        }
                    }
                }, 100);
            }
        }

        function switchUserType() {
            const userType = document.getElementById('userType').value;
            currentUserType = userType;
            if (userType === 'citizen') {
                document.querySelectorAll('.admin-only').forEach(el => el.style.display = 'none');
            } else {
                document.querySelectorAll('.admin-only').forEach(el => el.style.display = 'list-item');
            }
            showPage('overview', {target: document.querySelector('.nav-link.active')});
        }
        
        function updateTime() {
            document.getElementById('currentTime').textContent = new Date().toLocaleString();
        }

        function generateReport() {
            alert('Generating a detailed traffic report...');
        }
        
        // Initial setup
        function init() {
            updateTime();
            setInterval(updateTime, 1000);
            initMaps();
            createCharts();
            updateMaps();
            updateTable();
            populateIntersectionSelect();
            switchUserType();
        }

        window.onload = init;
    </script>
</body>
</html>
