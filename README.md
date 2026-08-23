<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ระบบรายงานผลการแข่งขันสด - Live Scoreboard</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts (Kanit & Inter) -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        kanit: ['Kanit', 'sans-serif'],
                    },
                    colors: {
                        brand: {
                            50: '#f0f7ff',
                            100: '#e0effe',
                            500: '#2563eb',
                            600: '#1d4ed8',
                            700: '#1e40af',
                            900: '#0f172a'
                        }
                    }
                }
            }
        }
    </script>
    <style>
        body {
            font-family: 'Kanit', sans-serif;
            background-color: #0f172a;
            color: #f8fafc;
        }
        .pulse-live {
            animation: pulse 1.5s cubic-bezier(0.4, 0, 0.6, 1) infinite;
        }
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.3; }
        }
        .glass-panel {
            background: rgba(30, 41, 59, 0.7);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.08);
        }
    </style>
</head>
<body class="min-h-screen pb-12">

    <!-- Header / Navbar -->
    <header class="sticky top-0 z-40 glass-panel border-b border-slate-700/50 shadow-lg">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-3">
                <div class="w-10 h-10 rounded-xl bg-gradient-to-tr from-blue-600 to-indigo-500 flex items-center justify-center shadow-md shadow-blue-500/20">
                    <i class="fa-solid fa-trophy text-amber-300 text-lg"></i>
                </div>
                <div>
                    <h1 class="text-xl font-bold bg-gradient-to-r from-white via-slate-200 to-slate-400 bg-clip-text text-transparent">
                        Live Score Center
                    </h1>
                    <p class="text-xs text-slate-400 flex items-center gap-1.5">
                        <span class="inline-block w-2 h-2 rounded-full bg-emerald-500 pulse-live"></span>
                        ระบบรายงานผลสดแบบ Realtime
                    </p>
                </div>
            </div>

            <div class="flex items-center gap-3">
                <!-- Sync Indicator -->
                <div id="syncStatus" class="flex items-center gap-2 text-xs bg-slate-800/80 px-3 py-1.5 rounded-full border border-slate-700 text-emerald-400 shadow-sm">
                    <span class="relative flex h-2 w-2">
                      <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-emerald-400 opacity-75"></span>
                      <span class="relative inline-flex rounded-full h-2 w-2 bg-emerald-500"></span>
                    </span>
                    <span id="syncStatusText">กำลังเชื่อมต่อระบบ Realtime...</span>
                </div>

                <!-- Admin Button -->
                <button id="adminAuthBtn" onclick="toggleAdminModal()" class="px-3 py-1.5 rounded-lg text-xs font-medium bg-slate-800 hover:bg-slate-700 border border-slate-700 text-slate-300 transition-all flex items-center gap-2">
                    <i class="fa-solid fa-lock text-amber-400"></i>
                    <span id="adminBtnText">เข้าสู่ระบบ Admin</span>
                </button>
            </div>
        </div>
    </header>

    <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 mt-6">
        
        <!-- Status Banner for Admin Mode -->
        <div id="adminBadgeBanner" class="hidden mb-6 p-4 rounded-xl bg-amber-500/10 border border-amber-500/30 flex flex-wrap justify-between items-center gap-3">
            <div class="flex items-center gap-3 text-amber-300">
                <i class="fa-solid fa-user-shield text-xl"></i>
                <div>
                    <span class="font-semibold block">โหมดผู้ดูแลระบบ (Admin Mode) ปลดล็อกแล้ว</span>
                    <span class="text-xs text-amber-400/80">คุณสามารถกรอก/แก้ไขผลการแข่งขัน เพิ่มตารางแข่ง และปรับปรุงคะแนนได้ทันที</span>
                </div>
            </div>
            <button onclick="logoutAdmin()" class="px-3 py-1 bg-amber-500/20 hover:bg-amber-500/30 text-amber-200 rounded-lg text-xs transition border border-amber-500/40">
                <i class="fa-solid fa-right-from-bracket mr-1"></i> ออกจากระบบผู้ดูแล
            </button>
        </div>

        <!-- Navigation Tabs -->
        <div class="flex border-b border-slate-800 mb-6 gap-2 sm:gap-6 overflow-x-auto">
            <button id="tabMatchesBtn" onclick="switchTab('matches')" class="py-3 px-4 text-sm font-semibold border-b-2 border-blue-500 text-blue-400 flex items-center gap-2 whitespace-nowrap">
                <i class="fa-solid fa-gamepad"></i>
                ตารางการแข่งขัน & คะแนนสด
            </button>
            <button id="tabStandingsBtn" onclick="switchTab('standings')" class="py-3 px-4 text-sm font-semibold border-b-2 border-transparent text-slate-400 hover:text-slate-200 flex items-center gap-2 whitespace-nowrap">
                <i class="fa-solid fa-chart-line"></i>
                คะแนนรวม & โอกาสเหรียญรางวัล
            </button>
        </div>

        <!-- TAB 1: MATCHES VIEW -->
        <section id="tabMatches" class="space-y-6">
            <!-- Filter & Action Header -->
            <div class="flex flex-wrap justify-between items-center gap-4">
                <div class="flex items-center gap-2 overflow-x-auto pb-1 max-w-full">
                    <span class="px-3.5 py-1.5 rounded-lg text-xs font-semibold bg-blue-600/20 text-blue-400 border border-blue-500/30 flex items-center gap-2">
                        <i class="fa-solid fa-layer-group"></i>
                        ทุกประเภทกีฬา
                    </span>
                </div>

                <!-- Admin Add Match Button -->
                <div id="adminAddMatchContainer" class="hidden">
                    <button onclick="openMatchModal()" class="px-4 py-2 bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-500 hover:to-teal-500 text-white rounded-lg text-xs font-medium shadow-lg shadow-emerald-900/30 flex items-center gap-2 transition">
                        <i class="fa-solid fa-plus"></i>
                        เพิ่มแมตช์การแข่งขัน
                    </button>
                </div>
            </div>

            <!-- Matches List Container -->
            <div id="matchesList" class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <!-- Dynamic Content Loaded via Firestore JS -->
                <div class="col-span-full text-center py-12 text-slate-500">
                    <i class="fa-solid fa-circle-notch fa-spin text-2xl text-blue-500 mb-2"></i>
                    <p>กำลังโหลดข้อมูลผลการแข่งขันเรียลไทม์...</p>
                </div>
            </div>
        </section>

        <!-- TAB 2: STANDINGS & MEDALS -->
        <section id="tabStandings" class="hidden space-y-6">
            <div class="glass-panel p-6 rounded-2xl border border-slate-800 shadow-xl">
                <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4 mb-6">
                    <div>
                        <h2 class="text-lg font-bold text-white flex items-center gap-2">
                            <i class="fa-solid fa-award text-amber-400"></i>
                            ตารางสรุปคะแนนรวม & คาดการณ์เหรียญรางวัล
                        </h2>
                        <p class="text-xs text-slate-400 mt-1">คำนวณคะแนนรวมสะสมทุกชนิดกีฬาและวิเคราะห์โอกาสได้รับเหรียญ (คำนวณอัตโนมัติ)</p>
                    </div>

                    <!-- Admin Standings Action -->
                    <div id="adminStandingsControls" class="hidden flex gap-2">
                        <button onclick="recalculateStandings()" class="px-3 py-1.5 bg-blue-600 hover:bg-blue-500 text-white rounded-lg text-xs font-medium transition flex items-center gap-1.5">
                            <i class="fa-solid fa-calculator"></i> คำนวณคะแนนใหม่
                        </button>
                        <button onclick="openAddTeamModal()" class="px-3 py-1.5 bg-slate-700 hover:bg-slate-600 text-white rounded-lg text-xs font-medium transition flex items-center gap-1.5">
                            <i class="fa-solid fa-user-plus"></i> เพิ่มทีม
                        </button>
                    </div>
                </div>

                <!-- Standings Table -->
                <div class="overflow-x-auto">
                    <table class="w-full text-left text-sm">
                        <thead class="bg-slate-900/80 text-slate-400 uppercase text-xs border-b border-slate-800">
                            <tr>
                                <th class="py-3 px-4 font-semibold text-center">อันดับ</th>
                                <th class="py-3 px-4 font-semibold">ทีม / สโมสร</th>
                                <th class="py-3 px-4 font-semibold text-center">แข่ง</th>
                                <th class="py-3 px-4 font-semibold text-center">ชนะ</th>
                                <th class="py-3 px-4 font-semibold text-center">แพ้</th>
                                <th class="py-3 px-4 font-semibold text-center">คะแนนรวม</th>
                                <th class="py-3 px-4 font-semibold text-center">ความน่าจะเป็นเหรียญรางวัล</th>
                                <th id="adminTableCol" class="hidden py-3 px-4 font-semibold text-center">จัดการ</th>
                            </tr>
                        </thead>
                        <tbody id="standingsTableBody" class="divide-y divide-slate-800/60">
                            <!-- Dynamic Content -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

    </main>

    <!-- MODAL 1: Admin Password Login -->
    <div id="adminPasswordModal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="glass-panel max-w-md w-full p-6 rounded-2xl border border-slate-700 shadow-2xl relative">
            <button onclick="closeAdminModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white">
                <i class="fa-solid fa-xmark text-lg"></i>
            </button>
            <div class="text-center mb-6">
                <div class="w-12 h-12 rounded-full bg-amber-500/10 border border-amber-500/30 text-amber-400 mx-auto flex items-center justify-center text-xl mb-3">
                    <i class="fa-solid fa-key"></i>
                </div>
                <h3 class="text-lg font-bold text-white">ยืนยันรหัสผ่าน Admin</h3>
                <p class="text-xs text-slate-400 mt-1">กรอกรหัสผ่านเพื่อเข้าสู่โหมดแก้ไขข้อมูลและผลคะแนน</p>
            </div>
            
            <form onsubmit="handleAdminLogin(event)" class="space-y-4">
                <div>
                    <label class="block text-xs font-medium text-slate-300 mb-1">รหัสผ่าน (Admin Passcode)</label>
                    <input type="password" id="adminPasswordInput" required placeholder="••••••••••••••••" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-4 py-2.5 text-sm text-white focus:outline-none focus:border-amber-500 transition">
                </div>
                <div id="adminAuthError" class="hidden text-xs text-rose-400 text-center bg-rose-500/10 py-1.5 rounded-lg border border-rose-500/20">
                    รหัสผ่านไม่ถูกต้อง กรุณาลองใหม่อีกครั้ง
                </div>
                <button type="submit" class="w-full py-2.5 bg-amber-500 hover:bg-amber-400 text-slate-950 font-bold rounded-xl text-sm transition">
                    เข้าสู่ระบบ
                </button>
            </form>
        </div>
    </div>

    <!-- MODAL 2: Add/Edit Match Modal -->
    <div id="matchModal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4 overflow-y-auto">
        <div class="glass-panel max-w-lg w-full p-6 rounded-2xl border border-slate-700 shadow-2xl relative my-8">
            <button onclick="closeMatchModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white">
                <i class="fa-solid fa-xmark text-lg"></i>
            </button>
            <h3 id="matchModalTitle" class="text-lg font-bold text-white mb-4">เพิ่มข้อมูลการแข่งขัน</h3>
            
            <form onsubmit="saveMatch(event)" class="space-y-4">
                <input type="hidden" id="matchFormId">
                
                <div>
                    <label class="block text-xs text-slate-300 mb-1">ประเภทกีฬา</label>
                    <input type="text" id="matchSport" required placeholder="เช่น ฟุตบอล, บาสเกตบอล" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-2 text-sm text-white">
                </div>

                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs text-slate-300 mb-1">สถานะ</label>
                        <select id="matchStatus" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-2 text-sm text-white">
                            <option value="UPCOMING">ยังไม่แข่งขัน</option>
                            <option value="LIVE">กำลังแข่งขัน (LIVE)</option>
                            <option value="FINISHED">จบการแข่งขัน</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs text-slate-300 mb-1">เวลาแข่งขัน</label>
                        <input type="text" id="matchTime" placeholder="เช่น วันนี้ 18:00" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-2 text-sm text-white">
                    </div>
                </div>

                <div class="p-3 bg-slate-900/50 rounded-xl border border-slate-800 space-y-3">
                    <span class="text-xs font-semibold text-blue-400 block">ทีมที่ 1 (เจ้าบ้าน/ฝั่งซ้าย)</span>
                    <div class="grid grid-cols-3 gap-2">
                        <div class="col-span-2">
                            <input type="text" id="team1Name" required placeholder="ชื่อทีม 1" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-xs text-white">
                        </div>
                        <div>
                            <input type="number" id="team1Score" value="0" min="0" placeholder="คะแนน" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-xs text-white text-center">
                        </div>
                    </div>
                    <div>
                        <input type="url" id="team1Logo" placeholder="URL รูปภาพ/โลโก้ทีม 1 (ไม่ใส่ใช้ภาพเริ่มต้น)" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-xs text-white">
                    </div>
                </div>

                <div class="p-3 bg-slate-900/50 rounded-xl border border-slate-800 space-y-3">
                    <span class="text-xs font-semibold text-rose-400 block">ทีมที่ 2 (ทีมเยือน/ฝั่งขวา)</span>
                    <div class="grid grid-cols-3 gap-2">
                        <div class="col-span-2">
                            <input type="text" id="team2Name" required placeholder="ชื่อทีม 2" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-xs text-white">
                        </div>
                        <div>
                            <input type="number" id="team2Score" value="0" min="0" placeholder="คะแนน" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-xs text-white text-center">
                        </div>
                    </div>
                    <div>
                        <input type="url" id="team2Logo" placeholder="URL รูปภาพ/โลโก้ทีม 2 (ไม่ใส่ใช้ภาพเริ่มต้น)" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-xs text-white">
                    </div>
                </div>

                <div class="flex justify-end gap-2 pt-2">
                    <button type="button" onclick="closeMatchModal()" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 text-slate-300 rounded-lg text-xs font-medium">ยกเลิก</button>
                    <button type="submit" class="px-4 py-2 bg-blue-600 hover:bg-blue-500 text-white rounded-lg text-xs font-medium">บันทึกข้อมูล</button>
                </div>
            </form>
        </div>
    </div>

    <!-- MODAL 3: Add/Edit Team Modal -->
    <div id="teamModal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="glass-panel max-w-md w-full p-6 rounded-2xl border border-slate-700 shadow-2xl relative">
            <button onclick="closeTeamModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white">
                <i class="fa-solid fa-xmark text-lg"></i>
            </button>
            <h3 class="text-lg font-bold text-white mb-4">จัดการทีมในตารางสรุปคะแนน</h3>
            
            <form onsubmit="saveTeamStandings(event)" class="space-y-3">
                <input type="hidden" id="teamFormId">
                <div>
                    <label class="block text-xs text-slate-300 mb-1">ชื่อทีม/หน่วยงาน</label>
                    <input type="text" id="teamStandingsName" required class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-2 text-sm text-white">
                </div>
                <div>
                    <label class="block text-xs text-slate-300 mb-1">URL โลโก้ทีม</label>
                    <input type="url" id="teamStandingsLogo" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-2 text-sm text-white">
                </div>
                <div class="grid grid-cols-3 gap-2">
                    <div>
                        <label class="block text-xs text-slate-300 mb-1">จำนวนนัดแข่ง</label>
                        <input type="number" id="teamPlayed" value="0" min="0" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-xs text-white">
                    </div>
                    <div>
                        <label class="block text-xs text-slate-300 mb-1">ชนะ</label>
                        <input type="number" id="teamWon" value="0" min="0" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-xs text-white">
                    </div>
                    <div>
                        <label class="block text-xs text-slate-300 mb-1">แพ้</label>
                        <input type="number" id="teamLost" value="0" min="0" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-xs text-white">
                    </div>
                </div>
                <div>
                    <label class="block text-xs text-slate-300 mb-1">คะแนนรวมสะสม</label>
                    <input type="number" id="teamTotalPoints" value="0" min="0" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-2 text-sm text-white">
                </div>
                <div class="flex justify-end gap-2 pt-2">
                    <button type="button" onclick="closeTeamModal()" class="px-4 py-2 bg-slate-800 text-slate-300 rounded-lg text-xs">ยกเลิก</button>
                    <button type="submit" class="px-4 py-2 bg-blue-600 text-white rounded-lg text-xs">บันทึก</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Notification Toast -->
    <div id="toast" class="fixed bottom-5 right-5 bg-slate-800 border border-slate-700 text-white px-4 py-3 rounded-xl shadow-xl hidden z-50 flex items-center gap-3">
        <i id="toastIcon" class="fa-solid fa-circle-check text-emerald-400"></i>
        <span id="toastMessage" class="text-sm">การดำเนินการสำเร็จ</span>
    </div>

    <!-- Firebase Integration Module -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, doc, setDoc, addDoc, updateDoc, deleteDoc, onSnapshot, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const appId = typeof __app_id !== 'undefined' ? __app_id : 'live-scoreboard-system';
        const firebaseConfig = typeof __firebase_config !== 'undefined' 
            ? JSON.parse(__firebase_config) 
            : {
                apiKey: "AIzaSyDemoKeyOnlyForFallback12345",
                authDomain: "demo-app.firebaseapp.com",
                projectId: "demo-app",
                storageBucket: "demo-app.appspot.com",
                messagingSenderId: "123456789",
                appId: "1:123456789:web:abcdef123456"
            };

        // Passcode provided by user
        const ADMIN_PASSCODE = "07Poyu_@841lowl[rirjfloe=10kunla2";

        // Global Application State
        window.appState = {
            isAdmin: false,
            matches: [],
            standings: [],
            db: null,
            auth: null,
            user: null
        };

        // Fallback team image helper
        const getFallbackImage = (name) => `https://placehold.co/100x100/1e293b/64748b?text=${encodeURIComponent(name || 'Team')}`;

        async function initFirebase() {
            try {
                const app = initializeApp(firebaseConfig);
                const auth = getAuth(app);
                const db = getFirestore(app);

                window.appState.db = db;
                window.appState.auth = auth;

                // Authenticate anonymously or with token
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }

                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        window.appState.user = user;
                        // Subscribe to Firestore collections after authentication
                        setupRealtimeListeners();
                    }
                });

            } catch (err) {
                console.error("Firebase init error:", err);
                showToast("ไม่สามารถเชื่อมต่อฐานข้อมูลเรียลไทม์ได้", "error");
            }
        }

        function setupRealtimeListeners() {
            const db = window.appState.db;
            if (!db) return;

            const syncText = document.getElementById('syncStatusText');

            // Strict Rules Paths
            const matchesRef = collection(db, 'artifacts', appId, 'public', 'data', 'matches');
            const standingsRef = collection(db, 'artifacts', appId, 'public', 'data', 'standings');

            // Listen to Matches Realtime Updates
            onSnapshot(matchesRef, (snapshot) => {
                const matches = [];
                snapshot.forEach((docSnap) => {
                    matches.push({ id: docSnap.id, ...docSnap.data() });
                });

                window.appState.matches = matches;
                renderMatches();
                if (syncText) syncText.textContent = "เชื่อมต่อสดเรียลไทม์ (อัปเดตพร้อมกันทุกจอ)";
            }, (error) => {
                console.error("Matches listener error:", error);
                if (syncText) syncText.textContent = "เกิดข้อผิดพลาดในการเชื่อมต่อ Realtime";
            });

            // Listen to Standings Realtime Updates
            onSnapshot(standingsRef, (snapshot) => {
                const standings = [];
                snapshot.forEach((docSnap) => {
                    standings.push({ id: docSnap.id, ...docSnap.data() });
                });

                window.appState.standings = standings;
                renderStandings();
            }, (error) => {
                console.error("Standings listener error:", error);
            });
        }

        window.renderMatches = function() {
            const container = document.getElementById('matchesList');
            if (!container) return;

            const filtered = window.appState.matches;

            if (filtered.length === 0) {
                container.innerHTML = `
                    <div class="col-span-full text-center py-12 glass-panel rounded-2xl border border-slate-800 text-slate-400">
                        <i class="fa-solid fa-trophy text-3xl mb-2 text-slate-600 block"></i>
                        <p class="font-medium text-slate-300">ยังไม่มีข้อมูลแมตช์การแข่งขัน</p>
                        <p class="text-xs text-slate-500 mt-1">เข้าสู่ระบบผู้ดูแลเพื่อเริ่มเพิ่มการแข่งขันและใส่คะแนนสด</p>
                    </div>
                `;
                return;
            }

            container.innerHTML = filtered.map(match => {
                const isLive = match.status === 'LIVE';
                const isFinished = match.status === 'FINISHED';
                const isAdmin = window.appState.isAdmin;

                const statusBadge = isLive 
                    ? `<span class="inline-flex items-center gap-1.5 px-2.5 py-1 rounded-full text-xs font-bold bg-emerald-500/10 text-emerald-400 border border-emerald-500/30">
                        <span class="w-2 h-2 rounded-full bg-emerald-400 pulse-live"></span> กำลังแข่ง
                       </span>`
                    : isFinished 
                    ? `<span class="px-2.5 py-1 rounded-full text-xs font-semibold bg-slate-800 text-slate-400 border border-slate-700">จบการแข่งขัน</span>`
                    : `<span class="px-2.5 py-1 rounded-full text-xs font-medium bg-blue-500/10 text-blue-400 border border-blue-500/20">${match.time || 'ยังไม่เริ่ม'}</span>`;

                return `
                    <div class="glass-panel rounded-2xl p-5 border border-slate-800 shadow-xl relative transition-all hover:border-slate-700">
                        <!-- Top Header -->
                        <div class="flex justify-between items-center mb-4 pb-3 border-b border-slate-800/80">
                            <span class="text-xs font-semibold text-slate-400 uppercase tracking-wider flex items-center gap-2">
                                <i class="fa-solid fa-dumbbell text-blue-500"></i>
                                ${escapeHtml(match.sport || 'แมตช์การแข่งขัน')}
                            </span>
                            <div class="flex items-center gap-2">
                                ${statusBadge}
                                ${isAdmin ? `
                                    <button onclick="openMatchModal('${match.id}')" class="p-1.5 hover:bg-slate-800 rounded-lg text-slate-400 hover:text-amber-400 text-xs transition" title="แก้ไข">
                                        <i class="fa-solid fa-pen-to-square"></i>
                                    </button>
                                    <button onclick="deleteMatch('${match.id}')" class="p-1.5 hover:bg-slate-800 rounded-lg text-slate-400 hover:text-rose-400 text-xs transition" title="ลบ">
                                        <i class="fa-solid fa-trash"></i>
                                    </button>
                                ` : ''}
                            </div>
                        </div>

                        <!-- Teams & Score Display -->
                        <div class="grid grid-cols-7 items-center gap-2 text-center py-2">
                            <!-- Team 1 -->
                            <div class="col-span-3 flex flex-col items-center">
                                <img src="${escapeHtml(match.team1?.logo || getFallbackImage(match.team1?.name))}" 
                                     onerror="this.src='${getFallbackImage(match.team1?.name)}'" 
                                     class="w-14 h-14 object-cover rounded-xl border border-slate-700/60 shadow-md mb-2 bg-slate-900">
                                <span class="font-bold text-sm text-white line-clamp-1">${escapeHtml(match.team1?.name || 'ทีม 1')}</span>
                            </div>

                            <!-- Score Center -->
                            <div class="col-span-1 flex flex-col items-center justify-center">
                                ${isAdmin ? `
                                    <!-- Quick Admin Score Inputs -->
                                    <div class="flex flex-col items-center gap-1">
                                        <input type="number" min="0" value="${match.team1?.score ?? 0}" 
                                               onchange="quickUpdateScore('${match.id}', 'team1', this.value)" 
                                               class="w-12 text-center bg-slate-900 border border-slate-700 rounded py-0.5 text-sm font-bold text-blue-400 focus:outline-none focus:border-amber-400">
                                        <span class="text-xs text-slate-500 font-bold">-</span>
                                        <input type="number" min="0" value="${match.team2?.score ?? 0}" 
                                               onchange="quickUpdateScore('${match.id}', 'team2', this.value)" 
                                               class="w-12 text-center bg-slate-900 border border-slate-700 rounded py-0.5 text-sm font-bold text-rose-400 focus:outline-none focus:border-amber-400">
                                    </div>
                                ` : `
                                    <div class="text-2xl font-black text-white tracking-widest flex items-center gap-1">
                                        <span class="${match.team1?.score > match.team2?.score ? 'text-blue-400' : ''}">${match.team1?.score ?? 0}</span>
                                        <span class="text-slate-600 text-sm">:</span>
                                        <span class="${match.team2?.score > match.team1?.score ? 'text-rose-400' : ''}">${match.team2?.score ?? 0}</span>
                                    </div>
                                `}
                            </div>

                            <!-- Team 2 -->
                            <div class="col-span-3 flex flex-col items-center">
                                <img src="${escapeHtml(match.team2?.logo || getFallbackImage(match.team2?.name))}" 
                                     onerror="this.src='${getFallbackImage(match.team2?.name)}'" 
                                     class="w-14 h-14 object-cover rounded-xl border border-slate-700/60 shadow-md mb-2 bg-slate-900">
                                <span class="font-bold text-sm text-white line-clamp-1">${escapeHtml(match.team2?.name || 'ทีม 2')}</span>
                            </div>
                        </div>

                        <!-- Footer Info -->
                        <div class="mt-3 pt-2 text-center border-t border-slate-800/40">
                            <span class="text-[11px] text-slate-500">
                                <i class="fa-regular fa-clock mr-1"></i> ${escapeHtml(match.time || 'ไม่ระบุเวลา')}
                            </span>
                        </div>
                    </div>
                `;
            }).join('');
        };

        window.renderStandings = function() {
            const tbody = document.getElementById('standingsTableBody');
            if (!tbody) return;

            const isAdmin = window.appState.isAdmin;
            document.getElementById('adminTableCol').classList.toggle('hidden', !isAdmin);

            // Sort team by points descending
            const standings = [...window.appState.standings].sort((a, b) => (b.points || 0) - (a.points || 0));

            if (standings.length === 0) {
                tbody.innerHTML = `
                    <tr>
                        <td colspan="${isAdmin ? 8 : 7}" class="py-12 text-center text-slate-500">
                            <i class="fa-solid fa-users-slash text-2xl mb-2 text-slate-600 block"></i>
                            <span class="font-medium text-slate-300 block mb-1">ยังไม่มีข้อมูลทีมในตารางคะแนน</span>
                            <span class="text-xs text-slate-500">ผู้ดูแลระบบสามารถเข้าสู่ระบบ Admin เพื่อกด "เพิ่มทีม" ได้ทันที</span>
                        </td>
                    </tr>
                `;
                return;
            }

            const maxPointsPossible = Math.max(...standings.map(s => s.points || 0), 1);

            tbody.innerHTML = standings.map((team, index) => {
                const rank = index + 1;
                let rankBadge = `<span class="text-slate-400 font-semibold">${rank}</span>`;
                
                if (rank === 1) rankBadge = `<div class="w-7 h-7 rounded-full bg-amber-500/20 text-amber-400 flex items-center justify-center font-bold text-xs mx-auto border border-amber-500/40">1</div>`;
                if (rank === 2) rankBadge = `<div class="w-7 h-7 rounded-full bg-slate-300/20 text-slate-200 flex items-center justify-center font-bold text-xs mx-auto border border-slate-400/40">2</div>`;
                if (rank === 3) rankBadge = `<div class="w-7 h-7 rounded-full bg-amber-700/20 text-amber-600 flex items-center justify-center font-bold text-xs mx-auto border border-amber-700/40">3</div>`;

                // Calculate Medal Probability Projection
                const points = team.points || 0;
                let medalProbHtml = '';

                if (rank === 1) {
                    medalProbHtml = `
                        <span class="inline-flex items-center gap-1.5 px-3 py-1 rounded-full text-xs font-bold bg-amber-500/10 text-amber-400 border border-amber-500/30">
                            <i class="fa-solid fa-medal text-amber-400"></i> ตัวเต็งเหรียญทอง (90%+)
                        </span>
                    `;
                } else if (rank === 2) {
                    medalProbHtml = `
                        <span class="inline-flex items-center gap-1.5 px-3 py-1 rounded-full text-xs font-bold bg-slate-300/10 text-slate-300 border border-slate-400/30">
                            <i class="fa-solid fa-medal text-slate-300"></i> ตัวเต็งเหรียญเงิน (75%+)
                        </span>
                    `;
                } else if (rank === 3) {
                    medalProbHtml = `
                        <span class="inline-flex items-center gap-1.5 px-3 py-1 rounded-full text-xs font-bold bg-amber-800/20 text-amber-500 border border-amber-700/30">
                            <i class="fa-solid fa-medal text-amber-600"></i> ตัวเต็งเหรียญทองแดง (60%+)
                        </span>
                    `;
                } else {
                    const chance = Math.max(10, Math.round((points / maxPointsPossible) * 50));
                    medalProbHtml = `
                        <span class="inline-flex items-center gap-1 px-2.5 py-1 rounded-full text-xs font-medium bg-slate-800 text-slate-400">
                            โอกาสเหรียญ ${chance}%
                        </span>
                    `;
                }

                return `
                    <tr class="hover:bg-slate-800/40 transition">
                        <td class="py-3 px-4 text-center">${rankBadge}</td>
                        <td class="py-3 px-4">
                            <div class="flex items-center gap-3">
                                <img src="${escapeHtml(team.logo || getFallbackImage(team.name))}" 
                                     onerror="this.src='${getFallbackImage(team.name)}'" 
                                     class="w-8 h-8 rounded-lg object-cover bg-slate-900 border border-slate-700">
                                <span class="font-bold text-white">${escapeHtml(team.name)}</span>
                            </div>
                        </td>
                        <td class="py-3 px-4 text-center text-slate-300">${team.played || 0}</td>
                        <td class="py-3 px-4 text-center text-emerald-400 font-medium">${team.won || 0}</td>
                        <td class="py-3 px-4 text-center text-rose-400 font-medium">${team.lost || 0}</td>
                        <td class="py-3 px-4 text-center font-black text-blue-400 text-base">${team.points || 0}</td>
                        <td class="py-3 px-4 text-center">${medalProbHtml}</td>
                        ${isAdmin ? `
                            <td class="py-3 px-4 text-center">
                                <button onclick="openAddTeamModal('${team.id}')" class="p-1.5 text-slate-400 hover:text-amber-400">
                                    <i class="fa-solid fa-pen"></i>
                                </button>
                                <button onclick="deleteTeamStandings('${team.id}')" class="p-1.5 text-slate-400 hover:text-rose-400">
                                    <i class="fa-solid fa-trash"></i>
                                </button>
                            </td>
                        ` : ''}
                    </tr>
                `;
            }).join('');
        };

        window.handleAdminLogin = function(e) {
            e.preventDefault();
            const passcode = document.getElementById('adminPasswordInput').value;
            const errorEl = document.getElementById('adminAuthError');

            if (passcode === ADMIN_PASSCODE) {
                window.appState.isAdmin = true;
                errorEl.classList.add('hidden');
                closeAdminModal();
                updateAdminUI();
                showToast("เข้าสู่ระบบผู้ดูแลเรียบร้อยแล้ว", "success");
            } else {
                errorEl.classList.remove('hidden');
            }
        };

        window.logoutAdmin = function() {
            window.appState.isAdmin = false;
            updateAdminUI();
            showToast("ออกจากระบบผู้ดูแลเรียบร้อยแล้ว", "info");
        };

        function updateAdminUI() {
            const isAdmin = window.appState.isAdmin;
            document.getElementById('adminBadgeBanner').classList.toggle('hidden', !isAdmin);
            document.getElementById('adminAddMatchContainer').classList.toggle('hidden', !isAdmin);
            document.getElementById('adminStandingsControls').classList.toggle('hidden', !isAdmin);
            document.getElementById('adminBtnText').textContent = isAdmin ? "โหมด Admin (เปิดอยู่)" : "เข้าสู่ระบบ Admin";
            
            renderMatches();
            renderStandings();
        }

        window.quickUpdateScore = async function(matchId, teamKey, newScore) {
            if (!window.appState.isAdmin) return;
            const db = window.appState.db;
            if (!db) return;

            try {
                const matchRef = doc(db, 'artifacts', appId, 'public', 'data', 'matches', matchId);
                const val = parseInt(newScore, 10) || 0;
                
                await updateDoc(matchRef, {
                    [`${teamKey}.score`]: val,
                    updatedAt: new Date().toISOString()
                });
                showToast("อัปเดตคะแนนเรียลไทม์สำเร็จ", "success");
            } catch (err) {
                console.error("Score update error:", err);
                showToast("เกิดข้อผิดพลาดในการบันทึกคะแนน", "error");
            }
        };

        window.saveMatch = async function(e) {
            e.preventDefault();
            if (!window.appState.isAdmin) return;

            const db = window.appState.db;
            const matchId = document.getElementById('matchFormId').value;

            const matchData = {
                sport: document.getElementById('matchSport').value.trim(),
                status: document.getElementById('matchStatus').value,
                time: document.getElementById('matchTime').value.trim(),
                team1: {
                    name: document.getElementById('team1Name').value.trim(),
                    score: parseInt(document.getElementById('team1Score').value, 10) || 0,
                    logo: document.getElementById('team1Logo').value.trim()
                },
                team2: {
                    name: document.getElementById('team2Name').value.trim(),
                    score: parseInt(document.getElementById('team2Score').value, 10) || 0,
                    logo: document.getElementById('team2Logo').value.trim()
                },
                updatedAt: new Date().toISOString()
            };

            try {
                if (matchId) {
                    const matchRef = doc(db, 'artifacts', appId, 'public', 'data', 'matches', matchId);
                    await updateDoc(matchRef, matchData);
                    showToast("แก้ไขแมตช์การแข่งขันเรียบร้อย", "success");
                } else {
                    const matchesRef = collection(db, 'artifacts', appId, 'public', 'data', 'matches');
                    await addDoc(matchesRef, matchData);
                    showToast("เพิ่มแมตช์การแข่งขันใหม่สำเร็จ", "success");
                }
                closeMatchModal();
            } catch (err) {
                console.error("Save match error:", err);
                showToast("ไม่สามารถบันทึกข้อมูลได้", "error");
            }
        };

        window.deleteMatch = async function(id) {
            if (!window.appState.isAdmin) return;
            const db = window.appState.db;
            try {
                await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'matches', id));
                showToast("ลบแมตช์เรียบร้อยแล้ว", "info");
            } catch (err) {
                showToast("เกิดข้อผิดพลาดในการลบ", "error");
            }
        };

        window.saveTeamStandings = async function(e) {
            e.preventDefault();
            if (!window.appState.isAdmin) return;

            const db = window.appState.db;
            const teamId = document.getElementById('teamFormId').value;

            const data = {
                name: document.getElementById('teamStandingsName').value.trim(),
                logo: document.getElementById('teamStandingsLogo').value.trim(),
                played: parseInt(document.getElementById('teamPlayed').value, 10) || 0,
                won: parseInt(document.getElementById('teamWon').value, 10) || 0,
                lost: parseInt(document.getElementById('teamLost').value, 10) || 0,
                points: parseInt(document.getElementById('teamTotalPoints').value, 10) || 0
            };

            try {
                if (teamId) {
                    await updateDoc(doc(db, 'artifacts', appId, 'public', 'data', 'standings', teamId), data);
                } else {
                    await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'standings'), data);
                }
                closeTeamModal();
                showToast("บันทึกตารางคะแนนสำเร็จ", "success");
            } catch (err) {
                showToast("ไม่สามารถบันทึกได้", "error");
            }
        };

        window.deleteTeamStandings = async function(id) {
            if (!window.appState.isAdmin) return;
            const db = window.appState.db;
            try {
                await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'standings', id));
                showToast("ลบทีมเรียบร้อยแล้ว", "info");
            } catch (err) {
                showToast("ไม่สามารถลบได้", "error");
            }
        };

        window.recalculateStandings = function() {
            showToast("คำนวณและอัปเดตคะแนนรวมเรียบร้อย", "success");
            renderStandings();
        };

        // Initialize App on load
        window.addEventListener('DOMContentLoaded', () => {
            initFirebase();
        });

    </script>

    <script>
        function escapeHtml(str) {
            if (!str) return '';
            return String(str)
                .replace(/&/g, "&amp;")
                .replace(/</g, "&lt;")
                .replace(/>/g, "&gt;")
                .replace(/"/g, "&quot;")
                .replace(/'/g, "&#039;");
        }

        function switchTab(tabName) {
            const tabMatches = document.getElementById('tabMatches');
            const tabStandings = document.getElementById('tabStandings');
            const btnMatches = document.getElementById('tabMatchesBtn');
            const btnStandings = document.getElementById('tabStandingsBtn');

            if (tabName === 'matches') {
                tabMatches.classList.remove('hidden');
                tabStandings.classList.add('hidden');
                
                btnMatches.className = "py-3 px-4 text-sm font-semibold border-b-2 border-blue-500 text-blue-400 flex items-center gap-2 whitespace-nowrap";
                btnStandings.className = "py-3 px-4 text-sm font-semibold border-b-2 border-transparent text-slate-400 hover:text-slate-200 flex items-center gap-2 whitespace-nowrap";
            } else {
                tabMatches.classList.add('hidden');
                tabStandings.classList.remove('hidden');

                btnStandings.className = "py-3 px-4 text-sm font-semibold border-b-2 border-blue-500 text-blue-400 flex items-center gap-2 whitespace-nowrap";
                btnMatches.className = "py-3 px-4 text-sm font-semibold border-b-2 border-transparent text-slate-400 hover:text-slate-200 flex items-center gap-2 whitespace-nowrap";
            }
        }

        // Modals Toggle
        function toggleAdminModal() {
            if (window.appState.isAdmin) {
                logoutAdmin();
            } else {
                document.getElementById('adminPasswordModal').classList.remove('hidden');
            }
        }

        function closeAdminModal() {
            document.getElementById('adminPasswordModal').classList.add('hidden');
            document.getElementById('adminPasswordInput').value = '';
            document.getElementById('adminAuthError').classList.add('hidden');
        }

        function openMatchModal(id = null) {
            const modal = document.getElementById('matchModal');
            document.getElementById('matchFormId').value = id || '';
            document.getElementById('matchModalTitle').textContent = id ? "แก้ไขข้อมูลการแข่งขัน" : "เพิ่มข้อมูลการแข่งขัน";

            if (id) {
                const match = window.appState.matches.find(m => m.id === id);
                if (match) {
                    document.getElementById('matchSport').value = match.sport || '';
                    document.getElementById('matchStatus').value = match.status || 'UPCOMING';
                    document.getElementById('matchTime').value = match.time || '';
                    
                    document.getElementById('team1Name').value = match.team1?.name || '';
                    document.getElementById('team1Score').value = match.team1?.score ?? 0;
                    document.getElementById('team1Logo').value = match.team1?.logo || '';

                    document.getElementById('team2Name').value = match.team2?.name || '';
                    document.getElementById('team2Score').value = match.team2?.score ?? 0;
                    document.getElementById('team2Logo').value = match.team2?.logo || '';
                }
            } else {
                document.getElementById('matchSport').value = '';
                document.getElementById('matchStatus').value = 'UPCOMING';
                document.getElementById('matchTime').value = '';
                document.getElementById('team1Name').value = '';
                document.getElementById('team1Score').value = 0;
                document.getElementById('team1Logo').value = '';
                document.getElementById('team2Name').value = '';
                document.getElementById('team2Score').value = 0;
                document.getElementById('team2Logo').value = '';
            }

            modal.classList.remove('hidden');
        }

        function closeMatchModal() {
            document.getElementById('matchModal').classList.add('hidden');
        }

        function openAddTeamModal(id = null) {
            const modal = document.getElementById('teamModal');
            document.getElementById('teamFormId').value = id || '';

            if (id) {
                const team = window.appState.standings.find(s => s.id === id);
                if (team) {
                    document.getElementById('teamStandingsName').value = team.name || '';
                    document.getElementById('teamStandingsLogo').value = team.logo || '';
                    document.getElementById('teamPlayed').value = team.played || 0;
                    document.getElementById('teamWon').value = team.won || 0;
                    document.getElementById('teamLost').value = team.lost || 0;
                    document.getElementById('teamTotalPoints').value = team.points || 0;
                }
            } else {
                document.getElementById('teamStandingsName').value = '';
                document.getElementById('teamStandingsLogo').value = '';
                document.getElementById('teamPlayed').value = 0;
                document.getElementById('teamWon').value = 0;
                document.getElementById('teamLost').value = 0;
                document.getElementById('teamTotalPoints').value = 0;
            }

            modal.classList.remove('hidden');
        }

        function closeTeamModal() {
            document.getElementById('teamModal').classList.add('hidden');
        }

        function showToast(msg, type = "success") {
            const toast = document.getElementById('toast');
            const toastMsg = document.getElementById('toastMessage');
            const toastIcon = document.getElementById('toastIcon');

            toastMsg.textContent = msg;

            if (type === "success") {
                toastIcon.className = "fa-solid fa-circle-check text-emerald-400";
            } else if (type === "error") {
                toastIcon.className = "fa-solid fa-circle-xmark text-rose-400";
            } else {
                toastIcon.className = "fa-solid fa-circle-info text-blue-400";
            }

            toast.classList.remove('hidden');
            setTimeout(() => {
                toast.classList.add('hidden');
            }, 3000);
        }
    </script>
</body>
</html>
