# smart-civic-triage-chd
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Smart Civic Triage - Chandigarh</title>
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Leaflet Maps CSS & JS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script src="https://unpkg.com/leaflet.heat@0.2.0/dist/leaflet-heat.js"></script>
    
    <style>
        body { font-family: 'Inter', system-ui, sans-serif; background-color: #f0fdf4; transition: background-color 0.3s; overscroll-behavior: none; }
        .body-admin-mode { background-color: #f8fafc; } 
        .sr-only { position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px; overflow: hidden; clip: rect(0, 0, 0, 0); white-space: nowrap; border-width: 0; }
        .glass-card { background: rgba(255, 255, 255, 0.95); backdrop-filter: blur(10px); box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.1); }
        
        /* Uber-Style Map Marker Animations */
        .map-center-pin { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -100%); z-index: 1000; pointer-events: none; filter: drop-shadow(0 4px 4px rgba(0,0,0,0.3)); transition: transform 0.2s ease;}
        .map-container.is-dragging .map-center-pin { transform: translate(-50%, -120%) scale(1.1); }
        .pulse-shadow { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 20px; height: 10px; background: rgba(0,0,0,0.2); border-radius: 50%; z-index: 999; pointer-events: none; }
        
        /* Status Badges */
        .status-new { background-color: #fee2e2; color: #991b1b; }
        .status-progress { background-color: #fef3c7; color: #92400e; }
        .status-resolved { background-color: #dcfce3; color: #166534; }
    </style>
</head>
<body class="min-h-screen flex flex-col items-center justify-center p-4">

    <!-- ========================================== -->
    <!-- CITIZEN VIEW (Mobile First)                -->
    <!-- ========================================== -->
    <div id="citizenApp" class="glass-card w-full max-w-md rounded-2xl p-6 border border-green-100 relative transition-all">
        
        <!-- DPI Accessibility: Language Toggle -->
        <div class="absolute top-4 right-4 flex items-center gap-3">
            <select id="langSelector" onchange="changeLanguage(this.value)" class="text-xs bg-white border border-gray-200 rounded px-2 py-1 text-gray-700 outline-none cursor-pointer shadow-sm font-medium">
                <option value="en">English</option>
                <option value="hi">हिंदी</option>
                <option value="pa">ਪੰਜਾਬੀ</option>
            </select>
            <button type="button" id="goToAdminBtn" class="text-xs font-bold text-gray-400 hover:text-blue-600 transition-colors">
                Municipal Login &rarr;
            </button>
        </div>

        <!-- Header -->
        <div class="text-center mb-6 mt-4">
            <div class="w-16 h-16 bg-green-100 text-green-600 rounded-full flex items-center justify-center mx-auto mb-3 shadow-inner">
                <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z"></path></svg>
            </div>
            <h1 id="appTitle" class="text-2xl font-extrabold text-gray-800">Smart Triage</h1>
            <p id="appSubtitle" class="text-sm text-gray-500 mt-1">Snap a photo. AI handles the rest.</p>
        </div>

        <div id="alertBox" class="hidden mb-4 p-4 rounded-xl text-sm font-medium transition-all duration-300"></div>

        <!-- Main Form Area -->
        <div id="formSection" class="space-y-6">
            
            <div class="space-y-2">
                <label id="lblPhoto" class="block text-sm font-semibold text-gray-700 text-center mb-3">Provide Photo Evidence</label>
                <div class="flex gap-3">
                    <!-- Camera Button -->
                    <label for="cameraInput" class="flex-1 flex flex-col items-center justify-center p-4 bg-green-50 border-2 border-dashed border-green-300 rounded-xl cursor-pointer hover:bg-green-100 transition-colors">
                        <svg class="w-6 h-6 text-green-600 mb-2" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 9a2 2 0 012-2h.93a2 2 0 001.664-.89l.812-1.22A2 2 0 0110.07 4h3.86a2 2 0 011.664.89l.812 1.22A2 2 0 0018.07 7H19a2 2 0 012 2v9a2 2 0 01-2 2H5a2 2 0 01-2-2V9z"></path></svg>
                        <span id="btnCamera" class="text-sm font-semibold text-green-700">Camera</span>
                    </label>
                    <input type="file" id="cameraInput" accept="image/*" capture="environment" class="sr-only">

                    <!-- Gallery Button -->
                    <label for="galleryInput" class="flex-1 flex flex-col items-center justify-center p-4 bg-blue-50 border-2 border-dashed border-blue-300 rounded-xl cursor-pointer hover:bg-blue-100 transition-colors">
                        <svg class="w-6 h-6 text-blue-600 mb-2" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"></path></svg>
                        <span id="btnGallery" class="text-sm font-semibold text-blue-700">Gallery</span>
                    </label>
                    <input type="file" id="galleryInput" accept="image/*" class="sr-only">
                </div>
            </div>

            <div id="previewArea" class="hidden relative rounded-xl overflow-hidden shadow-sm border border-gray-200 bg-gray-50">
                <img id="imagePreview" src="" alt="Preview" class="w-full h-48 object-cover">
                <button type="button" id="removeBtn" class="absolute top-2 right-2 bg-red-500 text-white p-1.5 rounded-full shadow-md hover:bg-red-600">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                </button>
            </div>

            <!-- Uber-Style Map Section -->
            <div id="mapSection" class="hidden space-y-3 animate-fade-in mt-2">
                <div class="flex justify-between items-end px-1">
                    <label id="lblPinpoint" class="block text-sm font-semibold text-gray-700">Confirm Location</label>
                    <button type="button" id="recenterBtn" class="text-xs text-blue-600 font-bold bg-blue-50 px-3 py-1.5 rounded-lg hover:bg-blue-100 flex items-center gap-1 transition-colors">
                        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z"></path></svg>
                        <span id="btnLocate">Locate Me</span>
                    </button>
                </div>
                
                <div class="h-64 w-full rounded-2xl border-2 border-gray-200 overflow-hidden relative shadow-inner map-container" id="mapWrapper">
                    <div id="map" class="h-full w-full z-0"></div>
                    
                    <!-- Fixed Center Pin (Uber Style) -->
                    <div class="pulse-shadow"></div>
                    <svg class="map-center-pin w-10 h-10 text-red-500" fill="currentColor" viewBox="0 0 24 24">
                        <path d="M12 0c-4.198 0-8 3.403-8 7.602 0 4.198 3.469 9.21 8 16.398 4.531-7.188 8-12.2 8-16.398 0-4.199-3.801-7.602-8-7.602zm0 11c-1.657 0-3-1.343-3-3s1.343-3 3-3 3 1.343 3 3-1.343 3-3 3z"/>
                    </svg>

                    <!-- Floating Address Card -->
                    <div class="absolute bottom-3 left-3 right-3 bg-white/95 backdrop-blur shadow-lg p-3 rounded-xl z-[1000] border border-gray-100 flex items-center gap-3 pointer-events-none">
                        <div class="bg-red-50 text-red-500 p-2 rounded-lg flex-shrink-0">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 20l-5.447-2.724A1 1 0 013 16.382V5.618a1 1 0 011.447-.894L9 7m0 13l6-3m-6 3V7m6 10l4.553 2.276A1 1 0 0021 18.382V7.618a1 1 0 00-.553-.894L15 4m0 13V4m0 0L9 7"></path></svg>
                        </div>
                        <div class="flex-1 overflow-hidden">
                            <p id="sectorText" class="text-sm font-extrabold text-gray-800 truncate">Fetching Location...</p>
                            <p id="addressText" class="text-xs text-gray-500 truncate mt-0.5">Move map to adjust</p>
                        </div>
                    </div>
                </div>
            </div>

            <button type="button" id="analyzeBtn" class="w-full bg-green-600 text-white font-bold text-lg py-4 rounded-xl shadow-lg hover:bg-green-700 transition-all disabled:opacity-50 disabled:cursor-not-allowed flex items-center justify-center gap-2">
                <span>Analyze & Report Issue</span>
                <svg id="spinner" class="hidden w-5 h-5 animate-spin" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
            </button>
        </div>

        <!-- Results Section -->
        <div id="resultsSection" class="hidden space-y-4">
            
            <div class="bg-blue-50 border border-blue-200 rounded-xl p-4 text-center relative overflow-hidden">
                <div class="absolute -right-4 -top-4 w-16 h-16 bg-blue-200 rounded-full opacity-50"></div>
                <h3 id="lblTicketTitle" class="text-xs font-bold text-blue-500 uppercase tracking-wider mb-1 relative z-10">Citizen Tracking Ticket</h3>
                <p id="resTicketId" class="text-2xl font-mono font-bold text-blue-800 tracking-widest relative z-10"></p>
                <p id="lblScreenshot" class="text-[11px] text-blue-600 mt-2 font-medium relative z-10">Take a screenshot to track your report status</p>
            </div>

            <div class="bg-gray-50 border border-gray-200 rounded-xl p-4">
                <h3 id="lblAiResults" class="text-xs font-bold text-gray-400 uppercase tracking-wider mb-3">AI Analysis Results</h3>
                <div class="flex justify-between items-center mb-2">
                    <span id="lblDetectedIssue" class="text-sm text-gray-600">Detected Issue:</span>
                    <span id="resAiType" class="font-bold text-gray-800 bg-gray-200 px-2 py-1 rounded text-xs capitalize"></span>
                </div>
                <div class="flex justify-between items-center">
                    <span id="lblSeverityScore" class="text-sm text-gray-600">Severity Score:</span>
                    <div class="flex items-center gap-2">
                        <div class="w-16 h-2 bg-gray-200 rounded-full overflow-hidden">
                            <div id="resAiSeverityBar" class="h-full bg-red-500" style="width: 0%"></div>
                        </div>
                        <span id="resAiSeverity" class="font-bold text-red-600 text-sm"></span>
                    </div>
                </div>
            </div>

            <div class="bg-green-50 border border-green-200 rounded-xl p-4">
                <h3 id="lblMuniRouting" class="text-xs font-bold text-green-500 uppercase tracking-wider mb-3">Municipal Routing</h3>
                <div class="flex justify-between items-center mb-2">
                    <span id="lblAssignedSector" class="text-sm text-gray-600">Assigned Sector:</span>
                    <span id="resSector" class="font-bold text-gray-800"></span>
                </div>
                <div class="flex justify-between items-center mb-2">
                    <span id="lblCalcPriority" class="text-sm text-gray-600">Calculated Priority:</span>
                    <span id="resPriority" class="font-bold text-gray-800"></span>
                </div>
                <div class="flex justify-between items-center">
                    <span id="lblDispatchAction" class="text-sm text-gray-600">Dispatch Action:</span>
                    <span id="resAction" class="font-bold text-green-700 bg-green-100 px-2 py-1 rounded text-xs"></span>
                </div>
            </div>

            <button type="button" id="resetBtn" class="w-full bg-gray-100 text-gray-700 font-bold py-3 rounded-xl hover:bg-gray-200 transition-all mt-4">Report Another Issue</button>
        </div>
    </div>


    <!-- ========================================== -->
    <!-- ADMIN LOGIN PORTAL                         -->
    <!-- ========================================== -->
    <div id="adminLogin" class="hidden glass-card w-full max-w-md rounded-2xl p-8 border border-slate-200 relative">
        <button type="button" id="backToCitizenBtn" class="absolute top-4 left-4 text-xs font-bold text-gray-400 hover:text-slate-600 transition-colors">
            &larr; Back to Citizen View
        </button>

        <div class="text-center mb-8 mt-4">
            <div class="w-16 h-16 bg-slate-800 text-white rounded-xl flex items-center justify-center mx-auto mb-4 shadow-lg">
                <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"></path></svg>
            </div>
            <h1 class="text-2xl font-extrabold text-slate-800">Authority Portal</h1>
            <p class="text-sm text-slate-500 mt-1">Authorized municipal personnel only.</p>
        </div>

        <div id="loginAlert" class="hidden mb-4 p-3 rounded-lg text-sm text-red-600 bg-red-50 border border-red-100 text-center"></div>

        <form id="adminLoginForm" class="space-y-4">
            <div>
                <label class="block text-sm font-semibold text-slate-700 mb-1">Username</label>
                <input type="text" id="adminUser" placeholder="username" class="w-full px-4 py-3 rounded-xl border border-slate-300 focus:ring-2 focus:ring-slate-500 focus:border-slate-500 outline-none transition-all">
            </div>
            <div>
                <label class="block text-sm font-semibold text-slate-700 mb-1">Password</label>
                <input type="password" id="adminPass" placeholder="password" class="w-full px-4 py-3 rounded-xl border border-slate-300 focus:ring-2 focus:ring-slate-500 focus:border-slate-500 outline-none transition-all">
            </div>
            <button type="submit" class="w-full bg-slate-800 text-white font-bold text-lg py-3.5 rounded-xl shadow-lg hover:bg-slate-900 transition-all mt-4">
                Login to Dashboard
            </button>
        </form>
    </div>


    <!-- ========================================== -->
    <!-- ADMIN DASHBOARD                            -->
    <!-- ========================================== -->
    <div id="adminDashboard" class="hidden w-full max-w-5xl space-y-6">
        
        <div class="flex flex-col md:flex-row justify-between items-start md:items-end gap-4 mb-6">
            <div>
                <h1 class="text-3xl font-extrabold text-slate-800">Incident Command</h1>
                <p class="text-slate-500">Chandigarh Municipal Corporation</p>
            </div>
            <button id="logoutBtn" class="text-sm font-bold text-red-500 bg-red-50 px-4 py-2 rounded-lg hover:bg-red-100 transition-colors">
                Logout
            </button>
        </div>

        <div id="adminAlertBox" class="hidden mb-4 p-4 rounded-xl text-sm font-medium transition-all duration-300 bg-blue-50 text-blue-700 border border-blue-200 shadow-sm text-center"></div>

        <div class="grid grid-cols-2 md:grid-cols-3 gap-4">
            <div class="bg-white p-4 md:p-5 rounded-2xl shadow-sm border border-slate-100">
                <p class="text-[10px] md:text-xs font-bold text-slate-400 uppercase tracking-wide">Active Tickets</p>
                <p class="text-2xl md:text-3xl font-black text-slate-700 mt-1" id="statTotal">0</p>
            </div>
            <div class="bg-white p-4 md:p-5 rounded-2xl shadow-sm border border-slate-100">
                <p class="text-[10px] md:text-xs font-bold text-slate-400 uppercase tracking-wide">Critical Issues</p>
                <p class="text-2xl md:text-3xl font-black text-red-600 mt-1" id="statCritical">0</p>
            </div>
            <div class="bg-white p-4 md:p-5 rounded-2xl shadow-sm border border-slate-100 col-span-2 md:col-span-1">
                <p class="text-[10px] md:text-xs font-bold text-slate-400 uppercase tracking-wide">Resolved Today</p>
                <p class="text-2xl md:text-3xl font-black text-green-600 mt-1" id="statResolved">0</p>
            </div>
        </div>

        <div class="flex border-b border-slate-200 mt-6 overflow-x-auto">
            <button onclick="switchAdminTab('tickets')" id="tabTickets" class="whitespace-nowrap px-6 py-3 font-bold text-sm border-b-2 border-blue-600 text-blue-600">Active Tickets</button>
            <button onclick="switchAdminTab('heatmap')" id="tabHeatmap" class="whitespace-nowrap px-6 py-3 font-bold text-sm border-b-2 border-transparent text-slate-500 hover:text-slate-700 flex gap-2 items-center">
                <svg class="w-4 h-4 text-orange-500" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M12.395 2.553a1 1 0 00-1.45-.385c-.345.23-.614.558-.822.88-.214.33-.403.713-.57 1.116-.334.804-.614 1.768-.84 2.734a31.365 31.365 0 00-.613 3.58 2.64 2.64 0 01-.945-1.067c-.328-.68-.398-1.534-.398-2.654A1 1 0 005.05 6.05 6.981 6.981 0 003 11a7 7 0 1011.95-4.95c-.592-.591-.98-.985-1.348-1.467-.363-.476-.724-1.063-1.207-2.03zM12.12 15.12A3 3 0 017 13s.879.5 2.5.5c0-1 .5-4 1.25-4.5.5 1 .786 1.293 1.371 1.879A2.99 2.99 0 0113 13a2.99 2.99 0 01-.879 2.121z" clip-rule="evenodd"></path></svg>
                AI Predictive Heatmap
            </button>
        </div>

        <div id="viewTickets" class="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden mt-4">
            <div class="px-6 py-4 border-b border-slate-100 flex justify-between items-center bg-slate-50">
                <h2 class="font-bold text-slate-700">Citizen Tracking Tickets</h2>
                <div class="text-xs text-slate-500 font-medium">Live Feed</div>
            </div>
            
            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse min-w-[600px]">
                    <thead>
                        <tr class="bg-slate-50 text-slate-400 text-xs uppercase tracking-wider border-b border-slate-100">
                            <th class="p-4 font-semibold">Ticket ID</th>
                            <th class="p-4 font-semibold">Issue Type</th>
                            <th class="p-4 font-semibold">Location</th>
                            <th class="p-4 font-semibold">Priority</th>
                            <th class="p-4 font-semibold">Status</th>
                            <th class="p-4 font-semibold">Action</th>
                        </tr>
                    </thead>
                    <tbody id="ticketsTableBody" class="divide-y divide-slate-100">
                        <!-- Tickets injected here -->
                    </tbody>
                </table>
            </div>
        </div>

        <div id="viewHeatmap" class="hidden bg-white rounded-2xl shadow-sm border border-slate-200 p-2 mt-4 relative">
            <div class="absolute top-4 left-4 z-[1000] bg-white/90 backdrop-blur p-3 rounded-lg border border-slate-200 shadow-sm">
                <h3 class="font-bold text-slate-800 text-sm">Risk Forecast (Next 72h)</h3>
                <p class="text-xs text-slate-500 mt-1 flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-red-500 inline-block"></span> High Probability</p>
                <p class="text-xs text-slate-500 mt-1 flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-yellow-400 inline-block"></span> Moderate Risk</p>
            </div>
            <div id="adminHeatmap" class="w-full h-[500px] rounded-xl z-0"></div>
        </div>
    </div>

    <script>
        // ==========================================
        // CONFIGURATION
        // ==========================================
        // LEAVE BLANK IN IMMERSIVE CHAT - It auto-injects to bypass demo mode!
        // If testing on Github Pages, paste your real key here!
        const GEMINI_API_KEY = ""; 

        // ==========================================
        // SHARED STATE & DATA
        // ==========================================
        
        let globalTickets = [
            { id: '#CHD-2026-9912', type: 'Severe Pothole', sector: 'Sector 22', priority: 82.5, status: 'new', date: 'Just now' },
            { id: '#CHD-2026-8834', type: 'Waterlogging', sector: 'Sector 17', priority: 95.0, status: 'in-progress', date: '2 hrs ago' },
            { id: '#CHD-2026-1123', type: 'Street Light', sector: 'Sector 35', priority: 45.2, status: 'resolved', date: 'Yesterday' }
        ];

        let currentIssueRaw = "";
        let currentActionRaw = "";

        const termDict = {
            'pothole': { en: 'Pothole', hi: 'गड्ढा', pa: 'ਟੋਆ' },
            'waterlogging': { en: 'Waterlogging', hi: 'जलभराव', pa: 'ਪਾਣੀ ਭਰਨਾ' },
            'garbage': { en: 'Garbage', hi: 'कचरा', pa: 'ਕੂੜਾ' },
            'overflowing_garbage': { en: 'Overflowing Garbage', hi: 'कचरा ओवरफ्लो', pa: 'ਕੂੜਾ ਓਵਰਫਲੋ' },
            'broken_street_light': { en: 'Broken Street Light', hi: 'खराब स्ट्रीट लाइट', pa: 'ਖਰਾਬ ਸਟਰੀਟ ਲਾਈਟ' },
            'dust_accumulation': { en: 'Excessive Dust/Debris', hi: 'अत्यधिक धूल/मलबा', pa: 'ਬਹੁਤ ਜ਼ਿਆਦਾ ਧੂੜ/ਮਲਬਾ' },
            'damaged_road': { en: 'Damaged Road', hi: 'क्षतिग्रस्त सड़क', pa: 'ਨੁਕਸਾਨੀ ਸੜਕ' },
            'construction_road_defects': { en: 'Construction Defects', hi: 'सड़क निर्माण दोष', pa: 'ਸੜਕ ਨਿਰਮਾਣ ਦੇ ਨੁਕਸ' },
            'ruttings': { en: 'Rutting (Depression)', hi: 'सड़क में रटिंग/धंसना', pa: 'ਸੜਕ ਦਾ ਧੱਸਣਾ (Rutting)' },
            'raveling': { en: 'Raveling (Surface Wear)', hi: 'रेवलिंग (कंक्रीट उखड़ना)', pa: 'ਸੜਕ ਦਾ ਉਖੜਨਾ (Raveling)' },
            'alligator_cracking': { en: 'Alligator Cracking', hi: 'ऑलिगेटर क्रैकिंग (जालीदार दरारें)', pa: 'ਜਾਲੀਦਾਰ ਤਰੇੜਾਂ (Alligator Cracking)' },
            'drainage_related': { en: 'Drainage Issue', hi: 'जल निकासी समस्या', pa: 'ਪਾਣੀ ਦੀ ਨਿਕਾਸੀ ਦੀ ਸਮੱਸਿਆ' },
            'traffic_related': { en: 'Traffic Infrastructure', hi: 'यातायात संबंधित समस्या', pa: 'ਟ੍ਰੈਫਿਕ ਬੁਨਿਆਦੀ ਢਾਂਚੇ ਦੀ ਸਮੱਸਿਆ' },
            'IMMEDIATE_DISPATCH': { en: 'IMMEDIATE DISPATCH', hi: 'तत्काल डिस्पैच', pa: 'ਤੁਰੰਤ ਡਿਸਪੈਚ' },
            'SCHEDULE_WITHIN_24H': { en: 'SCHEDULE WITHIN 24H', hi: '24 घंटे में निर्धारित', pa: '24 ਘੰਟੇ ਵਿੱਚ ਨਿਰਧਾਰਤ' },
            'ADD_TO_ROUTE': { en: 'ADD TO ROUTE', hi: 'रूट में जोड़ें', pa: 'ਰूट ਵਿੱਚ ਸ਼ਾਮਲ ਕਰੋ' }
        };

        // DPI Translations
        const translations = {
            en: {
                appTitle: "Smart Triage",
                appSubtitle: "Snap a photo. AI handles the rest.",
                lblPhoto: "Provide Photo Evidence",
                btnCamera: "Camera",
                btnGallery: "Gallery",
                lblPinpoint: "Confirm Location",
                btnLocate: "Locate Me",
                btnAnalyzeText: "Analyze & Report Issue",
                btnReset: "Report Another Issue",
                lblTicketTitle: "Citizen Tracking Ticket",
                lblScreenshot: "Take a screenshot to track your report status",
                lblAiResults: "AI Analysis Results",
                lblDetectedIssue: "Detected Issue:",
                lblSeverityScore: "Severity Score:",
                lblMuniRouting: "Municipal Routing",
                lblAssignedSector: "Assigned Sector:",
                lblCalcPriority: "Calculated Priority:",
                lblDispatchAction: "Dispatch Action:"
            },
            hi: {
                appTitle: "स्मार्ट ट्राइएज",
                appSubtitle: "एक फोटो लें। एआई बाकी काम करेगा।",
                lblPhoto: "फोटो प्रमाण प्रदान करें",
                btnCamera: "कैमरा",
                btnGallery: "गैलरी",
                lblPinpoint: "स्थान की पुष्टि करें",
                btnLocate: "मुझे खोजें",
                btnAnalyzeText: "विश्लेषण करें और रिपोर्ट करें",
                btnReset: "एक और समस्या की रिपोर्ट करें",
                lblTicketTitle: "नागरिक ट्रैकिंग टिकट",
                lblScreenshot: "स्थिति ट्रैक करने के लिए स्क्रीनशॉट लें",
                lblAiResults: "एआई विश्लेषण परिणाम",
                lblDetectedIssue: "पाई गई समस्या:",
                lblSeverityScore: "गंभीरता स्कोर:",
                lblMuniRouting: "नगर निगम रूटिंग",
                lblAssignedSector: "आवंटित सेक्टर:",
                lblCalcPriority: "अनुमानित प्राथमिकता:",
                lblDispatchAction: "डिस्पैच कार्रवाई:"
            },
            pa: {
                appTitle: "ਸਮਾਰਟ ਟ੍ਰਾਈਏਜ",
                appSubtitle: "ਇੱਕ ਫੋਟੋ ਲਓ। AI ਬਾਕੀ ਕੰਮ ਕਰੇਗਾ।",
                lblPhoto: "ਫੋਟੋ ਸਬੂਤ ਪ੍ਰਦਾਨ ਕਰੋ",
                btnCamera: "ਕੈਮਰਾ",
                btnGallery: "ਗੈਲਰੀ",
                lblPinpoint: "ਸਥਾਨ ਦੀ ਪੁਸ਼ਟੀ ਕਰੋ",
                btnLocate: "ਮੈਨੂੰ ਲੱਭੋ",
                btnAnalyzeText: "ਵਿਸ਼ਲੇਸ਼ਣ ਕਰੋ ਅਤੇ ਰਿਪੋਰਟ ਕਰੋ",
                btnReset: "ਇੱਕ ਹੋਰ ਸਮੱਸਿਆ ਦੀ ਰਿਪੋਰਟ ਕਰੋ",
                lblTicketTitle: "ਨਾਗਰਿਕ ਟ੍ਰੈਕਿੰਗ ਟਿਕਟ",
                lblScreenshot: "ਸਥਿਤੀ ਨੂੰ ਟਰੈਕ ਕਰਨ ਲਈ ਸਕ੍ਰੀਨਸ਼ੌਟ ਲਓ",
                lblAiResults: "AI ਵਿਸ਼ਲੇਸ਼ਣ ਨਤੀਜੇ",
                lblDetectedIssue: "ਲੱਭੀ ਗਈ ਸਮੱਸਿਆ:",
                lblSeverityScore: "ਗੰਭੀਰਤਾ ਸਕੋਰ:",
                lblMuniRouting: "ਮਿਉਂਸਪਲ ਰੂਟਿੰਗ",
                lblAssignedSector: "ਨਿਰਧਾਰਤ ਸੈਕਟਰ:",
                lblCalcPriority: "ਅਨੁਮਾਨਿਤ ਤਰਜੀਹ:",
                lblDispatchAction: "ਡਿਸਪੈਚ ਕਾਰਵਾਈ:"
            }
        };

        function changeLanguage(lang) {
            const t = translations[lang];
            document.getElementById('appTitle').textContent = t.appTitle;
            document.getElementById('appSubtitle').textContent = t.appSubtitle;
            document.getElementById('lblPhoto').textContent = t.lblPhoto;
            document.getElementById('btnCamera').textContent = t.btnCamera;
            document.getElementById('btnGallery').textContent = t.btnGallery;
            document.getElementById('lblPinpoint').textContent = t.lblPinpoint;
            document.getElementById('btnLocate').textContent = t.btnLocate;
            document.getElementById('resetBtn').textContent = t.btnReset;
            
            // Results Section Translations
            document.getElementById('lblTicketTitle').textContent = t.lblTicketTitle;
            document.getElementById('lblScreenshot').textContent = t.lblScreenshot;
            document.getElementById('lblAiResults').textContent = t.lblAiResults;
            document.getElementById('lblDetectedIssue').textContent = t.lblDetectedIssue;
            document.getElementById('lblSeverityScore').textContent = t.lblSeverityScore;
            document.getElementById('lblMuniRouting').textContent = t.lblMuniRouting;
            document.getElementById('lblAssignedSector').textContent = t.lblAssignedSector;
            document.getElementById('lblCalcPriority').textContent = t.lblCalcPriority;
            document.getElementById('lblDispatchAction').textContent = t.lblDispatchAction;
            
            // Dynamic Translations for AI Output
            if (currentIssueRaw) {
                let cleanIssue = currentIssueRaw.toLowerCase();
                document.getElementById('resAiType').textContent = termDict[cleanIssue] ? termDict[cleanIssue][lang] : currentIssueRaw.replace(/_/g, ' ');
            }
            if (currentActionRaw) {
                document.getElementById('resAction').textContent = termDict[currentActionRaw] ? termDict[currentActionRaw][lang] : currentActionRaw.replace(/_/g, ' ');
            }

            const analyzeBtn = document.getElementById('analyzeBtn');
            const btnSpan = analyzeBtn.querySelector('span');
            if(!analyzeBtn.disabled) btnSpan.textContent = t.btnAnalyzeText;
        }

        const citizenApp = document.getElementById('citizenApp');
        const adminLogin = document.getElementById('adminLogin');
        const adminDashboard = document.getElementById('adminDashboard');

        function showView(view) {
            citizenApp.classList.add('hidden'); adminLogin.classList.add('hidden'); adminDashboard.classList.add('hidden');
            document.body.classList.remove('body-admin-mode');

            if(view === 'citizen') { citizenApp.classList.remove('hidden'); if(map) setTimeout(() => map.invalidateSize(), 100); }
            if(view === 'login') { adminLogin.classList.remove('hidden'); document.body.classList.add('body-admin-mode'); }
            if(view === 'dashboard') { adminDashboard.classList.remove('hidden'); document.body.classList.add('body-admin-mode'); renderDashboard(); }
        }

        document.getElementById('goToAdminBtn').addEventListener('click', () => showView('login'));
        document.getElementById('backToCitizenBtn').addEventListener('click', () => showView('citizen'));
        document.getElementById('logoutBtn').addEventListener('click', () => showView('login'));

        // ==========================================
        // ADMIN LOGIC
        // ==========================================
        let adminMapInitialized = false;
        let adminMap = null;

        function switchAdminTab(tab) {
            const viewTickets = document.getElementById('viewTickets');
            const viewHeatmap = document.getElementById('viewHeatmap');
            const tabTickets = document.getElementById('tabTickets');
            const tabHeatmap = document.getElementById('tabHeatmap');

            if (tab === 'tickets') {
                viewTickets.classList.remove('hidden'); viewHeatmap.classList.add('hidden');
                tabTickets.className = "whitespace-nowrap px-6 py-3 font-bold text-sm border-b-2 border-blue-600 text-blue-600";
                tabHeatmap.className = "whitespace-nowrap px-6 py-3 font-bold text-sm border-b-2 border-transparent text-slate-500 hover:text-slate-700 flex gap-2 items-center";
            } else {
                viewTickets.classList.add('hidden'); viewHeatmap.classList.remove('hidden');
                tabHeatmap.className = "whitespace-nowrap px-6 py-3 font-bold text-sm border-b-2 border-blue-600 text-blue-600 flex gap-2 items-center";
                tabTickets.className = "whitespace-nowrap px-6 py-3 font-bold text-sm border-b-2 border-transparent text-slate-500 hover:text-slate-700";
                
                if(!adminMapInitialized) setTimeout(initAdminMap, 50);
                else if(adminMap) setTimeout(() => adminMap.invalidateSize(), 100);
            }
        }

        function initAdminMap() {
            adminMapInitialized = true;
            adminMap = L.map('adminHeatmap').setView([30.7333, 76.7794], 13);
            
            // ESRI Dark Gray Canvas (Free, No API Key Watermarks)
            L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Dark_Gray_Base/MapServer/tile/{z}/{y}/{x}', {
                attribution: 'Tiles &copy; Esri',
                maxZoom: 16
            }).addTo(adminMap);

            const heatPoints = [
                [30.7333, 76.7794, 0.8], [30.7350, 76.7810, 0.9], [30.7320, 76.7750, 0.6],
                [30.7200, 76.7600, 0.9], [30.7180, 76.7550, 0.8], [30.7250, 76.7650, 0.7],
                [30.7450, 76.8000, 0.5], [30.7480, 76.8050, 0.4], [30.7100, 76.7400, 0.7], [30.7120, 76.7450, 0.8],
            ];

            L.heatLayer(heatPoints, { radius: 35, blur: 25, gradient: {0.4: 'blue', 0.6: 'lime', 0.8: 'yellow', 1.0: 'red'} }).addTo(adminMap);
        }

        document.getElementById('adminLoginForm').addEventListener('submit', (e) => {
            e.preventDefault();
            const user = document.getElementById('adminUser').value; const pass = document.getElementById('adminPass').value;
            const alertBox = document.getElementById('loginAlert');

            if (user === 'username' && pass === 'password') {
                alertBox.classList.add('hidden'); showView('dashboard');
                document.getElementById('adminUser').value = ''; document.getElementById('adminPass').value = '';
            } else {
                alertBox.textContent = 'Invalid credentials. Use username / password'; alertBox.classList.remove('hidden');
            }
        });

        function updateTicketStatus(id, newStatus) {
            const ticket = globalTickets.find(t => t.id === id);
            if(ticket) { ticket.status = newStatus; renderDashboard(); }
        }

        function contactCitizen(id) {
            const adminAlert = document.getElementById('adminAlertBox');
            adminAlert.textContent = `Opening SMS channel for Ticket ${id}. Automated update sent to citizen!`;
            adminAlert.classList.remove('hidden');
            setTimeout(() => adminAlert.classList.add('hidden'), 4000);
        }

        function renderDashboard() {
            const tbody = document.getElementById('ticketsTableBody'); tbody.innerHTML = '';
            let activeCount = 0, criticalCount = 0, resolvedCount = 0;

            globalTickets.forEach(ticket => {
                if (ticket.status !== 'resolved') activeCount++;
                if (ticket.priority >= 75 && ticket.status !== 'resolved') criticalCount++;
                if (ticket.status === 'resolved') resolvedCount++;

                const tr = document.createElement('tr'); tr.className = 'hover:bg-slate-50 transition-colors';
                let statusClass = 'status-new'; let statusText = 'New';
                if (ticket.status === 'in-progress') { statusClass = 'status-progress'; statusText = 'In Progress'; }
                if (ticket.status === 'resolved') { statusClass = 'status-resolved'; statusText = 'Resolved'; }

                tr.innerHTML = `
                    <td class="p-4 font-mono text-sm font-bold text-slate-700">${ticket.id}</td>
                    <td class="p-4 text-sm font-medium text-slate-800 capitalize">${ticket.type.replace(/_/g, ' ')}</td>
                    <td class="p-4 text-sm text-slate-500 flex flex-col"><span class="font-bold text-slate-700">${ticket.sector}</span><span class="text-xs">${ticket.date}</span></td>
                    <td class="p-4">
                        <div class="flex items-center gap-2">
                            <div class="w-12 h-1.5 bg-slate-200 rounded-full overflow-hidden">
                                <div class="h-full ${ticket.priority >= 75 ? 'bg-red-500' : 'bg-amber-500'}" style="width: ${ticket.priority}%"></div>
                            </div>
                            <span class="text-xs font-bold text-slate-600">${ticket.priority.toFixed(0)}</span>
                        </div>
                    </td>
                    <td class="p-4">
                        <select onchange="updateTicketStatus('${ticket.id}', this.value)" class="text-xs font-bold px-2 py-1 rounded-md border-0 ring-1 ring-inset ring-slate-200 focus:ring-2 focus:ring-slate-400 ${statusClass} outline-none cursor-pointer">
                            <option value="new" ${ticket.status === 'new' ? 'selected' : ''}>New</option>
                            <option value="in-progress" ${ticket.status === 'in-progress' ? 'selected' : ''}>In Progress</option>
                            <option value="resolved" ${ticket.status === 'resolved' ? 'selected' : ''}>Resolved</option>
                        </select>
                    </td>
                    <td class="p-4">
                        <button onclick="contactCitizen('${ticket.id}')" class="text-xs bg-blue-50 text-blue-600 font-bold px-3 py-1.5 rounded-lg hover:bg-blue-100 transition-colors flex items-center gap-1">
                            <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"></path></svg>
                            Contact
                        </button>
                    </td>
                `;
                tbody.appendChild(tr);
            });

            document.getElementById('statTotal').textContent = activeCount;
            document.getElementById('statCritical').textContent = criticalCount;
            document.getElementById('statResolved').textContent = resolvedCount;
        }

        // ==========================================
        // CITIZEN LOGIC 
        // ==========================================
        const cameraInput = document.getElementById('cameraInput');
        const galleryInput = document.getElementById('galleryInput');
        const previewArea = document.getElementById('previewArea');
        const imagePreview = document.getElementById('imagePreview');
        const analyzeBtn = document.getElementById('analyzeBtn');
        const spinner = document.getElementById('spinner');
        const formSection = document.getElementById('formSection');
        const resultsSection = document.getElementById('resultsSection');
        
        let currentBase64Image = null; let mimeType = null;
        let map = null; let currentLat = 30.7333; let currentLng = 76.7794;
        let userLat = null; let userLng = null; let resolvedSectorName = "Sector 17";

        function initMap(lat, lng) {
            if (map) { map.setView([lat, lng], 16); } 
            else {
                map = L.map('map', { center: [lat, lng], zoom: 16, zoomControl: false });
                L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { attribution: '© OSM' }).addTo(map);

                // Chandigarh Geo-Fencing
                const chandigarhBounds = L.latLngBounds(L.latLng(30.65, 76.65), L.latLng(30.85, 76.85));
                map.setMaxBounds(chandigarhBounds);
                map.on('drag', () => map.panInsideBounds(chandigarhBounds, { animate: false }));

                map.on('movestart', () => document.getElementById('mapWrapper').classList.add('is-dragging'));
                map.on('moveend', () => {
                    document.getElementById('mapWrapper').classList.remove('is-dragging');
                    const center = map.getCenter();
                    currentLat = center.lat; currentLng = center.lng;
                    fetchAddress(currentLat, currentLng);
                });
            }
            setTimeout(() => map.invalidateSize(), 100); 
            fetchAddress(lat, lng);
        }

        async function fetchAddress(lat, lng) {
            document.getElementById('sectorText').textContent = "Locating...";
            document.getElementById('addressText').textContent = "Translating coordinates...";
            try {
                const res = await fetch(`https://nominatim.openstreetmap.org/reverse?format=json&lat=${lat}&lon=${lng}&zoom=18&addressdetails=1`);
                const data = await res.json();
                if(data.address) {
                    resolvedSectorName = data.address.suburb || data.address.neighbourhood || data.address.residential || "Chandigarh";
                    document.getElementById('sectorText').textContent = resolvedSectorName;
                    document.getElementById('addressText').textContent = `${data.address.road || 'Unknown Street'}, ${data.address.state || 'Chandigarh'}`;
                }
            } catch (e) {
                document.getElementById('sectorText').textContent = "Location Pinned";
            }
        }

        document.getElementById('recenterBtn').addEventListener('click', () => {
            if (userLat && userLng) map.setView([userLat, userLng], 16, { animate: true });
        });

        function handleFileSelect(event) {
            const file = event.target.files[0]; if (!file) return;
            mimeType = file.type;
            const reader = new FileReader();
            reader.onload = function(e) {
                imagePreview.src = e.target.result;
                previewArea.classList.remove('hidden'); document.getElementById('mapSection').classList.remove('hidden');
                currentBase64Image = e.target.result.split(',')[1];
                
                if (navigator.geolocation) {
                    navigator.geolocation.getCurrentPosition(
                        pos => { userLat = pos.coords.latitude; userLng = pos.coords.longitude; initMap(userLat, userLng); },
                        err => initMap(currentLat, currentLng), { enableHighAccuracy: true, timeout: 5000 }
                    );
                } else initMap(currentLat, currentLng);
            };
            reader.readAsDataURL(file);
        }

        cameraInput.addEventListener('change', handleFileSelect); galleryInput.addEventListener('change', handleFileSelect);

        document.getElementById('removeBtn').addEventListener('click', () => {
            currentBase64Image = null; imagePreview.src = '';
            previewArea.classList.add('hidden'); document.getElementById('mapSection').classList.add('hidden');
            cameraInput.value = ''; galleryInput.value = '';
        });

        async function processTriage() {
            try {
                if(!currentBase64Image) throw new Error("No image provided");
                
                // IMPORTANT: In the Immersive Chat, leaving GEMINI_API_KEY empty auto-injects a real key!
                // If you host this elsewhere, you MUST define GEMINI_API_KEY at the top of the script.
                const apiKeyToUse = GEMINI_API_KEY || "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKeyToUse}`;
                
                const payload = {
                    contents: [{
                        role: "user",
                        parts: [
                            { text: "You are an expert civil engineer and municipal triage AI for Chandigarh. Look at this image and identify the primary infrastructure or civic issue. Return JSON containing 'issue_type' (must be EXACTLY one of these strings: 'pothole', 'garbage', 'overflowing_garbage', 'waterlogging', 'broken_street_light', 'dust_accumulation', 'damaged_road', 'construction_road_defects', 'ruttings', 'raveling', 'alligator_cracking', 'drainage_related', 'traffic_related') and 'severity' (a float from 1.0 to 10.0 based on danger to public and infrastructure)." },
                            { inlineData: { mimeType: mimeType || "image/jpeg", data: currentBase64Image } }
                        ]
                    }],
                    generationConfig: {
                        responseMimeType: "application/json",
                        responseSchema: {
                            type: "OBJECT",
                            properties: {
                                "issue_type": { type: "STRING" },
                                "severity": { type: "NUMBER" }
                            },
                            required: ["issue_type", "severity"]
                        }
                    }
                };

                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) {
                    const errData = await response.json();
                    console.error("API Error Details:", errData);
                    throw new Error(`API failed: ${response.status} - ${errData.error?.message || 'Unknown error'}`);
                }

                const result = await response.json();
                if (result.candidates && result.candidates[0].content.parts[0].text) {
                    const jsonRes = JSON.parse(result.candidates[0].content.parts[0].text);
                    const aiConfidence = jsonRes.severity / 10; 
                    
                    let priority = Math.min((jsonRes.severity * 10), 100.0);
                    let action = priority >= 75 ? "IMMEDIATE_DISPATCH" : (priority >= 40 ? "SCHEDULE_WITHIN_24H" : "ADD_TO_ROUTE");

                    const alertBox = document.getElementById('alertBox');
                    alertBox.textContent = "AI Analysis Complete: Verified by Gemini Vision (Civil Engineering Mode).";
                    alertBox.className = "mb-4 p-4 rounded-xl text-sm font-medium bg-green-50 text-green-700 border border-green-200 block";
                    setTimeout(() => alertBox.classList.add('hidden'), 4000);

                    return { 
                        issue_type: jsonRes.issue_type, 
                        ai_confidence: aiConfidence, 
                        assigned_sector: resolvedSectorName, 
                        dispatch_priority: priority, 
                        action: action 
                    };
                }
            } catch (err) {
                console.warn("Using Fallback Simulation because API key is missing or invalid:", err);
                
                const alertBox = document.getElementById('alertBox');
                alertBox.textContent = "Demo Mode Active: Please add your Gemini API key for real AI analysis.";
                alertBox.className = "mb-4 p-4 rounded-xl text-sm font-medium bg-yellow-50 text-yellow-700 border border-yellow-200 block";
                setTimeout(() => alertBox.classList.add('hidden'), 4000);

                // MOCK SIMULATION FALLBACK
                await new Promise(resolve => setTimeout(resolve, 1500));
                const issues = ["pothole", "overflowing_garbage", "broken_street_light", "waterlogging", "alligator_cracking", "damaged_road"];
                const randomIssue = issues[Math.floor(Math.random() * issues.length)];
                const aiConfidence = (Math.random() * (0.98 - 0.75) + 0.75); 
                
                let base_weight = 1.0;
                if (randomIssue === "waterlogging") base_weight = 2.5; else if (randomIssue === "pothole") base_weight = 2.0; 
                
                let priority = Math.min((aiConfidence * 10.0) * Math.pow(base_weight, 1.2), 100.0);
                let action = priority >= 75 ? "IMMEDIATE_DISPATCH" : (priority >= 40 ? "SCHEDULE_WITHIN_24H" : "ADD_TO_ROUTE");

                return { issue_type: randomIssue, ai_confidence: aiConfidence, assigned_sector: resolvedSectorName, dispatch_priority: priority, action: action };
            }
        }

        analyzeBtn.addEventListener('click', async () => {
            if (!currentBase64Image) return;
            analyzeBtn.disabled = true; analyzeBtn.querySelector('span').textContent = "AI Analyzing..."; spinner.classList.remove('hidden');
            
            try {
                const data = await processTriage();
                const randomId = `#CHD-2026-${Math.floor(1000 + Math.random() * 9000)}`;
                
                globalTickets.unshift({ id: randomId, type: data.issue_type, sector: data.assigned_sector, priority: data.dispatch_priority, status: 'new', date: 'Just now' });

                formSection.classList.add('hidden'); resultsSection.classList.remove('hidden');
                
                currentIssueRaw = data.issue_type;
                currentActionRaw = data.action;
                const currentLang = document.getElementById('langSelector').value;

                let cleanIssue = currentIssueRaw.toLowerCase();
                document.getElementById('resAiType').textContent = termDict[cleanIssue] ? termDict[cleanIssue][currentLang] : data.issue_type.replace(/_/g, ' ');
                document.getElementById('resAiSeverity').textContent = `${(data.ai_confidence * 10).toFixed(1)} / 10`;
                document.getElementById('resAiSeverityBar').style.width = `${data.ai_confidence * 100}%`;
                document.getElementById('resSector').textContent = data.assigned_sector;
                document.getElementById('resPriority').textContent = data.dispatch_priority.toFixed(1);
                document.getElementById('resAction').textContent = termDict[currentActionRaw] ? termDict[currentActionRaw][currentLang] : data.action.replace(/_/g, ' ');
                document.getElementById('resTicketId').textContent = randomId;
            } finally {
                analyzeBtn.disabled = false; spinner.classList.add('hidden');
            }
        });

        document.getElementById('resetBtn').addEventListener('click', () => {
            document.getElementById('removeBtn').click();
            resultsSection.classList.add('hidden'); formSection.classList.remove('hidden');
        });

    </script>
</body>
</html>
