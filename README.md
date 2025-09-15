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
        }
        
        .header {
            background: #1a1f2e;
            padding: 20px 30px;
            border-bottom: 1px solid #2d3748;
            display: flex;
            justify-content: between;
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
        
        #map {
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
        
        .chart-container {
            background: #1a202c;
            padding: 20px;
            border-radius: 12px;
            border: 1px solid #2d3748;
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
    <!-- User Type Selector -->
    <div class="user-type-selector">
        <select id="userType" onchange="switchUserType()">
            <option value="admin">Admin (RTO Controller)</option>
            <option value="citizen">Citizen View</option>
        </select>
    </div>

    <!-- Sidebar -->
    <div class="sidebar">
        <div class="logo">
            <h2>TrafficFlow Pro</h2>
        </div>
        <ul class="nav-menu">
            <li class="nav-item">
                <a class="nav-link active" onclick="showPage('overview')">
                    <span class="nav-icon">📊</span>Overview
                </a>
            </li>
            <li class="nav-item">
                <a class="nav-link" onclick="showPage('traffic-map')">
                    <span class="nav-icon">🗺️</span>Traffic Map
                </a>
            </li>
            <li class="nav-item">
                <a class="nav-link" onclick="showPage('analytics')">
                    <span class="nav-icon">📈</span>Analytics
                </a>
            </li>
            <li class="nav-item admin-only">
                <a class="nav-link" onclick="showPage('signal-control')">
                    <span class="nav-icon">🚦</span>Signal Control
                </a>
            </li>
            <li class="nav-item admin-only">
                <a class="nav-link" onclick="showPage('alerts')">
                    <span class="nav-icon">⚠️</span>Alerts
                </a>
            </li>
            <li class="nav-item">
                <a class="nav-link" onclick="showPage('reports')">
                    <span class="nav-icon">📋</span>Reports
                </a>
            </li>
        </ul>
    </div>

    <!-- Main Content -->
    <div class="main-content">
        <!-- Header -->
        <div class="header">
            <div>
                <h1>Traffic Control Center</h1>
                <p class="section-subtitle">Real-time intersection monitoring and control - Odisha</p>
            </div>
            <div class="time" id="currentTime"></div>
        </div>

        <!-- Overview Page -->
        <div id="overview" class="page active">
            <!-- Metrics Grid -->
            <div class="metrics-grid">
                <div class="metric-card">
                    <div class="metric-header">
                        <span class="metric-title">Traffic Efficiency</span>
                        <span class="metric-icon">📈</span>
                    </div>
                    <div class="metric-value" id="trafficEfficiency">88.9%</div>
                    <div class="metric-change">+2.4% from last hour</div>
                </div>
                
                <div class="metric-card">
                    <div class="metric-header">
                        <span class="metric-title">Active Signals</span>
                        <span class="metric-icon">📍</span>
                    </div>
                    <div class="metric-value"><span id="activeSignals">48</span>/50</div>
                    <div class="metric-change">2 signals in maintenance mode</div>
                </div>
                
                <div class="metric-card">
                    <div class="metric-header">
                        <span class="metric-title">Avg Commute Time</span>
                        <span class="metric-icon">⏱️</span>
                    </div>
                    <div class="metric-value" id="avgCommute">22.8</div>
                    <div class="metric-change">-10.2% reduction achieved</div>
                </div>
                
                <div class="metric-card">
                    <div class="metric-header">
                        <span class="metric-title">System Status</span>
                        <span class="metric-icon">🔧</span>
                    </div>
                    <div class="metric-value status-optimal">Optimal</div>
                    <div class="metric-change">All systems operational</div>
                </div>
            </div>

            <!-- Main Dashboard Content -->
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

        <!-- Traffic Map Page -->
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

        <!-- Analytics Page -->
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

        <!-- Signal Control Page (Admin Only) -->
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
                                    <button class="btn-danger" onclick="emergencyOverride()">🚨 Emergency Override</button>
                                </div>
                                <div class="control-row">
                                    <button onclick="maintenanceMode()">🔧 Maintenance Mode</button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Alerts Page (Admin Only) -->
        <div id="alerts" class="page">
            <div class="content-section">
                <div class="section-card">
                    <div class="section-header">
                        <h3 class="section-title">⚠️ System Alerts</h3>
                    </div>
                    <div class="alerts-panel">
                        <div class="alert-item">
                            <div class="alert-icon">🔴</div>
                            <div class="alert-content">
                                <div class="alert-title">Heavy traffic detected on Broadway corridor</div>
                                <div class="alert-time">2 min ago</div>
                            </div>
                        </div>
                        <div class="alert-item">
                            <div class="alert-icon">🔵</div>
                            <div class="alert-content">
                                <div class="alert-title">Signal timing optimized for rush hour</div>
                                <div class="alert-time">15 min ago</div>
                            </div>
                        </div>
                        <div class="alert-item">
                            <div class="alert-icon">🟢</div>
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
                            <button class="action-btn" onclick="emergencyOverride()">⚡ Emergency Override</button>
                            <button class="action-btn" onclick="optimizeAll()">⚙️ Optimize All Signals</button>
                            <button class="action-btn" onclick="generateReport()">📊 Generate Report</button>
                            <button class="action-btn" onclick="maintenanceMode()">🔧 Maintenance Mode</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Reports Page -->
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
                </div>
            </div>
        </div>
    </div>

    <script>
        // Global variables
        let map, mapDetailed;
        let intersections = [];
        let currentUserType = 'admin';
        let signalOverrides = {};
        
        // Initialize the application
        function init() {
            updateTime();
            setInterval(updateTime, 1000);
            initMaps();
            generateIntersectionData();
            initCharts();
            updateIntersectionTable();
            populateIntersectionSelect();
            setInterval(updateRealTimeData, 5000);
        }
        
        // Update current time
        function updateTime() {
            const now = new Date();
            document.getElementById('currentTime').textContent = now.toLocaleTimeString();
        }
        
        // Initialize Leaflet maps with Odisha focus
        function initMaps() {
            // Main overview map
            map = L.map('map').setView([20.2961, 85.8245], 10);
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                maxZoom: 19,
                attribution: '© OpenStreetMap contributors'
            }).addTo(map);
            
            // Detailed traffic map
            mapDetailed = L.map('mapDetailed').setView([20.2961, 85.8245], 11);
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                maxZoom: 19,
                attribution: '© OpenStreetMap contributors'
            }).addTo(mapDetailed);
            
            // Load roads and intersections
            loadOdishaRoads();
        }
        
        // Load Odisha roads using Overpass API
        async function loadOdishaRoads() {
            try {
                const query = `
                [out:json][timeout:25];
                area["name"="Odisha"]->.a;
                (
                  way["highway"~"primary|secondary|trunk|residential"](area.a);
                );
                out geom;
                `;
                
                const url = "https://overpass-api.de/api/interpreter?data=" + encodeURIComponent(query);
                const response = await fetch(url);
                const data = await response.json();
                
                data.elements.forEach((element, index) => {
                    if (element.type === 'way' && element.geometry) {
                        const coords = element.geometry.map(point => [point.lat, point.lon]);
                        const trafficStatus = ['optimal', 'warning', 'congested'][Math.floor(Math.random() * 3)];
                        const color = getTrafficColor(trafficStatus);
                        
                        // Add to both maps
                        L.polyline(coords, {
                            color: color,
                            weight: 4,
                            opacity: 0.7
                        }).addTo(map);
                        
                        L.polyline(coords, {
                            color: color,
                            weight: 5,
                            opacity: 0.8
                        }).addTo(mapDetailed);
                        
                        // Create intersection markers at road junctions
                        if (index % 10 === 0 && coords.length > 0) {
                            createIntersectionMarker(coords[Math.floor(coords.length / 2)], index);
                        }
                    }
                });
                
            } catch (error) {
                console.error('Error loading roads:', error);
                // Fallback: create demo intersections
                createDemoIntersections();
            }
        }
        
        // Create intersection marker
        function createIntersectionMarker(coords, index) {
            const intersectionId = `ODI-${String(index).padStart(3, '0')}`;
            const status = ['optimal', 'warning', 'congested'][Math.floor(Math.random() * 3)];
            const color = getTrafficColor(status);
            
            const intersection = {
                id: intersectionId,
                coords: coords,
                status: status,
                waitTime: Math.floor(Math.random() * 120) + 10,
                queueLength: Math.floor(Math.random() * 20) + 2,
                efficiency: Math.floor(Math.random() * 30) + 70,
                lastUpdated: new Date()
            };
            
            intersections.push(intersection);
            
            // Create markers for both maps
            const marker1 = L.circle(coords, {
                radius: 100,
                color: color,
                fill: true,
                fillOpacity: 0.8
            }).addTo(map);
            
            const marker2 = L.circle(coords, {
                radius: 150,
                color: color,
                fill: true,
                fillOpacity: 0.8
            }).addTo(mapDetailed);
            
            const popupContent = `
                <strong>${intersectionId}</strong><br>
                Status: <span style="color: ${color}">${status.toUpperCase()}</span><br>
                Wait Time: ${intersection.waitTime}s<br>
                Queue: ${intersection.queueLength} vehicles
            `;
            
            marker1.bindPopup(popupContent);
            marker2.bindPopup(popupContent);
        }
        
        // Create demo intersections if API fails
        function createDemoIntersections() {
            const demoPoints = [
                [20.2961, 85.8245], // Bhubaneswar center
                [20.3029, 85.8207], // Near Kalinga Institute
                [20.2906, 85.8313], // Near railway station
                [20.3102, 85.8312], // Patia area
                [20.2844, 85.8394], // Old town area
            ];
            
            demoPoints.forEach((coords, index) => {
                createIntersectionMarker(coords, index + 100);
            });
        }
        
        // Get traffic color based on status
        function getTrafficColor(status) {
            const colors = {
                'optimal': '#48bb78',
                'warning': '#ed8936',
                'congested': '#f56565'
            };
            return colors[status] || '#4299e1';
        }
        
        // Generate intersection data
        function generateIntersectionData() {
            // This function is called after roads are loaded
            // Data is already generated in createIntersectionMarker
        }
        
        // Initialize charts
        function initCharts() {
            const chartOptions = {
                responsive: true,
                plugins: {
                    legend: {
                        labels: {
                            color: '#a0aec0'
                        }
                    }
                },
                scales: {
                    y: {
                        ticks: {
                            color: '#a0aec0'
                        },
                        grid: {
                            color: '#2d3748'
                        }
                    },
                    x: {
                        ticks: {
                            color: '#a0aec0'
                        },
                        grid: {
                            color: '#2d3748'
                        }
                    }
                }
            };
            
            // Waiting Time Chart
            const waitingTimeCtx = document.getElementById('waitingTimeChart').getContext('2d');
            new Chart(waitingTimeCtx, {
                type: 'line',
                data: {
                    labels: ['6 AM', '9 AM', '12 PM', '3 PM', '6 PM', '9 PM'],
                    datasets: [{
                        label: 'Average Wait Time (seconds)',
                        data: [45, 78, 52, 64, 89, 43],
                        borderColor: '#4299e1',
                        backgroundColor: 'rgba(66, 153, 225, 0.1)',
                        tension: 0.4
                    }]
                },
                options: chartOptions
            });
            
            // Queue Length Chart
            const queueLengthCtx = document.getElementById('queueLengthChart').getContext('2d');
            new Chart(queueLengthCtx, {
                type: 'bar',
                data: {
                    labels: ['ODI-001', 'ODI-002', 'ODI-003', 'ODI-004', 'ODI-005'],
                    datasets: [{
                        label: 'Queue Length (vehicles)',
                        data: [12, 8, 15, 6, 10],
                        backgroundColor: ['#48bb78', '#48bb78', '#ed8936', '#48bb78', '#48bb78']
                    }]
                },
                options: chartOptions
            });
            
            // Traffic Flow Chart
            const trafficFlowCtx = document.getElementById('trafficFlowChart').getContext('2d');
            new Chart(trafficFlowCtx, {
                type: 'doughnut',
                data: {
                    labels: ['Optimal', 'Warning', 'Congested'],
                    datasets: [{
                        data: [65, 25, 10],
                        backgroundColor: ['#48bb78', '#ed8936', '#f56565']
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            labels: {
                                color: '#a0aec0'
                            }
                        }
                    }
                }
            });
            
            // Performance Chart
            const performanceCtx = document.getElementById('performanceChart').getContext('2d');
            new Chart(performanceCtx, {
                type: 'radar',
                data: {
                    labels: ['Response Time', 'Queue Management', 'Flow Optimization', 'Signal Timing', 'Emergency Handling'],
                    datasets: [{
                        label: 'Current Performance',
                        data: [85, 92, 78, 88, 95],
                        borderColor: '#4299e1',
                        backgroundColor: 'rgba(66, 153, 225, 0.2)'
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            labels: {
                                color: '#a0aec0'
                            }
                        }
                    },
                    scales: {
                        r: {
                            ticks: {
                                color: '#a0aec0'
                            },
                            grid: {
                                color: '#2d3748'
                            }
                        }
                    }
                }
            });
            
            // Daily Summary Chart
            const dailySummaryCtx = document.getElementById('dailySummaryChart').getContext('2d');
            new Chart(dailySummaryCtx, {
                type: 'line',
                data: {
                    labels: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'],
                    datasets: [{
                        label: 'Efficiency %',
                        data: [88, 92, 85, 90, 87, 75, 68],
                        borderColor: '#48bb78',
                        backgroundColor: 'rgba(72, 187, 120, 0.1)',
                        tension: 0.4
                    }]
                },
                options: chartOptions
            });
            
            // Weekly Chart
            const weeklyCtx = document.getElementById('weeklyChart').getContext('2d');
            new Chart(weeklyCtx, {
                type: 'bar',
                data: {
                    labels: ['Week 1', 'Week 2', 'Week 3', 'Week 4'],
                    datasets: [{
                        label: 'Average Commute Time (min)',
                        data: [25.4, 23.8, 22.1, 22.8],
                        backgroundColor: '#4299e1'
                    }]
                },
                options: chartOptions
            });
        }
        
        // Update intersection table
        function updateIntersectionTable() {
            const tbody = document.getElementById('intersectionTableBody');
            if (!tbody) return;
            
            tbody.innerHTML = intersections.slice(0, 10).map(intersection => `
                <tr>
                    <td>${intersection.id}</td>
                    <td>
                        <span class="status-dot ${intersection.status === 'optimal' ? 'green' : intersection.status === 'warning' ? 'yellow' : 'red'}"></span>
                        ${intersection.status.toUpperCase()}
                    </td>
                    <td>${intersection.waitTime}s</td>
                    <td>${intersection.queueLength} vehicles</td>
                    <td>${intersection.efficiency}%</td>
                    <td>${intersection.lastUpdated.toLocaleTimeString()}</td>
                </tr>
            `).join('');
        }
        
        // Populate intersection select dropdown
        function populateIntersectionSelect() {
            const select = document.getElementById('intersectionSelect');
            if (!select) return;
            
            select.innerHTML = '<option value="">Choose intersection...</option>';
            intersections.forEach(intersection => {
                const option = document.createElement('option');
                option.value = intersection.id;
                option.textContent = `${intersection.id} - ${intersection.status.toUpperCase()}`;
                select.appendChild(option);
            });
        }
        
        // Update signal status display
        function updateSignalStatus() {
            const select = document.getElementById('intersectionSelect');
            const statusText = document.getElementById('signalStatusText');
            
            if (!select || !statusText) return;
            
            const selectedId = select.value;
            if (!selectedId) {
                statusText.textContent = 'Select an intersection';
                return;
            }
            
            const intersection = intersections.find(i => i.id === selectedId);
            if (intersection) {
                const overrideStatus = signalOverrides[selectedId];
                if (overrideStatus) {
                    statusText.innerHTML = `<span style="color: ${overrideStatus.color}">MANUAL OVERRIDE - ${overrideStatus.signal.toUpperCase()}</span> (${overrideStatus.remaining}s remaining)`;
                } else {
                    statusText.innerHTML = `Auto Mode - <span style="color: ${getTrafficColor(intersection.status)}">${intersection.status.toUpperCase()}</span>`;
                }
            }
        }
        
        // Force signal to specific state
        function forceSignal(signal) {
            const select = document.getElementById('intersectionSelect');
            const durationInput = document.getElementById('signalDuration');
            
            if (!select || !durationInput) return;
            
            const selectedId = select.value;
            const duration = parseInt(durationInput.value) || 30;
            
            if (!selectedId) {
                alert('Please select an intersection first');
                return;
            }
            
            const color = signal === 'green' ? '#48bb78' : '#f56565';
            
            signalOverrides[selectedId] = {
                signal: signal,
                duration: duration,
                remaining: duration,
                color: color,
                startTime: Date.now()
            };
            
            updateSignalStatus();
            
            // Set timer to remove override
            setTimeout(() => {
                delete signalOverrides[selectedId];
                updateSignalStatus();
            }, duration * 1000);
            
            // Update countdown
            const countdown = setInterval(() => {
                if (signalOverrides[selectedId]) {
                    signalOverrides[selectedId].remaining--;
                    updateSignalStatus();
                    
                    if (signalOverrides[selectedId].remaining <= 0) {
                        clearInterval(countdown);
                    }
                } else {
                    clearInterval(countdown);
                }
            }, 1000);
            
            console.log(`Manual override: ${selectedId} forced to ${signal} for ${duration} seconds`);
        }
        
        // Reset signal to auto mode
        function resetSignal() {
            const select = document.getElementById('intersectionSelect');
            const selectedId = select.value;
            
            if (!selectedId) {
                alert('Please select an intersection first');
                return;
            }
            
            delete signalOverrides[selectedId];
            updateSignalStatus();
            console.log(`${selectedId} returned to auto mode`);
        }
        
        // Emergency override
        function emergencyOverride() {
            const confirmed = confirm('Emergency override will force ALL intersections to allow emergency vehicle passage. Continue?');
            if (confirmed) {
                intersections.forEach(intersection => {
                    signalOverrides[intersection.id] = {
                        signal: 'emergency',
                        duration: 120,
                        remaining: 120,
                        color: '#f56565',
                        startTime: Date.now()
                    };
                });
                alert('Emergency override activated for all intersections (120 seconds)');
                
                setTimeout(() => {
                    intersections.forEach(intersection => {
                        delete signalOverrides[intersection.id];
                    });
                    updateSignalStatus();
                }, 120000);
            }
        }
        
        // Maintenance mode
        function maintenanceMode() {
            const select = document.getElementById('intersectionSelect');
            const selectedId = select.value;
            
            if (!selectedId) {
                alert('Please select an intersection first');
                return;
            }
            
            const confirmed = confirm(`Put ${selectedId} in maintenance mode? This will disable automatic control.`);
            if (confirmed) {
                signalOverrides[selectedId] = {
                    signal: 'maintenance',
                    duration: 3600, // 1 hour
                    remaining: 3600,
                    color: '#ed8936',
                    startTime: Date.now()
                };
                updateSignalStatus();
                console.log(`${selectedId} entered maintenance mode`);
            }
        }
        
        // Optimize all signals
        function optimizeAll() {
            alert('AI optimization initiated for all intersections. This may take a few minutes.');
            console.log('Optimizing all signals using AI algorithms');
        }
        
        // Generate report
        function generateReport() {
            alert('Generating comprehensive traffic report...');
            console.log('Report generation started');
        }
        
        // Update real-time data
        function updateRealTimeData() {
            // Update metrics
            const efficiency = Math.floor(Math.random() * 10) + 85;
            document.getElementById('trafficEfficiency').textContent = efficiency + '%';
            
            const avgTime = (Math.random() * 5 + 20).toFixed(1);
            document.getElementById('avgCommute').textContent = avgTime;
            
            // Update intersection data
            intersections.forEach(intersection => {
                intersection.waitTime = Math.floor(Math.random() * 120) + 10;
                intersection.queueLength = Math.floor(Math.random() * 20) + 2;
                intersection.efficiency = Math.floor(Math.random() * 30) + 70;
                intersection.lastUpdated = new Date();
            });
            
            updateIntersectionTable();
            updateSignalStatus();
        }
        
        // Show specific page
        function showPage(pageId) {
            // Hide all pages
            document.querySelectorAll('.page').forEach(page => {
                page.classList.remove('active');
            });
            
            // Show selected page
            document.getElementById(pageId).classList.add('active');
            
            // Update nav active state
            document.querySelectorAll('.nav-link').forEach(link => {
                link.classList.remove('active');
            });
            
            event.target.classList.add('active');
            
            // Resize maps when shown
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
            }
        }
        
        // Switch user type
        function switchUserType() {
            const userType = document.getElementById('userType').value;
            currentUserType = userType;
            
            const adminOnlyElements = document.querySelectorAll('.admin-only');
            
            if (userType === 'citizen') {
                adminOnlyElements.forEach(el => el.style.display = 'none');
                // Show overview page for citizens
                showPage('overview');
            } else {
                adminOnlyElements.forEach(el => el.style.display = 'block');
            }
        }
        
        // Initialize application when page loads
        document.addEventListener('DOMContentLoaded', function() {
            init();
        });
    </script>
</body>
</html>
