<!DOCTYPE html>
<html lang="pt-PT">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestão de Embarques & Contentores</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Leaflet CSS & JS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- FontAwesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        brand: {
                            50: '#f0f7ff',
                            100: '#e0effe',
                            500: '#0284c7',
                            600: '#0369a1',
                            900: '#0c4a6e'
                        }
                    }
                }
            }
        }
    </script>
    <style>
        .leaflet-container {
            background-color: #0f172a !important;
        }
        .custom-scroll::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        .custom-scroll::-webkit-scrollbar-track {
            background: #1e293b;
        }
        .custom-scroll::-webkit-scrollbar-thumb {
            background: #475569;
            border-radius: 3px;
        }
    </style>
</head>
<body class="bg-slate-900 text-slate-100 font-sans antialiased min-h-screen flex flex-col custom-scroll">

    <!-- Top Navigation Header -->
    <header class="bg-slate-800/90 backdrop-blur border-b border-slate-700 sticky top-0 z-50">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex items-center justify-between h-16">
                <!-- Logo & Brand -->
                <div class="flex items-center gap-3">
                    <div class="w-10 h-10 rounded-xl bg-gradient-to-tr from-sky-500 to-indigo-600 flex items-center justify-center text-white font-bold text-xl shadow-lg shadow-sky-500/30">
                        <i class="fa-solid font-bold fa-ship"></i>
                    </div>
                    <div>
                        <h1 class="font-bold text-lg text-white leading-none">LogiControl Pro</h1>
                        <span class="text-xs text-sky-400 font-medium">Gestão Integrada de Contentores & Documentos</span>
                    </div>
                </div>

                <!-- Navigation Tabs -->
                <nav class="flex space-x-2 bg-slate-900/60 p-1 rounded-xl border border-slate-700/60">
                    <button onclick="switchTab('shipments')" id="tab-btn-shipments" class="px-4 py-2 text-sm font-semibold rounded-lg transition-all flex items-center gap-2 bg-sky-600 text-white shadow">
                        <i class="fa-solid fa-boxes-packing"></i> Embarques & Contentores
                    </button>
                    <button onclick="switchTab('documents')" id="tab-btn-documents" class="px-4 py-2 text-sm font-semibold rounded-lg transition-all flex items-center gap-2 text-slate-400 hover:text-white">
                        <i class="fa-solid fa-file-invoice"></i> Controlo Documental
                    </button>
                    <button onclick="switchTab('analytics')" id="tab-btn-analytics" class="px-4 py-2 text-sm font-semibold rounded-lg transition-all flex items-center gap-2 text-slate-400 hover:text-white">
                        <i class="fa-solid fa-chart-pie"></i> Análise de Atrasos
                    </button>
                </nav>

                <!-- Header Actions -->
                <div class="flex items-center gap-3">
                    <button onclick="openNewShipmentModal()" class="bg-gradient-to-r from-sky-500 to-indigo-600 hover:from-sky-400 hover:to-indigo-500 text-white px-4 py-2 rounded-lg font-semibold text-sm shadow-md transition-all flex items-center gap-2">
                        <i class="fa-solid fa-plus"></i> Novo Embarque
                    </button>
                </div>
            </div>
        </div>
    </header>

    <!-- Main Content Container -->
    <main class="flex-1 max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6">

        <!-- GLOBAL SUMMARY KPI BAR -->
        <section class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-5 gap-4 mb-6">
            <div class="bg-slate-800/60 border border-slate-700/60 rounded-xl p-4 flex items-center gap-4">
                <div class="w-12 h-12 rounded-lg bg-sky-500/10 border border-sky-500/20 text-sky-400 flex items-center justify-center text-xl">
                    <i class="fa-solid fa-box-archive"></i>
                </div>
                <div>
                    <span class="text-xs text-slate-400 uppercase font-bold tracking-wider">Total Embarques</span>
                    <h3 class="text-2xl font-black text-white" id="kpi-total-shipments">0</h3>
                </div>
            </div>

            <div class="bg-slate-800/60 border border-slate-700/60 rounded-xl p-4 flex items-center gap-4">
                <div class="w-12 h-12 rounded-lg bg-indigo-500/10 border border-indigo-500/20 text-indigo-400 flex items-center justify-center text-xl">
                    <i class="fa-solid fa-cubes"></i>
                </div>
                <div>
                    <span class="text-xs text-slate-400 uppercase font-bold tracking-wider">Total Contentores</span>
                    <h3 class="text-2xl font-black text-white" id="kpi-total-containers">0</h3>
                </div>
            </div>

            <div class="bg-slate-800/60 border border-slate-700/60 rounded-xl p-4 flex items-center gap-4">
                <div class="w-12 h-12 rounded-lg bg-emerald-500/10 border border-emerald-500/20 text-emerald-400 flex items-center justify-center text-xl">
                    <i class="fa-solid fa-circle-check"></i>
                </div>
                <div>
                    <span class="text-xs text-slate-400 uppercase font-bold tracking-wider">Em Dia (On-Time)</span>
                    <h3 class="text-2xl font-black text-emerald-400" id="kpi-ontime">0</h3>
                </div>
            </div>

            <div class="bg-slate-800/60 border border-slate-700/60 rounded-xl p-4 flex items-center gap-4">
                <div class="w-12 h-12 rounded-lg bg-rose-500/10 border border-rose-500/20 text-rose-400 flex items-center justify-center text-xl">
                    <i class="fa-solid fa-clock-rotate-left"></i>
                </div>
                <div>
                    <span class="text-xs text-slate-400 uppercase font-bold tracking-wider">Com Atraso</span>
                    <h3 class="text-2xl font-black text-rose-400" id="kpi-delayed">0</h3>
                </div>
            </div>

            <div class="bg-slate-800/60 border border-slate-700/60 rounded-xl p-4 flex items-center gap-4">
                <div class="w-12 h-12 rounded-lg bg-amber-500/10 border border-amber-500/20 text-amber-400 flex items-center justify-center text-xl">
                    <i class="fa-solid fa-file-circle-exclamation"></i>
                </div>
                <div>
                    <span class="text-xs text-slate-400 uppercase font-bold tracking-wider">Docs Pendentes</span>
                    <h3 class="text-2xl font-black text-amber-400" id="kpi-pending-docs">0</h3>
                </div>
            </div>
        </section>

        <!-- VIEW 1: SHIPMENTS & CONTAINERS (MAIN VIEW) -->
        <div id="view-shipments" class="space-y-6">
            
            <!-- Global Map & Filter Toolbar -->
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                
                <!-- Esri Ocean Marine Map Container -->
                <div class="lg:col-span-2 bg-slate-800/80 border border-slate-700 rounded-2xl p-4 shadow-xl flex flex-col">
                    <div class="flex items-center justify-between mb-3">
                        <div class="flex items-center gap-2">
                            <i class="fa-solid fa-earth-americas text-sky-400"></i>
                            <h2 class="font-bold text-slate-200">Localização Estimada (Rotas & Posições)</h2>
                        </div>
                        <span class="text-xs bg-slate-700 text-slate-300 px-2 py-1 rounded-full font-mono">Esri Ocean Basemap</span>
                    </div>
                    <div id="map" class="w-full h-[320px] rounded-xl overflow-hidden border border-slate-700 shadow-inner"></div>
                </div>

                <!-- Control Panel & Quick Filters -->
                <div class="bg-slate-800/80 border border-slate-700 rounded-2xl p-4 shadow-xl flex flex-col justify-between">
                    <div>
                        <h2 class="font-bold text-slate-200 mb-3 flex items-center gap-2">
                            <i class="fa-solid fa-filter text-indigo-400"></i> Filtros de Embarque
                        </h2>
                        
                        <div class="space-y-3">
                            <div>
                                <label class="text-xs text-slate-400 font-medium">Pesquisa por Ref, Booking ou Contentor</label>
                                <div class="relative mt-1">
                                    <input type="text" id="filter-search" oninput="applyFilters()" placeholder="Ex: REF-2026-001..." class="w-full bg-slate-900 border border-slate-700 rounded-lg pl-9 pr-3 py-2 text-sm text-white focus:outline-none focus:border-sky-500">
                                    <i class="fa-solid fa-magnifying-glass absolute left-3 top-2.5 text-slate-500 text-sm"></i>
                                </div>
                            </div>

                            <div class="grid grid-cols-2 gap-2">
                                <div>
                                    <label class="text-xs text-slate-400 font-medium">Estado do Atraso</label>
                                    <select id="filter-delay" onchange="applyFilters()" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-2 py-2 text-sm text-white focus:outline-none focus:border-sky-500 mt-1">
                                        <option value="all">Todos</option>
                                        <option value="ontime">Em Dia (<= 0d)</option>
                                        <option value="delayed">Atrasados (> 0d)</option>
                                    </select>
                                </div>
                                <div>
                                    <label class="text-xs text-slate-400 font-medium">Transitário</label>
                                    <select id="filter-forwarder" onchange="applyFilters()" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-2 py-2 text-sm text-white focus:outline-none focus:border-sky-500 mt-1">
                                        <option value="all">Todos</option>
                                    </select>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div class="bg-slate-900/50 rounded-xl p-3 border border-slate-700/50 mt-4">
                        <h4 class="text-xs font-bold text-slate-300 uppercase tracking-wider mb-2">Legenda de Rastreio Ext.</h4>
                        <p class="text-xs text-slate-400 leading-relaxed">
                            Clica no botão <span class="text-sky-400 font-bold"><i class="fa-solid fa-arrow-up-right-from-square"></i> Track</span> num contentor para abrir diretamente no portal da Armadora (Maersk, MSC, CMA CGM) ou SeaRates.
                        </p>
                    </div>
                </div>

            </div>

            <!-- EMBARQUES CARDS WRAPPER -->
            <div id="shipments-container-list" class="space-y-6">
                <!-- Rendered Dynamically via JS -->
            </div>

        </div>

        <!-- VIEW 2: DOCUMENTATION CONTROL TAB -->
        <div id="view-documents" class="hidden space-y-6">
            <div class="bg-slate-800/80 border border-slate-700 rounded-2xl p-6 shadow-xl">
                <div class="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-6">
                    <div>
                        <h2 class="text-xl font-bold text-white flex items-center gap-2">
                            <i class="fa-solid fa-folder-closed text-amber-400"></i> Matriz de Controlo Documental por Embarque
                        </h2>
                        <p class="text-sm text-slate-400">Gerir a prontidão de envio de documentos. Escolha entre <strong>Enviado</strong>, <strong>Pendente</strong> ou <strong>Não Aplicável (N/A)</strong>.</p>
                    </div>
                    <div class="flex items-center gap-2">
                        <span class="text-xs text-slate-400">Documentos suportados:</span>
                        <span class="px-2 py-1 bg-slate-700 text-slate-200 text-xs rounded-md font-mono">BL, Cert. Origem, Packing List, Fatura, EUR1, ATR, AWB</span>
                    </div>
                </div>

                <div class="overflow-x-auto custom-scroll">
                    <table class="w-full text-left border-collapse">
                        <thead>
                            <tr class="bg-slate-900/80 text-slate-300 text-xs uppercase tracking-wider border-b border-slate-700">
                                <th class="p-4 rounded-tl-lg">Ref. Embarque</th>
                                <th class="p-4">Transitário / Linha</th>
                                <th class="p-4 text-center">Progresso Documental</th>
                                <th class="p-4 text-center">BL</th>
                                <th class="p-4 text-center">Cert. Origem</th>
                                <th class="p-4 text-center">Packing List</th>
                                <th class="p-4 text-center">Fatura</th>
                                <th class="p-4 text-center">EUR1</th>
                                <th class="p-4 text-center">ATR</th>
                                <th class="p-4 text-center">AWB</th>
                            </tr>
                        </thead>
                        <tbody id="documents-table-body" class="divide-y divide-slate-700/60 text-sm">
                            <!-- Rendered via JS -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- VIEW 3: ANALYTICS & DELAY METRICS -->
        <div id="view-analytics" class="hidden space-y-6">
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <!-- Chart 1: Delay Days per Carrier -->
                <div class="bg-slate-800/80 border border-slate-700 rounded-2xl p-6 shadow-xl">
                    <h3 class="font-bold text-slate-200 mb-4 flex items-center gap-2">
                        <i class="fa-solid fa-chart-column text-sky-400"></i> Média de Dias de Atraso por Companhia / Linha
                    </h3>
                    <div class="h-[280px]">
                        <canvas id="chart-carrier-delays"></canvas>
                    </div>
                </div>

                <!-- Chart 2: Delay Days per Forwarder -->
                <div class="bg-slate-800/80 border border-slate-700 rounded-2xl p-6 shadow-xl">
                    <h3 class="font-bold text-slate-200 mb-4 flex items-center gap-2">
                        <i class="fa-solid fa-chart-bar text-indigo-400"></i> Dias de Atraso Acumulados por Transitário
                    </h3>
                    <div class="h-[280px]">
                        <canvas id="chart-forwarder-delays"></canvas>
                    </div>
                </div>
            </div>
        </div>

    </main>

    <!-- MODAL: ADD / EDIT EMBARQUE -->
    <div id="modal-shipment" class="fixed inset-0 bg-slate-950/80 backdrop-blur-sm z-50 flex items-center justify-center hidden p-4">
        <div class="bg-slate-800 border border-slate-700 rounded-2xl w-full max-w-2xl max-h-[90vh] overflow-y-auto custom-scroll shadow-2xl p-6">
            <div class="flex items-center justify-between border-b border-slate-700 pb-4 mb-4">
                <h3 class="text-lg font-bold text-white flex items-center gap-2">
                    <i class="fa-solid fa-boxes-packing text-sky-400"></i> Criar Novo Embarque
                </h3>
                <button onclick="closeNewShipmentModal()" class="text-slate-400 hover:text-white">
                    <i class="fa-solid fa-xmark text-xl"></i>
                </button>
            </div>

            <form id="form-shipment" onsubmit="handleSaveShipment(event)" class="space-y-4">
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                        <label class="block text-xs font-semibold text-slate-300 mb-1">Referência do Embarque *</label>
                        <input type="text" id="m-ref" required placeholder="Ex: REF-2026-PT-09" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-2 text-sm text-white focus:outline-none focus:border-sky-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-300 mb-1">Rota (Origem -> Destino) *</label>
                        <input type="text" id="m-route" required placeholder="Ex: Ningbo -> Sines" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-2 text-sm text-white focus:outline-none focus:border-sky-500">
                    </div>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                        <label class="block text-xs font-semibold text-slate-300 mb-1">Transitário</label>
                        <input type="text" id="m-forwarder" placeholder="Ex: Kuehne+Nagel / DHL" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-2 text-sm text-white focus:outline-none focus:border-sky-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-300 mb-1">Companhia Marítima / Armadora</label>
                        <input type="text" id="m-carrier" placeholder="Ex: Maersk / MSC / CMA CGM" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-2 text-sm text-white focus:outline-none focus:border-sky-500">
                    </div>
                </div>

                <!-- Bookings & Contentores Section -->
                <div class="border-t border-slate-700 pt-4 mt-4">
                    <div class="flex items-center justify-between mb-2">
                        <label class="text-xs font-bold text-sky-400 uppercase tracking-wider">Bookings & Contentores</label>
                        <button type="button" onclick="addBookingFormRow()" class="text-xs bg-slate-700 hover:bg-slate-600 text-white px-2.5 py-1 rounded flex items-center gap-1">
                            <i class="fa-solid fa-plus text-sky-400"></i> Adicionar Booking
                        </button>
                    </div>

                    <div id="booking-rows-container" class="space-y-4">
                        <!-- Dynamic booking rows inserted here -->
                    </div>
                </div>

                <div class="flex justify-end gap-3 border-t border-slate-700 pt-4 mt-6">
                    <button type="button" onclick="closeNewShipmentModal()" class="px-4 py-2 bg-slate-700 hover:bg-slate-600 text-slate-200 rounded-lg text-sm font-semibold">Cancelar</button>
                    <button type="submit" class="px-5 py-2 bg-sky-600 hover:bg-sky-500 text-white rounded-lg text-sm font-semibold shadow-lg shadow-sky-600/30">Guardar Embarque</button>
                </div>
            </form>
        </div>
    </div>

    <!-- MAIN JAVASCRIPT LOGIC -->
    <script>
        // Sample Initial Data Structure (Embarque -> Bookings -> Containers)
        let sampleShipments = [
            {
                id: 'ship-1',
                reference: 'REF-2026-0891',
                forwarder: 'Kuehne + Nagel',
                carrier: 'MSC',
                route: 'Ningbo (CN) -> Sines (PT)',
                coordsOrigin: [29.8683, 121.5440],
                coordsDest: [37.9560, -8.8690],
                documents: {
                    bl: 'sent',           // 'sent', 'pending', 'na'
                    cert_origem: 'sent',
                    packing_list: 'sent',
                    fatura: 'sent',
                    eur1: 'na',
                    atr: 'na',
                    awb: 'na'
                },
                bookings: [
                    {
                        bookingNum: 'BKG-MSC-99012',
                        containers: [
                            {
                                containerNum: 'MSCU8819201',
                                type: "40' High Cube",
                                etd: '2026-10-01',
                                eta: '2026-10-25',
                                realETD: '2026-10-02',
                                realETA: '2026-10-29', // 4 days delay
                                status: 'Em Trânsito'
                            },
                            {
                                containerNum: 'MSCU1102934',
                                type: "20' Standard",
                                etd: '2026-10-01',
                                eta: '2026-10-25',
                                realETD: '2026-10-02',
                                realETA: '2026-10-29', // 4 days delay
                                status: 'Em Trânsito'
                            }
                        ]
                    }
                ]
            },
            {
                id: 'ship-2',
                reference: 'REF-2026-0904',
                forwarder: 'DSV Global',
                carrier: 'Maersk Line',
                route: 'Shanghai (CN) -> Leixões (PT)',
                coordsOrigin: [31.2304, 121.4737],
                coordsDest: [41.1843, -8.7011],
                documents: {
                    bl: 'sent',
                    cert_origem: 'pending',
                    packing_list: 'sent',
                    fatura: 'sent',
                    eur1: 'pending',
                    atr: 'na',
                    awb: 'na'
                },
                bookings: [
                    {
                        bookingNum: 'MAE-7710293',
                        containers: [
                            {
                                containerNum: 'MAEU4491028',
                                type: "40' High Cube",
                                etd: '2026-10-10',
                                eta: '2026-11-05',
                                realETD: '2026-10-10',
                                realETA: '2026-11-05', // 0 days delay
                                status: 'Em Trânsito'
                            }
                        ]
                    },
                    {
                        bookingNum: 'MAE-7710294',
                        containers: [
                            {
                                containerNum: 'MAEU9920194',
                                type: "40' Standard",
                                etd: '2026-10-12',
                                eta: '2026-11-07',
                                realETD: '2026-10-12',
                                realETA: '2026-11-07', // 0 days delay
                                status: 'Em Trânsito'
                            }
                        ]
                    }
                ]
            },
            {
                id: 'ship-3',
                reference: 'REF-2026-0740',
                forwarder: 'DHL Global Forwarding',
                carrier: 'CMA CGM',
                route: 'Santos (BR) -> Sines (PT)',
                coordsOrigin: [-23.9608, -46.3339],
                coordsDest: [37.9560, -8.8690],
                documents: {
                    bl: 'sent',
                    cert_origem: 'sent',
                    packing_list: 'sent',
                    fatura: 'sent',
                    eur1: 'sent',
                    atr: 'sent',
                    awb: 'na'
                },
                bookings: [
                    {
                        bookingNum: 'CMA-4410293',
                        containers: [
                            {
                                containerNum: 'CMAU1192039',
                                type: "20' Standard",
                                etd: '2026-09-15',
                                eta: '2026-10-05',
                                realETD: '2026-09-15',
                                realETA: '2026-10-12', // 7 days delay
                                status: 'Entregue'
                            }
                        ]
                    }
                ]
            }
        ];

        // Global State & Leaflet Map handle
        let shipmentsData = [...sampleShipments];
        let map;
        let mapMarkers = [];
        let carrierChartInstance = null;
        let forwarderChartInstance = null;

        // Initialize App on DOM Loaded
        document.addEventListener('DOMContentLoaded', () => {
            initMap();
            renderShipmentsList(shipmentsData);
            renderDocumentsTable();
            populateForwarderFilter();
            updateKPIs();
            initCharts();
        });

        // Tab Switching Logic
        function switchTab(tab) {
            const tabs = ['shipments', 'documents', 'analytics'];
            tabs.forEach(t => {
                const btn = document.getElementById(`tab-btn-${t}`);
                const view = document.getElementById(`view-${t}`);
                if (t === tab) {
                    btn.className = "px-4 py-2 text-sm font-semibold rounded-lg transition-all flex items-center gap-2 bg-sky-600 text-white shadow";
                    view.classList.remove('hidden');
                } else {
                    btn.className = "px-4 py-2 text-sm font-semibold rounded-lg transition-all flex items-center gap-2 text-slate-400 hover:text-white";
                    view.classList.add('hidden');
                }
            });

            if (tab === 'analytics') {
                updateCharts();
            }
        }

        /* -------------------------------------------------------------
           LEAFLET MAP INITIALIZATION (ESRI OCEAN BASEMAP - UNBLOCKED)
        ------------------------------------------------------------- */
        function initMap() {
            // Esri World Ocean Basemap (100% Free, High Quality, No User-Agent Block)
            map = L.map('map').setView([20.0, 0.0], 2);

            L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/Ocean/World_Ocean_Base/MapServer/tile/{z}/{y}/{x}', {
                maxZoom: 13,
                attribution: 'Tiles &copy; Esri &mdash; Sources: GEBCO, NOAA, CHS, OSM, National Geographic'
            }).addTo(map);

            updateMapMarkers();
        }

        function updateMapMarkers() {
            // Clear existing markers
            mapMarkers.forEach(m => map.removeLayer(m));
            mapMarkers = [];

            const today = new Date('2026-10-28'); // Simulated current date

            shipmentsData.forEach(ship => {
                ship.bookings.forEach(bkg => {
                    bkg.containers.forEach(cnt => {
                        const etd = new Date(cnt.etd);
                        const eta = new Date(cnt.eta);
                        
                        // Calculate progress ratio between 0 and 1
                        let progress = 0;
                        if (today >= eta) progress = 1;
                        else if (today <= etd) progress = 0;
                        else {
                            const totalDuration = eta - etd;
                            const elapsed = today - etd;
                            progress = elapsed / totalDuration;
                        }

                        // Interpolate lat/lng coordinates between origin & destination
                        const lat = ship.coordsOrigin[0] + (ship.coordsDest[0] - ship.coordsOrigin[0]) * progress;
                        const lng = ship.coordsOrigin[1] + (ship.coordsDest[1] - ship.coordsOrigin[1]) * progress;

                        // Create route polyline
                        const polyline = L.polyline([ship.coordsOrigin, ship.coordsDest], {
                            color: '#0284c7',
                            weight: 3,
                            dashArray: '5, 8',
                            opacity: 0.8
                        }).addTo(map);
                        mapMarkers.push(polyline);

                        // Custom Marker Icon
                        const customIcon = L.divIcon({
                            className: 'custom-ship-icon',
                            html: `<div style="background:#0284c7; color:white; border-radius:50%; width:28px; height:28px; display:flex; items-center justify-content:center; border:2px solid white; font-size:12px; box-shadow:0 2px 6px rgba(0,0,0,0.6);">
                                    <i class="fa-solid fa-ship" style="margin-top:7px; margin-left:6px;"></i>
                                   </div>`,
                            iconSize: [28, 28],
                            iconAnchor: [14, 14]
                        });

                        const delayDays = calculateDelayDays(cnt.eta, cnt.realETA);
                        const delayBadge = delayDays > 0 
                            ? `<span style="color:#ef4444; font-weight:bold;">Atraso: +${delayDays}d</span>`
                            : `<span style="color:#10b981; font-weight:bold;">Em dia</span>`;

                        const popupContent = `
                            <div style="font-family:sans-serif; font-size:12px; color:#0f172a;">
                                <strong>Ref: ${ship.reference}</strong><br/>
                                <span>Contentor: ${cnt.containerNum}</span><br/>
                                <span>Rota: ${ship.route}</span><br/>
                                <span>Progresso Estimado: ${Math.round(progress * 100)}%</span><br/>
                                ${delayBadge}
                            </div>
                        `;

                        const marker = L.marker([lat, lng], { icon: customIcon }).addTo(map).bindPopup(popupContent);
                        mapMarkers.push(marker);
                    });
                });
            });
        }

        /* -------------------------------------------------------------
           HELPER COMPUTATIONS
        ------------------------------------------------------------- */
        function calculateDelayDays(etaStr, realEtaStr) {
            if (!etaStr || !realEtaStr) return 0;
            const eta = new Date(etaStr);
            const realEta = new Date(realEtaStr);
            const diffTime = realEta - eta;
            const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
            return diffDays;
        }

        function getExternalTrackingUrl(carrierName, containerNum) {
            const cleanCarrier = (carrierName || '').toLowerCase();
            if (cleanCarrier.includes('maersk')) {
                return `https://www.maersk.com/tracking/${containerNum}`;
            } else if (cleanCarrier.includes('msc')) {
                return `https://www.msc.com/en/track-a-shipment?number=${containerNum}`;
            } else if (cleanCarrier.includes('cma')) {
                return `https://www.cma-cgm.com/ebusiness/tracking/search?SearchValue=${containerNum}`;
            }
            return `https://www.searates.com/container/tracking/?number=${containerNum}`;
        }

        /* -------------------------------------------------------------
           RENDER SHIPMENTS LIST & CARDS
        ------------------------------------------------------------- */
        function renderShipmentsList(data) {
            const container = document.getElementById('shipments-container-list');
            container.innerHTML = '';

            if (data.length === 0) {
                container.innerHTML = `
                    <div class="bg-slate-800/40 border border-slate-700/60 rounded-2xl p-8 text-center text-slate-400">
                        <i class="fa-solid fa-box-open text-4xl mb-3 text-slate-600"></i>
                        <p>Nenhum embarque encontrado para os filtros selecionados.</p>
                    </div>
                `;
                return;
            }

            data.forEach(ship => {
                let totalContainersInShipment = 0;
                let shipmentMaxDelay = 0;

                ship.bookings.forEach(b => {
                    totalContainersInShipment += b.containers.length;
                    b.containers.forEach(c => {
                        const d = calculateDelayDays(c.eta, c.realETA);
                        if (d > shipmentMaxDelay) shipmentMaxDelay = d;
                    });
                });

                const delayStatusBadge = shipmentMaxDelay > 0
                    ? `<span class="bg-rose-500/10 border border-rose-500/30 text-rose-400 px-3 py-1 rounded-full text-xs font-bold flex items-center gap-1.5">
                        <i class="fa-solid fa-triangle-exclamation"></i> Atrasado (+${shipmentMaxDelay} dias)
                       </span>`
                    : `<span class="bg-emerald-500/10 border border-emerald-500/30 text-emerald-400 px-3 py-1 rounded-full text-xs font-bold flex items-center gap-1.5">
                        <i class="fa-solid fa-circle-check"></i> Dentro do Prazo (On-Time)
                       </span>`;

                const shipCardHtml = `
                    <div class="bg-slate-800/80 border border-slate-700/80 rounded-2xl shadow-xl overflow-hidden">
                        <!-- Embarque Header -->
                        <div class="bg-slate-900/80 px-6 py-4 border-b border-slate-700/80 flex flex-col md:flex-row md:items-center justify-between gap-4">
                            <div class="flex items-center gap-3">
                                <span class="bg-sky-500/20 text-sky-400 border border-sky-500/30 px-3 py-1 rounded-lg font-mono font-bold text-sm">
                                    ${ship.reference}
                                </span>
                                <div>
                                    <h3 class="font-bold text-white text-base leading-tight">${ship.route}</h3>
                                    <div class="flex items-center gap-3 text-xs text-slate-400 mt-1">
                                        <span><i class="fa-solid fa-truck-ramp-box text-slate-500"></i> Transitário: <strong class="text-slate-200">${ship.forwarder || 'N/A'}</strong></span>
                                        <span>•</span>
                                        <span><i class="fa-solid fa-anchor text-slate-500"></i> Linha: <strong class="text-slate-200">${ship.carrier || 'N/A'}</strong></span>
                                    </div>
                                </div>
                            </div>

                            <div class="flex items-center gap-3">
                                ${delayStatusBadge}
                                <button onclick="deleteShipment('${ship.id}')" class="text-slate-500 hover:text-rose-400 p-2 text-sm transition-colors" title="Remover Embarque">
                                    <i class="fa-solid fa-trash"></i>
                                </button>
                            </div>
                        </div>

                        <!-- Bookings and Containers Container -->
                        <div class="p-6 space-y-6">
                            ${ship.bookings.map(bkg => `
                                <div class="bg-slate-900/40 border border-slate-700/50 rounded-xl p-4">
                                    <div class="flex items-center justify-between border-b border-slate-700/40 pb-3 mb-4">
                                        <div class="flex items-center gap-2">
                                            <i class="fa-solid fa-bookmark text-amber-400 text-sm"></i>
                                            <span class="text-xs text-slate-400 uppercase font-semibold">Booking:</span>
                                            <span class="text-sm font-bold text-white font-mono">${bkg.bookingNum}</span>
                                        </div>
                                        <span class="text-xs text-slate-400">${bkg.containers.length} Contentor(es)</span>
                                    </div>

                                    <!-- Containers Grid -->
                                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                                        ${bkg.containers.map(cnt => {
                                            const delay = calculateDelayDays(cnt.eta, cnt.realETA);
                                            const trackUrl = getExternalTrackingUrl(ship.carrier, cnt.containerNum);
                                            
                                            return `
                                                <div class="bg-slate-800 border border-slate-700 rounded-xl p-4 flex flex-col justify-between hover:border-slate-600 transition-all shadow-sm">
                                                    <div>
                                                        <div class="flex items-center justify-between mb-2">
                                                            <span class="font-mono font-bold text-sky-400 text-sm">${cnt.containerNum}</span>
                                                            <span class="text-[10px] bg-slate-700 text-slate-300 px-2 py-0.5 rounded">${cnt.type}</span>
                                                        </div>

                                                        <!-- Dates Grid -->
                                                        <div class="grid grid-cols-2 gap-2 text-xs my-3 bg-slate-900/60 p-2.5 rounded-lg border border-slate-700/40">
                                                            <div>
                                                                <span class="text-slate-400 text-[10px] uppercase font-semibold block">ETD (Acordado)</span>
                                                                <span class="text-slate-200 font-mono">${cnt.etd}</span>
                                                            </div>
                                                            <div>
                                                                <span class="text-slate-400 text-[10px] uppercase font-semibold block">ETD (Real)</span>
                                                                <span class="text-slate-200 font-mono">${cnt.realETD || '-'}</span>
                                                            </div>
                                                            <div>
                                                                <span class="text-slate-400 text-[10px] uppercase font-semibold block">ETA (Acordado)</span>
                                                                <span class="text-slate-200 font-mono">${cnt.eta}</span>
                                                            </div>
                                                            <div>
                                                                <span class="text-slate-400 text-[10px] uppercase font-semibold block">ETA (Real)</span>
                                                                <span class="text-slate-200 font-mono font-bold ${delay > 0 ? 'text-rose-400' : 'text-emerald-400'}">${cnt.realETA || '-'}</span>
                                                            </div>
                                                        </div>
                                                    </div>

                                                    <div class="flex items-center justify-between pt-2 border-t border-slate-700/50 mt-1">
                                                        <div class="text-xs">
                                                            ${delay > 0 
                                                                ? `<span class="text-rose-400 font-semibold"><i class="fa-solid fa-clock"></i> +${delay} dia(s)</span>`
                                                                : `<span class="text-emerald-400 font-semibold"><i class="fa-solid fa-check"></i> Sem Atraso</span>`
                                                            }
                                                        </div>
                                                        <a href="${trackUrl}" target="_blank" rel="noopener noreferrer" class="text-xs bg-sky-500/10 hover:bg-sky-500/20 text-sky-400 border border-sky-500/30 px-2.5 py-1 rounded flex items-center gap-1 transition-colors">
                                                            <i class="fa-solid fa-arrow-up-right-from-square text-[10px]"></i> Track
                                                        </a>
                                                    </div>
                                                </div>
                                            `;
                                        }).join('')}
                                    </div>
                                </div>
                            `).join('')}
                        </div>
                    </div>
                `;
                container.innerHTML += shipCardHtml;
            });
        }

        /* -------------------------------------------------------------
           DOCUMENTATION CONTROL TAB LOGIC WITH 3 STATES & AWB
        ------------------------------------------------------------- */
        function renderDocumentsTable() {
            const tbody = document.getElementById('documents-table-body');
            tbody.innerHTML = '';

            const docKeys = ['bl', 'cert_origem', 'packing_list', 'fatura', 'eur1', 'atr', 'awb'];

            shipmentsData.forEach(ship => {
                const docs = ship.documents || {};
                
                // Count active applicable docs
                let totalApplicable = 0;
                let totalSent = 0;

                docKeys.forEach(k => {
                    const status = docs[k] || 'pending';
                    if (status !== 'na') {
                        totalApplicable++;
                        if (status === 'sent') totalSent++;
                    }
                });

                const pct = totalApplicable > 0 ? Math.round((totalSent / totalApplicable) * 100) : 100;

                const tr = document.createElement('tr');
                tr.className = "hover:bg-slate-800/50 transition-colors";

                tr.innerHTML = `
                    <td class="p-4 font-bold text-sky-400 font-mono">${ship.reference}</td>
                    <td class="p-4">
                        <div class="text-xs font-semibold text-white">${ship.forwarder || 'N/A'}</div>
                        <div class="text-[11px] text-slate-400">${ship.carrier || 'N/A'}</div>
                    </td>
                    <td class="p-4 text-center">
                        <div class="w-full bg-slate-700 rounded-full h-2 mb-1 overflow-hidden">
                            <div class="bg-emerald-500 h-2 rounded-full" style="width: ${pct}%"></div>
                        </div>
                        <span class="text-xs font-mono font-bold text-slate-300">${pct}% (${totalSent}/${totalApplicable})</span>
                    </td>
                    ${docKeys.map(docKey => {
                        const status = docs[docKey] || 'pending';
                        return `
                            <td class="p-4 text-center">
                                <button onclick="toggleDocStatus('${ship.id}', '${docKey}')" class="focus:outline-none transition-transform active:scale-95">
                                    ${getDocBadge(status)}
                                </button>
                            </td>
                        `;
                    }).join('')}
                `;
                tbody.appendChild(tr);
            });
        }

        function getDocBadge(status) {
            if (status === 'sent') {
                return `<span class="inline-flex items-center justify-center w-8 h-8 rounded-lg bg-emerald-500/20 text-emerald-400 border border-emerald-500/30 hover:bg-emerald-500/30" title="Enviado (Clica para alterar)">
                            <i class="fa-solid fa-circle-check"></i>
                        </span>`;
            } else if (status === 'pending') {
                return `<span class="inline-flex items-center justify-center w-8 h-8 rounded-lg bg-rose-500/20 text-rose-400 border border-rose-500/30 hover:bg-rose-500/30" title="Pendente (Clica para alterar)">
                            <i class="fa-solid fa-circle-xmark"></i>
                        </span>`;
            } else {
                // N/A
                return `<span class="inline-flex items-center justify-center w-8 h-8 rounded-lg bg-slate-700/50 text-slate-400 border border-slate-600/40 hover:bg-slate-700" title="Não Aplicável (N/A)">
                            <span class="text-[10px] font-bold">N/A</span>
                        </span>`;
            }
        }

        function toggleDocStatus(shipmentId, docKey) {
            const ship = shipmentsData.find(s => s.id === shipmentId);
            if (ship) {
                if (!ship.documents) ship.documents = {};
                const current = ship.documents[docKey] || 'pending';
                // Cycle through: pending -> sent -> na -> pending
                if (current === 'pending') ship.documents[docKey] = 'sent';
                else if (current === 'sent') ship.documents[docKey] = 'na';
                else ship.documents[docKey] = 'pending';

                renderDocumentsTable();
                updateKPIs();
            }
        }

        /* -------------------------------------------------------------
           FILTERS & KPI UPDATES
        ------------------------------------------------------------- */
        function populateForwarderFilter() {
            const select = document.getElementById('filter-forwarder');
            const forwarders = [...new Set(shipmentsData.map(s => s.forwarder).filter(Boolean))];
            
            select.innerHTML = '<option value="all">Todos</option>';
            forwarders.forEach(f => {
                const opt = document.createElement('option');
                opt.value = f;
                opt.textContent = f;
                select.appendChild(opt);
            });
        }

        function applyFilters() {
            const searchVal = document.getElementById('filter-search').value.toLowerCase();
            const delayVal = document.getElementById('filter-delay').value;
            const forwarderVal = document.getElementById('filter-forwarder').value;

            const filtered = shipmentsData.filter(ship => {
                // Search check
                const matchSearch = ship.reference.toLowerCase().includes(searchVal) ||
                    ship.forwarder.toLowerCase().includes(searchVal) ||
                    ship.bookings.some(b => b.bookingNum.toLowerCase().includes(searchVal) || b.containers.some(c => c.containerNum.toLowerCase().includes(searchVal)));

                // Forwarder check
                const matchForwarder = forwarderVal === 'all' || ship.forwarder === forwarderVal;

                // Delay check
                let maxDelay = 0;
                ship.bookings.forEach(b => {
                    b.containers.forEach(c => {
                        const d = calculateDelayDays(c.eta, c.realETA);
                        if (d > maxDelay) maxDelay = d;
                    });
                });

                let matchDelay = true;
                if (delayVal === 'ontime') matchDelay = maxDelay <= 0;
                if (delayVal === 'delayed') matchDelay = maxDelay > 0;

                return matchSearch && matchForwarder && matchDelay;
            });

            renderShipmentsList(filtered);
        }

        function updateKPIs() {
            let totalShipments = shipmentsData.length;
            let totalContainers = 0;
            let onTimeCount = 0;
            let delayedCount = 0;
            let pendingDocsCount = 0;

            const docKeys = ['bl', 'cert_origem', 'packing_list', 'fatura', 'eur1', 'atr', 'awb'];

            shipmentsData.forEach(ship => {
                // Count docs
                if (ship.documents) {
                    docKeys.forEach(k => {
                        if (ship.documents[k] === 'pending') pendingDocsCount++;
                    });
                }

                ship.bookings.forEach(b => {
                    totalContainers += b.containers.length;
                    b.containers.forEach(c => {
                        const delay = calculateDelayDays(c.eta, c.realETA);
                        if (delay > 0) delayedCount++;
                        else onTimeCount++;
                    });
                });
            });

            document.getElementById('kpi-total-shipments').textContent = totalShipments;
            document.getElementById('kpi-total-containers').textContent = totalContainers;
            document.getElementById('kpi-ontime').textContent = onTimeCount;
            document.getElementById('kpi-delayed').textContent = delayedCount;
            document.getElementById('kpi-pending-docs').textContent = pendingDocsCount;
        }

        /* -------------------------------------------------------------
           MODAL & FORM CREATION
        ------------------------------------------------------------- */
        let bookingRowCount = 0;

        function openNewShipmentModal() {
            document.getElementById('form-shipment').reset();
            document.getElementById('booking-rows-container').innerHTML = '';
            bookingRowCount = 0;
            addBookingFormRow(); // Add first default booking
            document.getElementById('modal-shipment').classList.remove('hidden');
        }

        function closeNewShipmentModal() {
            document.getElementById('modal-shipment').classList.add('hidden');
        }

        function addBookingFormRow() {
            bookingRowCount++;
            const container = document.getElementById('booking-rows-container');
            const rowId = `bkg-row-${bookingRowCount}`;

            const html = `
                <div id="${rowId}" class="bg-slate-900/80 p-3 rounded-xl border border-slate-700 space-y-3">
                    <div class="flex items-center justify-between">
                        <span class="text-xs font-bold text-slate-300"># Booking ${bookingRowCount}</span>
                        <button type="button" onclick="document.getElementById('${rowId}').remove()" class="text-slate-500 hover:text-rose-400 text-xs">
                            <i class="fa-solid fa-trash"></i> Remover Booking
                        </button>
                    </div>
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
                        <div>
                            <label class="text-[11px] text-slate-400 block">Nº Booking *</label>
                            <input type="text" required class="input-bkg-num w-full bg-slate-800 border border-slate-700 rounded px-2.5 py-1.5 text-xs text-white">
                        </div>
                        <div>
                            <label class="text-[11px] text-slate-400 block">Nº Contentor *</label>
                            <input type="text" required placeholder="Ex: MSCU1234567" class="input-cnt-num w-full bg-slate-800 border border-slate-700 rounded px-2.5 py-1.5 text-xs text-white font-mono">
                        </div>
                    </div>
                    <div class="grid grid-cols-2 md:grid-cols-4 gap-2">
                        <div>
                            <label class="text-[10px] text-slate-400 block">ETD Acordado</label>
                            <input type="date" required class="input-etd w-full bg-slate-800 border border-slate-700 rounded px-2 py-1 text-xs text-white">
                        </div>
                        <div>
                            <label class="text-[10px] text-slate-400 block">ETA Acordado</label>
                            <input type="date" required class="input-eta w-full bg-slate-800 border border-slate-700 rounded px-2 py-1 text-xs text-white">
                        </div>
                        <div>
                            <label class="text-[10px] text-slate-400 block">ETD Real</label>
                            <input type="date" class="input-real-etd w-full bg-slate-800 border border-slate-700 rounded px-2 py-1 text-xs text-white">
                        </div>
                        <div>
                            <label class="text-[10px] text-slate-400 block">ETA Real</label>
                            <input type="date" class="input-real-eta w-full bg-slate-800 border border-slate-700 rounded px-2 py-1 text-xs text-white">
                        </div>
                    </div>
                </div>
            `;
            container.insertAdjacentHTML('beforeend', html);
        }

        function handleSaveShipment(e) {
            e.preventDefault();
            
            const ref = document.getElementById('m-ref').value;
            const route = document.getElementById('m-route').value;
            const forwarder = document.getElementById('m-forwarder').value;
            const carrier = document.getElementById('m-carrier').value;

            const bookingElements = document.querySelectorAll('#booking-rows-container > div');
            const bookings = [];

            bookingElements.forEach(el => {
                const bkgNum = el.querySelector('.input-bkg-num').value;
                const cntNum = el.querySelector('.input-cnt-num').value;
                const etd = el.querySelector('.input-etd').value;
                const eta = el.querySelector('.input-eta').value;
                const realEtd = el.querySelector('.input-real-etd').value || etd;
                const realEta = el.querySelector('.input-real-eta').value || eta;

                bookings.push({
                    bookingNum: bkgNum,
                    containers: [
                        {
                            containerNum: cntNum,
                            type: "40' High Cube",
                            etd: etd,
                            eta: eta,
                            realETD: realEtd,
                            realETA: realEta,
                            status: 'Em Trânsito'
                        }
                    ]
                });
            });

            const newShip = {
                id: 'ship-' + Date.now(),
                reference: ref,
                forwarder: forwarder,
                carrier: carrier,
                route: route,
                coordsOrigin: [31.2304, 121.4737], // Default origin/dest coordinates
                coordsDest: [37.9560, -8.8690],
                documents: {
                    bl: 'pending',
                    cert_origem: 'pending',
                    packing_list: 'pending',
                    fatura: 'pending',
                    eur1: 'na',
                    atr: 'na',
                    awb: 'na'
                },
                bookings: bookings
            };

            shipmentsData.unshift(newShip);
            
            renderShipmentsList(shipmentsData);
            renderDocumentsTable();
            populateForwarderFilter();
            updateKPIs();
            updateMapMarkers();
            closeNewShipmentModal();
        }

        function deleteShipment(shipmentId) {
            if (confirm('Tem a certeza que pretende eliminar este embarque?')) {
                shipmentsData = shipmentsData.filter(s => s.id !== shipmentId);
                renderShipmentsList(shipmentsData);
                renderDocumentsTable();
                populateForwarderFilter();
                updateKPIs();
                updateMapMarkers();
            }
        }

        /* -------------------------------------------------------------
           ANALYTICS CHARTS (CHART.JS)
        ------------------------------------------------------------- */
        function initCharts() {
            const ctx1 = document.getElementById('chart-carrier-delays').getContext('2d');
            carrierChartInstance = new Chart(ctx1, {
                type: 'bar',
                data: {
                    labels: [],
                    datasets: [{
                        label: 'Dias Médios de Atraso',
                        data: [],
                        backgroundColor: '#38bdf8',
                        borderRadius: 6
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { display: false }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            grid: { color: '#334155' },
                            ticks: { color: '#94a3b8' }
                        },
                        x: {
                            grid: { display: false },
                            ticks: { color: '#94a3b8' }
                        }
                    }
                }
            });

            const ctx2 = document.getElementById('chart-forwarder-delays').getContext('2d');
            forwarderChartInstance = new Chart(ctx2, {
                type: 'bar',
                data: {
                    labels: [],
                    datasets: [{
                        label: 'Total Dias de Atraso',
                        data: [],
                        backgroundColor: '#818cf8',
                        borderRadius: 6
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { display: false }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            grid: { color: '#334155' },
                            ticks: { color: '#94a3b8' }
                        },
                        x: {
                            grid: { display: false },
                            ticks: { color: '#94a3b8' }
                        }
                    }
                }
            });
        }

        function updateCharts() {
            if (!carrierChartInstance || !forwarderChartInstance) return;

            // Calculate Carrier Delays
            const carrierStats = {};
            const forwarderStats = {};

            shipmentsData.forEach(ship => {
                const c = ship.carrier || 'Desconhecido';
                const f = ship.forwarder || 'Desconhecido';

                if (!carrierStats[c]) carrierStats[c] = { totalDelay: 0, count: 0 };
                if (!forwarderStats[f]) forwarderStats[f] = { totalDelay: 0 };

                ship.bookings.forEach(b => {
                    b.containers.forEach(cnt => {
                        const delay = calculateDelayDays(cnt.eta, cnt.realETA);
                        if (delay > 0) {
                            carrierStats[c].totalDelay += delay;
                            forwarderStats[f].totalDelay += delay;
                        }
                        carrierStats[c].count++;
                    });
                });
            });

            // Carrier Chart Data
            const carrierLabels = Object.keys(carrierStats);
            const carrierAvgDelays = carrierLabels.map(k => (carrierStats[k].totalDelay / (carrierStats[k].count || 1)).toFixed(1));

            carrierChartInstance.data.labels = carrierLabels;
            carrierChartInstance.data.datasets[0].data = carrierAvgDelays;
            carrierChartInstance.update();

            // Forwarder Chart Data
            const forwarderLabels = Object.keys(forwarderStats);
            const forwarderTotals = forwarderLabels.map(k => forwarderStats[k].totalDelay);

            forwarderChartInstance.data.labels = forwarderLabels;
            forwarderChartInstance.data.datasets[0].data = forwarderTotals;
            forwarderChartInstance.update();
        }
    </script>
</body>
</html>
