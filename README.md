<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="theme-color" content="#1a237e">
    <title>FireGuard Map - Proximity Command System</title>
    
    <!-- Leaflet CSS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <!-- FontAwesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: #f0f2f5;
            overflow: hidden;
        }

        /* Hauptcontainer */
        #app {
            width: 100vw;
            height: 100vh;
            display: flex;
            position: relative;
        }

        /* Sidebar - dynamisch basierend auf Proximity */
        #sidebar {
            width: 380px;
            background: white;
            box-shadow: 2px 0 10px rgba(0,0,0,0.1);
            display: flex;
            flex-direction: column;
            transition: width 0.3s ease;
            z-index: 1000;
        }

        #sidebar.collapsed {
            width: 60px;
        }

        .sidebar-header {
            background: linear-gradient(135deg, #1a237e 0%, #283593 100%);
            color: white;
            padding: 20px;
            position: relative;
        }

        .sidebar-header.collapsed-content {
            display: none;
        }

        #sidebar.collapsed .sidebar-header {
            padding: 15px 10px;
            text-align: center;
        }

        #sidebar.collapsed .sidebar-header h1 {
            display: none;
        }

        .toggle-btn {
            position: absolute;
            right: -20px;
            top: 20px;
            background: #ff5722;
            color: white;
            border: none;
            width: 40px;
            height: 40px;
            border-radius: 50%;
            cursor: pointer;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
            z-index: 1001;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .role-selector {
            margin: 15px 0;
            padding: 0 20px;
        }

        #sidebar.collapsed .role-selector {
            display: none;
        }

        select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 6px;
            font-size: 14px;
            background: white;
        }

        /* Spracheingabe Bereich */
        .voice-control {
            padding: 15px 20px;
            background: #e8eaf6;
            border-bottom: 1px solid #c5cae9;
        }

        #sidebar.collapsed .voice-control {
            display: none;
        }

        .voice-btn {
            width: 100%;
            padding: 12px;
            background: #ff5722;
            color: white;
            border: none;
            border-radius: 25px;
            cursor: pointer;
            font-size: 14px;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
            transition: all 0.3s;
        }

        .voice-btn:hover {
            background: #e64a19;
            transform: translateY(-1px);
        }

        .voice-btn.recording {
            background: #c62828;
            animation: pulse 1.5s infinite;
        }

        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.7; }
        }

        .voice-status {
            text-align: center;
            font-size: 12px;
            color: #666;
            margin-top: 8px;
            min-height: 16px;
        }

        /* Proximity Status */
        .proximity-status {
            background: #fff3e0;
            padding: 12px 20px;
            border-left: 4px solid #ff9800;
            font-size: 13px;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        #sidebar.collapsed .proximity-status {
            display: none;
        }

        .proximity-indicator {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            background: #ccc;
            transition: background 0.3s;
        }

        .proximity-indicator.active {
            background: #4caf50;
            box-shadow: 0 0 8px #4caf50;
        }

        .proximity-indicator.near {
            background: #ff9800;
            box-shadow: 0 0 8px #ff9800;
        }

        /* Content Bereich */
        .sidebar-content {
            flex: 1;
            overflow-y: auto;
            padding: 20px;
        }

        #sidebar.collapsed .sidebar-content {
            display: none;
        }

        /* Kategorie Filter */
        .category-filters {
            margin-bottom: 20px;
        }

        .filter-title {
            font-size: 12px;
            color: #666;
            text-transform: uppercase;
            margin-bottom: 10px;
            letter-spacing: 0.5px;
        }

        .category-btn {
            display: inline-flex;
            align-items: center;
            gap: 5px;
            padding: 6px 12px;
            margin: 2px;
            border: none;
            border-radius: 15px;
            font-size: 12px;
            cursor: pointer;
            transition: all 0.2s;
        }

        .category-btn.active {
            box-shadow: 0 2px 4px rgba(0,0,0,0.2);
            transform: scale(1.05);
        }

        .cat-critical { background: #ffcdd2; color: #c62828; }
        .cat-warning { background: #ffe0b2; color: #ef6c00; }
        .cat-ok { background: #c8e6c9; color: #2e7d32; }
        .cat-info { background: #bbdefb; color: #1565c0; }
        .cat-offline { background: #e0e0e0; color: #616161; }

        /* Dynamische Liste basierend auf Proximity */
        .component-list {
            margin-top: 20px;
        }

        .list-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
        }

        .list-title {
            font-size: 14px;
            font-weight: 600;
            color: #333;
        }

        .distance-badge {
            background: #e3f2fd;
            color: #1976d2;
            padding: 2px 8px;
            border-radius: 12px;
            font-size: 11px;
        }

        .component-item {
            background: white;
            border: 1px solid #e0e0e0;
            border-radius: 8px;
            padding: 12px;
            margin-bottom: 8px;
            cursor: pointer;
            transition: all 0.2s;
            border-left: 4px solid transparent;
        }

        .component-item:hover {
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
            transform: translateX(2px);
        }

        .component-item.critical { border-left-color: #f44336; }
        .component-item.warning { border-left-color: #ff9800; }
        .component-item.ok { border-left-color: #4caf50; }
        .component-item.info { border-left-color: #2196f3; }

        .component-item.hidden-by-distance {
            opacity: 0.3;
            filter: grayscale(100%);
        }

        .comp-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 5px;
        }

        .comp-name {
            font-weight: 600;
            font-size: 13px;
            color: #333;
        }

        .comp-status {
            width: 8px;
            height: 8px;
            border-radius: 50%;
        }

        .comp-details {
            font-size: 12px;
            color: #666;
            line-height: 1.4;
            margin-top: 5px;
        }

        .comp-distance {
            font-size: 11px;
            color: #999;
            margin-top: 5px;
            display: flex;
            align-items: center;
            gap: 5px;
        }

        /* Karte */
        #map {
            flex: 1;
            height: 100vh;
            z-index: 1;
        }

        /* Overlay Controls */
        .map-controls {
            position: absolute;
            top: 20px;
            right: 20px;
            z-index: 500;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .control-btn {
            width: 40px;
            height: 40px;
            background: white;
            border: none;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            color: #333;
            font-size: 16px;
            transition: all 0.2s;
        }

        .control-btn:hover {
            background: #f5f5f5;
            transform: translateY(-2px);
        }

        .control-btn.active {
            background: #1a237e;
            color: white;
        }

        /* Legende */
        .legend {
            position: absolute;
            bottom: 30px;
            right: 20px;
            background: white;
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            z-index: 500;
            max-width: 200px;
        }

        .legend-title {
            font-size: 12px;
            font-weight: 600;
            margin-bottom: 10px;
            color: #333;
        }

        .legend-item {
            display: flex;
            align-items: center;
            gap: 8px;
            margin-bottom: 6px;
            font-size: 12px;
            color: #666;
        }

        .legend-icon {
            width: 12px;
            height: 12px;
            border-radius: 50%;
        }

        /* Responsive */
        @media (max-width: 768px) {
            #sidebar {
                position: absolute;
                height: 100vh;
                z-index: 1000;
                box-shadow: 2px 0 10px rgba(0,0,0,0.3);
            }
            
            #sidebar.collapsed {
                transform: translateX(-60px);
            }
            
            .legend {
                bottom: 10px;
                right: 10px;
                font-size: 11px;
                padding: 10px;
            }
        }

        /* Loading Spinner */
        .loading {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 2000;
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
            display: none;
        }

        .spinner {
            border: 3px solid #f3f3f3;
            border-top: 3px solid #ff5722;
            border-radius: 50%;
            width: 30px;
            height: 30px;
            animation: spin 1s linear infinite;
            margin: 0 auto 10px;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body>

<div id="app">
    <!-- Sidebar -->
    <div id="sidebar">
        <button class="toggle-btn" onclick="toggleSidebar()">
            <i class="fas fa-chevron-left" id="toggle-icon"></i>
        </button>

        <div class="sidebar-header">
            <h1 style="font-size: 20px; margin-bottom: 5px;">FireGuard Map</h1>
            <p style="font-size: 12px; opacity: 0.9;">Proximity Command System</p>
        </div>

        <!-- Rollen Selektor -->
        <div class="role-selector">
            <label style="font-size: 12px; color: #666; display: block; margin-bottom: 5px;">Ihre Rolle:</label>
            <select id="roleSelect" onchange="changeRole()">
                <option value="all">Alle anzeigen (Admin)</option>
                <option value="maintenance">Wartungstechniker</option>
                <option value="medical">Sanitäter</option>
                <option value="security">Sicherheitspersonal</option>
                <option value="event">Event-Manager</option>
                <option value="technician">Technik/Crew</option>
            </select>
        </div>

        <!-- Spracheingabe -->
        <div class="voice-control">
            <button class="voice-btn" id="voiceBtn" onclick="toggleVoice()">
                <i class="fas fa-microphone"></i>
                <span>Sprachbefehl</span>
            </button>
            <div class="voice-status" id="voiceStatus">Tippen zum Sprechen...</div>
        </div>

        <!-- Proximity Status -->
        <div class="proximity-status">
            <div class="proximity-indicator" id="proxIndicator"></div>
            <div>
                <div style="font-weight: 600;">Standort: <span id="locationName">Suche...</span></div>
                <div style="font-size: 11px; opacity: 0.8;" id="proximityText">Warte auf GPS...</div>
            </div>
        </div>

        <!-- Content -->
        <div class="sidebar-content">
            <!-- Kategorie Filter -->
            <div class="category-filters">
                <div class="filter-title">Status-Filter</div>
                <button class="category-btn cat-critical active" onclick="toggleCategory('critical')">
                    <i class="fas fa-exclamation-circle"></i> Akut
                </button>
                <button class="category-btn cat-warning active" onclick="toggleCategory('warning')">
                    <i class="fas fa-exclamation-triangle"></i> Wartung
                </button>
                <button class="category-btn cat-ok active" onclick="toggleCategory('ok')">
                    <i class="fas fa-check-circle"></i> OK
                </button>
                <button class="category-btn cat-info active" onclick="toggleCategory('info')">
                    <i class="fas fa-info-circle"></i> Info
                </button>
            </div>

            <!-- Dynamische Komponentenliste -->
            <div class="component-list" id="componentList">
                <div class="list-header">
                    <span class="list-title">Nahegelegene Objekte</span>
                    <span class="distance-badge" id="objectCount">0</span>
                </div>
                <div id="listContainer">
                    <!-- Wird dynamisch gefüllt -->
                </div>
            </div>
        </div>
    </div>

    <!-- Karte -->
    <div id="map"></div>

    <!-- Map Controls -->
    <div class="map-controls">
        <button class="control-btn active" id="gpsBtn" onclick="toggleGPS()" title="GPS Tracking">
            <i class="fas fa-location-arrow"></i>
        </button>
        <button class="control-btn" onclick="centerMap()" title="Zentrieren">
            <i class="fas fa-crosshairs"></i>
        </button>
        <button class="control-btn" onclick="toggleOffline()" title="Offline-Modus">
            <i class="fas fa-wifi"></i>
        </button>
    </div>

    <!-- Legende -->
    <div class="legend">
        <div class="legend-title">Status-Legende</div>
        <div class="legend-item">
            <div class="legend-icon" style="background: #f44336;"></div>
            <span>Akut/Kritisch</span>
        </div>
        <div class="legend-item">
            <div class="legend-icon" style="background: #ff9800;"></div>
            <span>Wartung fällig</span>
        </div>
        <div class="legend-item">
            <div class="legend-icon" style="background: #4caf50;"></div>
            <span>In Ordnung</span>
        </div>
        <div class="legend-item">
            <div class="legend-icon" style="background: #2196f3;"></div>
            <span>Information</span>
        </div>
        <div class="legend-item">
            <div class="legend-icon" style="background: #9e9e9e;"></div>
            <span>Offline/Unbekannt</span>
        </div>
        <div style="margin-top: 10px; padding-top: 10px; border-top: 1px solid #eee; font-size: 11px; color: #999;">
            <i class="fas fa-circle" style="color: #ff5722; font-size: 8px;"></i> Ihre Position
        </div>
    </div>

    <!-- Loading -->
    <div class="loading" id="loading">
        <div class="spinner"></div>
        <div style="font-size: 12px; color: #666;">Initialisiere...</div>
    </div>
</div>

<!-- Leaflet JS -->
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script>
    // Globale Variablen
    let map;
    let userMarker;
    let markers = [];
    let userPosition = null;
    let watchId = null;
    let currentRole = 'all';
    let activeCategories = ['critical', 'warning', 'ok', 'info'];
    let isRecording = false;
    let recognition = null;

    // Beispiel-Daten: Festival Gelände (Basierend auf echten Event-Layouts)
    const components = [
        // Medizinisch (Sanitäter)
        { id: 1, name: "Haupt-Sanitätszelt", type: "medical", status: "ok", lat: 52.5200, lng: 13.4050, 
          details: "Ausgestattet mit AED, Notfallkoffer, 2 Sanitätern", roleAccess: ['medical', 'all', 'security'] },
        { id: 2, name: "AED Station Bühne", type: "medical", status: "critical", lat: 52.5195, lng: 13.4045, 
          details: "Defibrillator, Wartung überfällig!", roleAccess: ['medical', 'all'] },
        { id: 3, name: "Erste-Hilfe-Posten Ost", type: "medical", status: "ok", lat: 52.5205, lng: 13.4060, 
          details: "Geöffnet bis 24:00", roleAccess: ['medical', 'all', 'security', 'event'] },

        // Security
        { id: 4, name: "Security Checkpoint A", type: "security", status: "ok", lat: 52.5198, lng: 13.4048, 
          details: "4 Personen, Funkkanal 3", roleAccess: ['security', 'all', 'event'] },
        { id: 5, name: "Security Checkpoint B", type: "security", status: "warning", lat: 52.5202, lng: 13.4055, 
          details: "Unterbesetzt (nur 2 Personen)", roleAccess: ['security', 'all'] },
        { id: 6, name: "Überwachungskamera 12", type: "security", status: "ok", lat: 52.5196, lng: 13.4052, 
          details: "Bereich: Haupteingang, 360°", roleAccess: ['security', 'all'] },

        // Technik/Strom
        { id: 7, name: "Hauptstromverteiler", type: "technical", status: "ok", lat: 52.5201, lng: 13.4049, 
          details: "400V, 250A, Last: 78%", roleAccess: ['technician', 'all', 'event'] },
        { id: 8, name: "Generator Bühne", type: "technical", status: "warning", lat: 52.5194, lng: 13.4046, 
          details: "Tankstand: 45%, Nachfüllung in 2h", roleAccess: ['technician', 'all'] },
        { id: 9, name: "Stromzelt Catering", type: "technical", status: "ok", lat: 52.5203, lng: 13.4058, 
          details: "230V Verteilung, 3 Phasen", roleAccess: ['technician', 'all', 'event'] },

        // Infrastruktur
        { id: 10, name: "Toilettenblock A", type: "infrastructure", status: "warning", lat: 52.5204, lng: 13.4053, 
          details: "Reinigung erforderlich, Papier nachfüllen", roleAccess: ['maintenance', 'all', 'event'] },
        { id: 11, name: "Wasserstation", type: "infrastructure", status: "ok", lat: 52.5199, lng: 13.4054, 
          details: "Trinkwasser, Druck: 3.2 bar", roleAccess: ['all', 'event', 'technician'] },
        { id: 12, name: "Abfallcontainer S1", type: "infrastructure", status: "critical", lat: 52.5197, lng: 13.4056, 
          details: "Überfüllt! Sofort Leerung!", roleAccess: ['maintenance', 'all', 'event'] },

        // Info/Service
        { id: 13, name: "Infopoint", type: "info", status: "ok", lat: 52.5206, lng: 13.4051, 
          details: "Öffnungszeiten: 10-22 Uhr", roleAccess: ['all'] },
        { id: 14, name: "Lost & Found", type: "info", status: "ok", lat: 52.5202, lng: 13.4044, 
          details: "Fundsachenannahme", roleAccess: ['all', 'security', 'event'] }
    ];

    // Status-Konfiguration
    const statusConfig = {
        critical: { color: '#f44336', icon: 'fa-exclamation-circle', label: 'Akut' },
        warning: { color: '#ff9800', icon: 'fa-exclamation-triangle', label: 'Wartung' },
        ok: { color: '#4caf50', icon: 'fa-check-circle', label: 'OK' },
        info: { color: '#2196f3', icon: 'fa-info-circle', label: 'Info' },
        offline: { color: '#9e9e9e', icon: 'fa-question-circle', label: 'Offline' }
    };

    // Initialisierung
    document.addEventListener('DOMContentLoaded', function() {
        initMap();
        initSpeechRecognition();
        startGPSTracking();
        renderComponentList();
        
        // Simulation: Position in der Mitte des Geländes
        setTimeout(() => {
            updateUserPosition(52.5200, 13.4050);
        }, 1000);
    });

    function initMap() {
        // Karte auf Berlin Mitte zentriert (Beispielkoordinaten)
        map = L.map('map').setView([52.5200, 13.4050], 17);

        // OpenStreetMap Layer
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '© OpenStreetMap contributors',
            maxZoom: 20
        }).addTo(map);

        // Custom Icons erstellen
        components.forEach(comp => {
            createMarker(comp);
        });

        // Benutzer-Position Marker
        const userIcon = L.divIcon({
            className: 'user-marker',
            html: '<div style="background: #ff5722; width: 20px; height: 20px; border-radius: 50%; border: 3px solid white; box-shadow: 0 0 10px rgba(255,87,34,0.5); animation: pulse 2s infinite;"></div>',
            iconSize: [20, 20],
            iconAnchor: [10, 10]
        });

        userMarker = L.marker([52.5200, 13.4050], { icon: userIcon, zIndexOffset: 1000 }).addTo(map);
    }

    function createMarker(comp) {
        const config = statusConfig[comp.status];
        
        const icon = L.divIcon({
            className: 'custom-marker',
            html: `<div style="background: ${config.color}; width: 30px; height: 30px; border-radius: 50%; border: 3px solid white; box-shadow: 0 2px 5px rgba(0,0,0,0.3); display: flex; align-items: center; justify-content: center; color: white; font-size: 12px;">
                    <i class="fas ${config.icon}"></i>
                   </div>`,
            iconSize: [30, 30],
            iconAnchor: [15, 15]
        });

        const marker = L.marker([comp.lat, comp.lng], { icon: icon }).addTo(map);
        
        // Popup mit Details
        const popupContent = `
            <div style="min-width: 200px;">
                <h3 style="margin: 0 0 10px 0; color: ${config.color}; font-size: 16px;">${comp.name}</h3>
                <p style="margin: 0 0 10px 0; font-size: 13px; color: #666;">${comp.details}</p>
                <div style="font-size: 11px; color: #999; margin-bottom: 10px;">
                    <i class="fas fa-ruler"></i> <span class="distance-display" data-id="${comp.id}">Berechne...</span>
                </div>
                <div style="display: flex; gap: 5px;">
                    <button onclick="focusComponent(${comp.id})" style="flex: 1; padding: 5px; background: #1a237e; color: white; border: none; border-radius: 4px; cursor: pointer; font-size: 12px;">
                        <i class="fas fa-crosshairs"></i> Fokus
                    </button>
                    <button onclick="reportIssue(${comp.id})" style="flex: 1; padding: 5px; background: #f44336; color: white; border: none; border-radius: 4px; cursor: pointer; font-size: 12px;">
                        <i class="fas fa-bell"></i> Melden
                    </button>
                </div>
            </div>
        `;
        
        marker.bindPopup(popupContent);
        marker.componentId = comp.id;
        markers.push({ marker: marker, data: comp });
    }

    // Spracherkennung
    function initSpeechRecognition() {
        if ('webkitSpeechRecognition' in window || 'SpeechRecognition' in window) {
            const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            recognition = new SpeechRecognition();
            recognition.lang = 'de-DE';
            recognition.continuous = false;
            recognition.interimResults = false;

            recognition.onstart = function() {
                isRecording = true;
                document.getElementById('voiceBtn').classList.add('recording');
                document.getElementById('voiceStatus').textContent = 'Höre zu...';
            };

            recognition.onend = function() {
                isRecording = false;
                document.getElementById('voiceBtn').classList.remove('recording');
                document.getElementById('voiceStatus').textContent = 'Tippen zum Sprechen...';
            };

            recognition.onresult = function(event) {
                const transcript = event.results[0][0].transcript.toLowerCase();
                document.getElementById('voiceStatus').textContent = `Erkannt: "${transcript}"`;
                processVoiceCommand(transcript);
            };

            recognition.onerror = function(event) {
                console.error('Spracherkennungsfehler:', event.error);
                document.getElementById('voiceStatus').textContent = 'Fehler. Bitte wiederholen.';
            };
        } else {
            document.getElementById('voiceBtn').style.display = 'none';
            document.getElementById('voiceStatus').textContent = 'Spracheingabe nicht unterstützt';
        }
    }

    function toggleVoice() {
        if (!recognition) return;
        
        if (isRecording) {
            recognition.stop();
        } else {
            recognition.start();
        }
    }

    function processVoiceCommand(command) {
        // Befehle verarbeiten
        if (command.includes('suche') || command.includes('wo ist') || command.includes('finde')) {
            // Extrahiere Namen
            const searchTerm = command.replace(/(suche|wo ist|finde)/g, '').trim();
            searchComponent(searchTerm);
        } else if (command.includes('status')) {
            const status = command.includes('kritisch') ? 'critical' : 
                          command.includes('warnung') ? 'warning' : 
                          command.includes('ok') ? 'ok' : null;
            if (status) filterByStatus(status);
        } else if (command.includes('alle anzeigen')) {
            resetFilters();
        } else if (command.includes('zoom')) {
            if (command.includes('rein')) map.zoomIn();
            if (command.includes('raus')) map.zoomOut();
        } else if (command.includes('zentrieren') || command.includes('position')) {
            centerMap();
        }
    }

    // GPS & Proximity
    function startGPSTracking() {
        if ("geolocation" in navigator) {
            watchId = navigator.geolocation.watchPosition(
                (position) => {
                    updateUserPosition(position.coords.latitude, position.coords.longitude);
                },
                (error) => {
                    console.error("GPS Fehler:", error);
                    // Fallback für Demo
                    simulateProximity();
                },
                {
                    enableHighAccuracy: true,
                    timeout: 5000,
                    maximumAge: 0
                }
            );
        } else {
            simulateProximity();
        }
    }

    function updateUserPosition(lat, lng) {
        userPosition = { lat, lng };
        if (userMarker) {
            userMarker.setLatLng([lat, lng]);
        }
        
        checkProximity();
        updateDistances();
    }

    function simulateProximity() {
        // Simuliert Bewegung für Demo-Zwecke
        let offset = 0;
        setInterval(() => {
            offset += 0.0001;
            const newLat = 52.5200 + Math.sin(offset) * 0.001;
            const newLng = 13.4050 + Math.cos(offset) * 0.001;
            updateUserPosition(newLat, newLng);
        }, 3000);
    }

    function checkProximity() {
        if (!userPosition) return;

        let nearestDist = Infinity;
        let nearestComp = null;

        components.forEach(comp => {
            const dist = calculateDistance(userPosition.lat, userPosition.lng, comp.lat, comp.lng);
            if (dist < nearestDist) {
                nearestDist = dist;
                nearestComp = comp;
            }

            // Proximity Logic: Daten freischalten bei < 50m
            if (dist < 0.05) { // 50 Meter
                unlockComponentData(comp);
            }
        });

        // UI Update
        updateProximityUI(nearestComp, nearestDist);
    }

    function calculateDistance(lat1, lng1, lat2, lng2) {
        const R = 6371; // Erdradius in km
        const dLat = (lat2 - lat1) * Math.PI / 180;
        const dLng = (lng2 - lng1) * Math.PI / 180;
        const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
                  Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
                  Math.sin(dLng/2) * Math.sin(dLng/2);
        const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
        return R * c;
    }

    function updateProximityUI(nearestComp, distance) {
        const indicator = document.getElementById('proxIndicator');
        const locName = document.getElementById('locationName');
        const proxText = document.getElementById('proximityText');

        if (nearestComp) {
            locName.textContent = nearestComp.name;
            const distM = Math.round(distance * 1000);
            proxText.textContent = `${distM}m entfernt`;

            if (distM < 20) {
                indicator.className = 'proximity-indicator active';
                proxText.textContent += ' - Details freigeschaltet';
            } else if (distM < 50) {
                indicator.className = 'proximity-indicator near';
                proxText.textContent += ' - Annäherung erkannt';
            } else {
                indicator.className = 'proximity-indicator';
            }
        }
    }

    function unlockComponentData(comp) {
        // Visuelles Feedback in der Liste
        const item = document.querySelector(`[data-comp-id="${comp.id}"]`);
        if (item) {
            item.classList.remove('hidden-by-distance');
            item.style.border = '2px solid #ff5722';
            setTimeout(() => {
                item.style.border = '';
            }, 2000);
        }
    }

    // Listen-Rendering
    function renderComponentList() {
        const container = document.getElementById('listContainer');
        container.innerHTML = '';

        const filtered = components.filter(comp => {
            // Rolle prüfen
            if (currentRole !== 'all' && !comp.roleAccess.includes(currentRole)) return false;
            // Kategorie prüfen
            if (!activeCategories.includes(comp.status)) return false;
            return true;
        });

        // Sortieren nach Distanz (wenn verfügbar)
        if (userPosition) {
            filtered.sort((a, b) => {
                const distA = calculateDistance(userPosition.lat, userPosition.lng, a.lat, a.lng);
                const distB = calculateDistance(userPosition.lat, userPosition.lng, b.lat, b.lng);
                return distA - distB;
            });
        }

        document.getElementById('objectCount').textContent = filtered.length;

        filtered.forEach(comp => {
            const config = statusConfig[comp.status];
            let distance = '';
            let distanceClass = '';
            
            if (userPosition) {
                const distKm = calculateDistance(userPosition.lat, userPosition.lng, comp.lat, comp.lng);
                const distM = Math.round(distKm * 1000);
                distance = `${distM}m`;
                
                // Visuelle Distanz-Hervorhebung
                if (distM > 100) distanceClass = 'hidden-by-distance';
            }

            const div = document.createElement('div');
            div.className = `component-item ${comp.status} ${distanceClass}`;
            div.setAttribute('data-comp-id', comp.id);
            div.onclick = () => focusComponent(comp.id);
            
            div.innerHTML = `
                <div class="comp-header">
                    <span class="comp-name">${comp.name}</span>
                    <div class="comp-status" style="background: ${config.color};"></div>
                </div>
                <div class="comp-details">${comp.details}</div>
                <div class="comp-distance">
                    <i class="fas fa-map-marker-alt"></i>
                    <span>${distance}</span>
                    <span style="margin-left: auto; font-size: 10px; color: ${config.color};">
                        <i class="fas ${config.icon}"></i> ${config.label}
                    </span>
                </div>
            `;
            
            container.appendChild(div);
        });
    }

    function updateDistances() {
        renderComponentList();
        
        // Auch Popup-Inhalte aktualisieren
        markers.forEach(({data}) => {
            const distDisplay = document.querySelector(`.distance-display[data-id="${data.id}"]`);
            if (distDisplay && userPosition) {
                const dist = calculateDistance(userPosition.lat, userPosition.lng, data.lat, data.lng);
                const distM = Math.round(dist * 1000);
                distDisplay.textContent = `${distM}m entfernt`;
            }
        });
    }

    // Interaktion
    function toggleSidebar() {
        const sidebar = document.getElementById('sidebar');
        const icon = document.getElementById('toggle-icon');
        sidebar.classList.toggle('collapsed');
        icon.className = sidebar.classList.contains('collapsed') ? 'fas fa-chevron-right' : 'fas fa-chevron-left';
        
        // Karte neu ausrichten
        setTimeout(() => map.invalidateSize(), 300);
    }

    function changeRole() {
        currentRole = document.getElementById('roleSelect').value;
        renderComponentList();
        
        // Marker-Filterung
        markers.forEach(({marker, data}) => {
            if (currentRole === 'all' || data.roleAccess.includes(currentRole)) {
                marker.addTo(map);
            } else {
                map.removeLayer(marker);
            }
        });
    }

    function toggleCategory(category) {
        const index = activeCategories.indexOf(category);
        if (index > -1) {
            if (activeCategories.length > 1) activeCategories.splice(index, 1);
        } else {
            activeCategories.push(category);
        }
        
        // Button-Status aktualisieren
        event.target.classList.toggle('active', activeCategories.includes(category));
        renderComponentList();
    }

    function focusComponent(id) {
        const comp = components.find(c => c.id === id);
        if (comp) {
            map.setView([comp.lat, comp.lng], 19);
            const markerObj = markers.find(m => m.data.id === id);
            if (markerObj) markerObj.marker.openPopup();
        }
    }

    function reportIssue(id) {
        alert(`Störung für Komponente ${id} gemeldet. Nächster Techniker wird informiert.`);
    }

    function centerMap() {
        if (userPosition) {
            map.setView([userPosition.lat, userPosition.lng], 18);
        }
    }

    function toggleGPS() {
        const btn = document.getElementById('gpsBtn');
        if (watchId) {
            navigator.geolocation.clearWatch(watchId);
            watchId = null;
            btn.classList.remove('active');
        } else {
            startGPSTracking();
            btn.classList.add('active');
        }
    }

    function toggleOffline() {
        alert('Offline-Modus: Daten werden lokal zwischengespeichert.');
    }

    function searchComponent(term) {
        const found = components.find(c => c.name.toLowerCase().includes(term.toLowerCase()));
        if (found) {
            focusComponent(found.id);
            document.getElementById('voiceStatus').textContent = `Gefunden: ${found.name}`;
        } else {
            document.getElementById('voiceStatus').textContent = 'Nicht gefunden';
        }
    }

    function filterByStatus(status) {
        activeCategories = [status];
        document.querySelectorAll('.category-btn').forEach(btn => {
            btn.classList.remove('active');
            if (btn.classList.contains(`cat-${status}`)) btn.classList.add('active');
        });
        renderComponentList();
    }

    function resetFilters() {
        activeCategories = ['critical', 'warning', 'ok', 'info'];
        document.querySelectorAll('.category-btn').forEach(btn => btn.classList.add('active'));
        renderComponentList();
    }

    // Tastatur-Shortcuts
    document.addEventListener('keydown', (e) => {
        if (e.key === 'v' && e.ctrlKey) {
            e.preventDefault();
            toggleVoice();
        }
        if (e.key === 'Escape') {
            document.getElementById('sidebar').classList.remove('collapsed');
        }
    });
</script>

</body>
</html>
# Career_map