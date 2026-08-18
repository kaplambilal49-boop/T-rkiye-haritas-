<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Türkiye 81 İl Haritası</title>

    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css" />

    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        html, body { height: 100%; width: 100%; overflow: hidden; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }

        #map { height: 100%; width: 100%; background: #e5e3df; }

        /* Sol panel */
        .sidebar {
            position: absolute;
            top: 0; left: 0;
            width: 280px;
            height: 100%;
            background: rgba(255,255,255,0.97);
            z-index: 1000;
            box-shadow: 2px 0 12px rgba(0,0,0,0.15);
            display: flex;
            flex-direction: column;
            transition: transform 0.3s ease;
        }
        .sidebar.collapsed { transform: translateX(-100%); }

        .sidebar-header {
            padding: 16px;
            background: #1a73e8;
            color: white;
        }
        .sidebar-header h2 { font-size: 18px; margin-bottom: 4px; }
        .sidebar-header p { font-size: 12px; opacity: 0.9; }

        .search-box {
            padding: 12px;
            border-bottom: 1px solid #eee;
        }
        .search-box input {
            width: 100%;
            padding: 10px 12px;
            border: 1px solid #ddd;
            border-radius: 8px;
            font-size: 14px;
            outline: none;
        }
        .search-box input:focus { border-color: #1a73e8; }

        .city-list {
            flex: 1;
            overflow-y: auto;
            padding: 8px 0;
        }
        .city-item {
            padding: 10px 16px;
            cursor: pointer;
            font-size: 14px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            transition: background 0.15s;
        }
        .city-item:hover { background: #f0f7ff; }
        .city-item.active { background: #e3f2fd; font-weight: 600; color: #1a73e8; }
        .city-item .plate {
            background: #eee;
            padding: 2px 8px;
            border-radius: 12px;
            font-size: 12px;
            color: #555;
        }

        /* Butonlar */
        .controls {
            position: absolute;
            top: 12px;
            right: 12px;
            z-index: 1000;
            display: flex;
            flex-direction: column;
            gap: 8px;
        }
        .ctrl-btn {
            width: 42px;
            height: 42px;
            background: white;
            border: none;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.2);
            cursor: pointer;
            font-size: 16px;
            color: #333;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: background 0.15s;
        }
        .ctrl-btn:hover { background: #f5f5f5; }

        .toggle-sidebar {
            position: absolute;
            top: 12px;
            left: 12px;
            z-index: 1001;
            width: 42px;
            height: 42px;
            background: white;
            border: none;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.2);
            cursor: pointer;
            font-size: 16px;
        }

        /* Leaflet stil düzeltmeleri */
        .leaflet-popup-content { margin: 10px 14px; font-size: 14px; }
        .leaflet-popup-content strong { color: #1a73e8; }

        @media (max-width: 600px) {
            .sidebar { width: 100%; }
        }
    </style>
</head>
<body>

    <div id="map"></div>

    <!-- Sol panel aç/kapa -->
    <button class="toggle-sidebar" id="toggleSidebar" title="İl Listesi">
        <i class="fas fa-bars"></i>
    </button>

    <!-- Sol panel -->
    <div class="sidebar" id="sidebar">
        <div class="sidebar-header">
            <h2>Türkiye 81 İl</h2>
            <p>İl seçerek haritada odaklan</p>
        </div>
        <div class="search-box">
            <input type="text" id="searchInput" placeholder="İl ara... (ör: İstanbul)">
        </div>
        <div class="city-list" id="cityList"></div>
    </div>

    <!-- Sağ kontroller -->
    <div class="controls">
        <button class="ctrl-btn" id="btnFullscreen" title="Tam Ekran"><i class="fas fa-expand"></i></button>
        <button class="ctrl-btn" id="btnLocate" title="Konumumu Bul"><i class="fas fa-location-crosshairs"></i></button>
        <button class="ctrl-btn" id="btnReset" title="Türkiye'ye Dön"><i class="fas fa-home"></i></button>
    </div>

    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script>
        // ====================== HARİTA ======================
        const map = L.map('map', {
            center: [39.0, 35.2],
            zoom: 6,
            minZoom: 5,
            maxZoom: 18,
            zoomControl: true
        });

        // Katmanlar
        const osm = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '© OpenStreetMap',
            maxZoom: 19
        });

        const carto = L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
            attribution: '© CARTO',
            maxZoom: 19
        });

        const dark = L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
            attribution: '© CARTO',
            maxZoom: 19
        });

        osm.addTo(map);

        L.control.layers({
            "OpenStreetMap": osm,
            "Açık Renkli": carto,
            "Karanlık": dark
        }, null, { position: 'bottomright' }).addTo(map);

        L.control.scale({ imperial: false, position: 'bottomleft' }).addTo(map);

        // ====================== GEOJSON + 81 İL ======================
        let geoLayer = null;
        let cityData = [];
        const highlightStyle = { weight: 3, color: '#ff9800', fillOpacity: 0.35 };
        const normalStyle = { weight: 1.2, color: '#1565c0', fillColor: '#42a5f5', fillOpacity: 0.15 };

        // GeoJSON yükle (81 il)
        fetch('https://raw.githubusercontent.com/alpers/Turkey-Maps-GeoJSON/master/tr-cities.json')
            .then(r => r.json())
            .then(data => {
                cityData = data.features.map(f => ({
                    name: f.properties.name,
                    number: f.properties.number,
                    feature: f
                })).sort((a, b) => a.name.localeCompare(b.name, 'tr'));

                // İl listesini doldur
                const listEl = document.getElementById('cityList');
                cityData.forEach(city => {
                    const div = document.createElement('div');
                    div.className = 'city-item';
                    div.innerHTML = `<span>\( {city.name}</span><span class="plate"> \){String(city.number).padStart(2,'0')}</span>`;
                    div.onclick = () => focusCity(city.name);
                    listEl.appendChild(div);
                });

                // Haritaya ekle
                geoLayer = L.geoJSON(data, {
                    style: normalStyle,
                    onEachFeature: (feature, layer) => {
                        const name = feature.properties.name;
                        const plate = feature.properties.number;

                        layer.bindPopup(`<strong>${name}</strong><br>Plaka: ${String(plate).padStart(2,'0')}`);

                        layer.on({
                            mouseover: (e) => {
                                e.target.setStyle(highlightStyle);
                                e.target.bringToFront();
                            },
                            mouseout: (e) => {
                                geoLayer.resetStyle(e.target);
                            },
                            click: (e) => {
                                map.fitBounds(e.target.getBounds(), { padding: [40, 40] });
                            }
                        });
                    }
                }).addTo(map);
            })
            .catch(err => {
                console.error('GeoJSON yüklenemedi:', err);
                alert('İl sınırları yüklenirken hata oluştu. İnternet bağlantını kontrol et.');
            });

        // ====================== FONKSİYONLAR ======================
        function focusCity(name) {
            if (!geoLayer) return;

            let found = null;
            geoLayer.eachLayer(layer => {
                if (layer.feature.properties.name === name) {
                    found = layer;
                }
            });

            if (found) {
                map.fitBounds(found.getBounds(), { padding: [50, 50], maxZoom: 9 });
                found.setStyle(highlightStyle);
                setTimeout(() => geoLayer.resetStyle(found), 2500);
                found.openPopup();

                // Listeyi güncelle
                document.querySelectorAll('.city-item').forEach(el => {
                    el.classList.toggle('active', el.querySelector('span').textContent === name);
                });
            }
        }

        // Arama
        document.getElementById('searchInput').addEventListener('input', (e) => {
            const q = e.target.value.trim().toLocaleLowerCase('tr');
            document.querySelectorAll('.city-item').forEach(el => {
                const name = el.querySelector('span').textContent.toLocaleLowerCase('tr');
                el.style.display = name.includes(q) ? 'flex' : 'none';
            });
        });

        // Sidebar aç/kapa
        document.getElementById('toggleSidebar').onclick = () => {
            document.getElementById('sidebar').classList.toggle('collapsed');
        };

        // Tam ekran
        document.getElementById('btnFullscreen').onclick = () => {
            if (!document.fullscreenElement) {
                document.documentElement.requestFullscreen();
            } else {
                document.exitFullscreen();
            }
        };

        // Konumumu bul
        document.getElementById('btnLocate').onclick = () => {
            map.locate({ setView: true, maxZoom: 12 });
        };

        map.on('locationfound', (e) => {
            L.marker(e.latlng).addTo(map)
                .bindPopup('Buradasınız').openPopup();
        });

        // Türkiye'ye dön
        document.getElementById('btnReset').onclick = () => {
            map.setView([39.0, 35.2], 6);
        };

        // Mobilde başlangıçta sidebar kapalı
        if (window.innerWidth < 700) {
            document.getElementById('sidebar').classList.add('collapsed');
        }
    </script>
</body>
</html>
