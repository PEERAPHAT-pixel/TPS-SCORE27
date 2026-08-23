<!DOCTYPE html>
<html lang="th" class="h-full bg-slate-900">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ระบบรายงานผลการแข่งขันสด - Live Scoreboard & Leaderboard</title>
  
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      theme: {
        extend: {
          fontFamily: {
            kanit: ['Kanit', 'sans-serif'],
          },
          colors: {
            gold: '#F59E0B',
            silver: '#9CA3AF',
            bronze: '#D97706',
          }
        }
      }
    }
  </script>

  <!-- Google Fonts Kanit (Ensures identical look on GitHub & external deployments) -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">

  <!-- FontAwesome Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">

  <style>
    body {
      font-family: 'Kanit', sans-serif;
    }
    /* Custom scrollbar */
    ::-webkit-scrollbar {
      width: 8px;
      height: 8px;
    }
    ::-webkit-scrollbar-track {
      background: #1e293b;
    }
    ::-webkit-scrollbar-thumb {
      background: #334155;
      border-radius: 4px;
    }
    ::-webkit-scrollbar-thumb:hover {
      background: #475569;
    }
    .pulse-live {
      box-shadow: 0 0 0 0 rgba(239, 68, 68, 0.7);
      animation: pulse-live-anim 1.5s infinite linear;
    }
    @keyframes pulse-live-anim {
      0% {
        box-shadow: 0 0 0 0 rgba(239, 68, 68, 0.7);
      }
      70% {
        box-shadow: 0 0 0 10px rgba(239, 68, 68, 0);
      }
      100% {
        box-shadow: 0 0 0 0 rgba(239, 68, 68, 0);
      }
    }
  </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen flex flex-col antialiased">

  <!-- Top Navigation Header -->
  <header class="bg-slate-900/90 backdrop-blur border-b border-slate-800 sticky top-0 z-30">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between">
      
      <!-- Logo / Title -->
      <div class="flex items-center space-x-3">
        <div class="w-10 h-10 rounded-xl bg-gradient-to-tr from-amber-500 to-indigo-600 flex items-center justify-center text-white shadow-lg shadow-indigo-500/20">
          <i class="fa-solid font-bold fa-trophy text-xl"></i>
        </div>
        <div>
          <h1 class="text-lg sm:text-xl font-bold tracking-tight text-white flex items-center gap-2">
            <span>LIVE SCORE CENTER</span>
            <span class="text-xs bg-indigo-500/20 text-indigo-400 px-2 py-0.5 rounded-full font-medium border border-indigo-500/30">REALTIME</span>
          </h1>
          <p class="text-xs text-slate-400 hidden sm:block">ระบบรายงานผลการแข่งขันสดและคำนวณคะแนนเหรียญรางวัล</p>
        </div>
      </div>

      <!-- Controls & Admin Status -->
      <div class="flex items-center space-x-3">
        <!-- Live Connection Indicator -->
        <div class="flex items-center space-x-2 bg-slate-800/80 px-3 py-1.5 rounded-full border border-slate-700/60 text-xs text-slate-300">
          <span id="sync-indicator" class="w-2.5 h-2.5 rounded-full bg-emerald-500 animate-pulse"></span>
          <span id="sync-text" class="hidden sm:inline">เชื่อมต่อเรียบร้อย</span>
        </div>

        <!-- Admin Login / Logout Toggle -->
        <div id="admin-auth-container">
          <button id="btn-admin-login-trigger" onclick="openAdminModal()" class="px-3.5 py-1.5 text-xs sm:text-sm font-medium bg-slate-800 hover:bg-slate-700 text-slate-200 border border-slate-700 rounded-lg transition-all flex items-center gap-2 shadow-sm">
            <i class="fa-solid fa-lock text-slate-400"></i>
            <span>เข้าสู่ระบบแอดมิน</span>
          </button>
        </div>
      </div>

    </div>
  </header>

  <!-- Navigation Tabs -->
  <nav class="bg-slate-900 border-b border-slate-800 px-4">
    <div class="max-w-7xl mx-auto flex space-x-1 sm:space-x-4 overflow-x-auto py-2" id="nav-tabs">
      <button onclick="switchTab('matches')" id="tab-btn-matches" class="px-4 py-2 text-sm font-medium rounded-lg transition-all flex items-center gap-2 bg-indigo-600 text-white shadow-md shadow-indigo-600/30">
        <i class="fa-solid fa-gamepad"></i>
        <span>ตารางการแข่งขันสด</span>
        <span id="live-match-badge" class="ml-1 bg-red-500 text-white text-xs px-1.5 py-0.2 rounded-full hidden">0</span>
      </button>

      <button onclick="switchTab('leaderboard')" id="tab-btn-leaderboard" class="px-4 py-2 text-sm font-medium rounded-lg transition-all flex items-center gap-2 text-slate-400 hover:bg-slate-800 hover:text-slate-200">
        <i class="fa-solid fa-medal text-amber-400"></i>
        <span>สรุปผลคะแนน & โอกาสเหรียญ</span>
      </button>

      <button onclick="switchTab('admin')" id="tab-btn-admin" class="px-4 py-2 text-sm font-medium rounded-lg transition-all flex items-center gap-2 text-slate-400 hover:bg-slate-800 hover:text-slate-200 hidden">
        <i class="fa-solid fa-sliders text-indigo-400"></i>
        <span>การจัดการ (แอดมิน)</span>
      </button>
    </div>
  </nav>

  <!-- Main Content Container -->
  <main class="flex-1 max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6 space-y-6">

    <!-- Toast Notification Banner -->
    <div id="toast-message" class="hidden fixed bottom-5 right-5 z-50 bg-indigo-600 text-white px-5 py-3 rounded-xl shadow-2xl flex items-center gap-3 border border-indigo-400/30 transform transition-all duration-300">
      <i id="toast-icon" class="fa-solid fa-circle-check text-lg"></i>
      <span id="toast-text" class="text-sm font-medium">ทำรายการสำเร็จ</span>
    </div>

    <!-- ==================== TAB 1: MATCHES VIEW ==================== -->
    <section id="view-matches" class="space-y-6">
      
      <!-- Filter Bar -->
      <div class="bg-slate-900 border border-slate-800 rounded-2xl p-4 flex flex-col md:flex-row gap-4 items-center justify-between shadow-lg">
        
        <!-- Category Filters -->
        <div class="flex items-center gap-2 overflow-x-auto w-full md:w-auto pb-2 md:pb-0" id="sport-filter-buttons">
          <button onclick="filterBySport('all')" class="sport-filter-btn px-3.5 py-1.5 rounded-lg text-xs sm:text-sm font-medium bg-indigo-600 text-white transition-all whitespace-nowrap" data-sport="all">
            ทั้งหมด
          </button>
          <!-- Dynamic sport filters injected here -->
        </div>

        <!-- Search / Quick Filter -->
        <div class="relative w-full md:w-64">
          <i class="fa-solid fa-magnifying-glass absolute left-3 top-1/2 -translate-y-1/2 text-slate-500 text-sm"></i>
          <input type="text" id="search-match-input" onkeyup="renderMatches()" placeholder="ค้นหาชื่อทีม หรือ กีฬา..." class="w-full bg-slate-950 border border-slate-800 rounded-xl pl-9 pr-4 py-1.5 text-sm text-slate-200 focus:outline-none focus:border-indigo-500">
        </div>

      </div>

      <!-- Matches Display Grid -->
      <div id="matches-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-5">
        <!-- Dynamic match cards will be rendered here -->
      </div>

      <!-- Empty State for Matches -->
      <div id="matches-empty" class="hidden text-center py-16 bg-slate-900/50 rounded-2xl border border-dashed border-slate-800">
        <div class="w-16 h-16 mx-auto mb-4 bg-slate-800/80 text-slate-500 rounded-full flex items-center justify-center text-2xl">
          <i class="fa-solid fa-volleyball"></i>
        </div>
        <h3 class="text-lg font-semibold text-slate-300">ยังไม่มีรายการแมตช์การแข่งขัน</h3>
        <p class="text-sm text-slate-500 max-w-sm mx-auto mt-1">ผู้ดูแลระบบ (Admin) สามารถเพิ่มประเภทกีฬาและจัดการสร้างแมตช์ใหม่ได้ทางเมนูแอดมิน</p>
      </div>

    </section>

    <!-- ==================== TAB 2: LEADERBOARD & MEDAL PROJECTIONS ==================== -->
    <section id="view-leaderboard" class="hidden space-y-6">
      
      <!-- Projection Logic Info Box -->
      <div class="bg-gradient-to-r from-slate-900 via-indigo-950/40 to-slate-900 border border-indigo-500/20 rounded-2xl p-5 shadow-lg flex flex-col md:flex-row items-start md:items-center justify-between gap-4">
        <div>
          <h2 class="text-lg font-bold text-white flex items-center gap-2">
            <i class="fa-solid fa-chart-line text-indigo-400"></i>
            <span>สรุปผลคะแนนรวม & การวิเคราะห์โอกาสเหรียญรางวัล</span>
          </h2>
          <p class="text-xs text-slate-400 mt-1">
            เกณฑ์คะแนนคำนวณจากคะแนนรวมทุกรายการแข่งขัน โดยเปรียบเทียบกับเกณฑ์มาตรฐานที่แอดมินกำหนด
          </p>
        </div>

        <div class="flex items-center gap-3 text-xs bg-slate-900/90 border border-slate-800 p-2.5 rounded-xl">
          <div class="flex items-center gap-1.5"><span class="w-3 h-3 rounded-full bg-amber-500"></span><span class="text-slate-300">ทอง (&ge; <span id="thresh-gold-label">100</span>)</span></div>
          <div class="flex items-center gap-1.5"><span class="w-3 h-3 rounded-full bg-slate-400"></span><span class="text-slate-300">เงิน (&ge; <span id="thresh-silver-label">60</span>)</span></div>
          <div class="flex items-center gap-1.5"><span class="w-3 h-3 rounded-full bg-amber-700"></span><span class="text-slate-300">ทองแดง (&ge; <span id="thresh-bronze-label">30</span>)</span></div>
        </div>
      </div>

      <!-- Leaderboard Table -->
      <div class="bg-slate-900 border border-slate-800 rounded-2xl shadow-xl overflow-hidden">
        <div class="overflow-x-auto">
          <table class="w-full text-left border-collapse">
            <thead>
              <tr class="bg-slate-950/80 border-b border-slate-800 text-xs font-semibold text-slate-400 uppercase tracking-wider">
                <th class="py-4 px-4 text-center w-16">อันดับ</th>
                <th class="py-4 px-4">ทีม / สโมสร</th>
                <th class="py-4 px-4 text-center">แข่งแล้ว</th>
                <th class="py-4 px-4 text-center">ชนะ</th>
                <th class="py-4 px-4 text-center">เสมอ</th>
                <th class="py-4 px-4 text-center">แพ้</th>
                <th class="py-4 px-4 text-center font-bold text-indigo-400">คะแนนรวม</th>
                <th class="py-4 px-4 text-center">คาดการณ์เหรียญ</th>
              </tr>
            </thead>
            <tbody id="leaderboard-tbody" class="divide-y divide-slate-800/60 text-sm">
              <!-- Dynamic overall ranking will be injected here -->
            </tbody>
          </table>
        </div>
      </div>

      <!-- Empty State Leaderboard -->
      <div id="leaderboard-empty" class="hidden text-center py-16 bg-slate-900/50 rounded-2xl border border-dashed border-slate-800">
        <div class="w-16 h-16 mx-auto mb-4 bg-slate-800/80 text-amber-500 rounded-full flex items-center justify-center text-2xl">
          <i class="fa-solid fa-trophy"></i>
        </div>
        <h3 class="text-lg font-semibold text-slate-300">ยังไม่มีข้อมูลตารางคะแนน</h3>
        <p class="text-sm text-slate-500 max-w-sm mx-auto mt-1">คะแนนรวมและอันดับจะถูกคำนวณอัตโนมัติเมื่อเริ่มมีการระบุผลการแข่งขัน</p>
      </div>

    </section>

    <!-- ==================== TAB 3: ADMIN CONTROL PANEL ==================== -->
    <section id="view-admin" class="hidden space-y-6">
      
      <!-- Admin Notice Bar -->
      <div class="bg-amber-500/10 border border-amber-500/30 rounded-2xl p-4 flex items-center justify-between">
        <div class="flex items-center gap-3">
          <i class="fa-solid fa-user-shield text-amber-400 text-xl"></i>
          <div>
            <h3 class="text-sm font-semibold text-amber-300">ระบบอยู่ในโหมดผู้ดูแลระบบ (Admin Control Mode)</h3>
            <p class="text-xs text-amber-200/70">คุณสามารถเพิ่มประเภทกีฬา สร้างแมตช์ อัปเดตผลคะแนนสด และปรับเกณฑ์เหรียญรางวัลได้ตลอดเวลา</p>
          </div>
        </div>
        <button onclick="adminLogout()" class="px-3 py-1.5 text-xs bg-red-500/20 hover:bg-red-500/30 text-red-300 border border-red-500/40 rounded-lg transition-all font-medium">
          ออกจากระบบ
        </button>
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
        
        <!-- Left Side: Management Forms -->
        <div class="lg:col-span-1 space-y-6">

          <!-- 1. Add Sports Form -->
          <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-lg space-y-4">
            <h3 class="text-base font-bold text-white flex items-center gap-2 border-b border-slate-800 pb-3">
              <i class="fa-solid fa-basketball text-indigo-400"></i>
              <span>1. เพิ่มประเภทกีฬา</span>
            </h3>

            <form id="form-add-sport" onsubmit="handleCreateSport(event)" class="space-y-3">
              <div>
                <label class="block text-xs font-medium text-slate-400 mb-1">ชื่อประเภทกีฬา *</label>
                <input type="text" id="sport-name-input" required placeholder="เช่น ฟุตบอล, แบดมินตัน, อีสปอร์ต" class="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2 text-sm text-slate-200 focus:outline-none focus:border-indigo-500">
              </div>
              <button type="submit" class="w-full py-2 bg-indigo-600 hover:bg-indigo-500 text-white rounded-xl text-sm font-medium transition-all shadow-md shadow-indigo-600/30 flex items-center justify-center gap-2">
                <i class="fa-solid fa-plus"></i>
                <span>เพิ่มชนิดกีฬา</span>
              </button>
            </form>

            <!-- Sports List Pills -->
            <div class="pt-2">
              <label class="block text-xs font-medium text-slate-400 mb-2">ชนิดกีฬาที่มีในระบบ:</label>
              <div id="admin-sports-list" class="flex flex-wrap gap-2">
                <span class="text-xs text-slate-500">ยังไม่มีชนิดกีฬา</span>
              </div>
            </div>
          </div>

          <!-- 2. Medal Criteria Threshold Settings Form -->
          <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-lg space-y-4">
            <h3 class="text-base font-bold text-white flex items-center gap-2 border-b border-slate-800 pb-3">
              <i class="fa-solid fa-sliders text-amber-400"></i>
              <span>2. ตั้งค่าเกณฑ์คะแนนคำนวณเหรียญ</span>
            </h3>

            <form id="form-medal-settings" onsubmit="handleSaveThresholds(event)" class="space-y-3">
              <div>
                <label class="block text-xs font-medium text-slate-400 mb-1">คะแนนขั้นต่ำ - เหรียญทอง 🥇</label>
                <input type="number" id="thresh-gold-input" min="0" value="100" class="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2 text-sm text-slate-200 focus:outline-none focus:border-indigo-500">
              </div>
              <div>
                <label class="block text-xs font-medium text-slate-400 mb-1">คะแนนขั้นต่ำ - เหรียญเงิน 🥈</label>
                <input type="number" id="thresh-silver-input" min="0" value="60" class="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2 text-sm text-slate-200 focus:outline-none focus:border-indigo-500">
              </div>
              <div>
                <label class="block text-xs font-medium text-slate-400 mb-1">คะแนนขั้นต่ำ - เหรียญทองแดง 🥉</label>
                <input type="number" id="thresh-bronze-input" min="0" value="30" class="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2 text-sm text-slate-200 focus:outline-none focus:border-indigo-500">
              </div>
              <button type="submit" class="w-full py-2 bg-amber-600 hover:bg-amber-500 text-white rounded-xl text-sm font-medium transition-all shadow-md shadow-amber-600/30 flex items-center justify-center gap-2">
                <i class="fa-solid fa-floppy-disk"></i>
                <span>บันทึกเกณฑ์คำนวณ</span>
              </button>
            </form>
          </div>

        </div>

        <!-- Right Side: Add / Manage Matches -->
        <div class="lg:col-span-2 space-y-6">

          <!-- Add Match Form Card -->
          <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-lg space-y-4">
            <h3 class="text-base font-bold text-white flex items-center gap-2 border-b border-slate-800 pb-3">
              <i class="fa-solid fa-square-plus text-emerald-400"></i>
              <span>3. สร้างแมตช์การแข่งขันใหม่</span>
            </h3>

            <form id="form-create-match" onsubmit="handleCreateMatch(event)" class="space-y-4">
              
              <!-- Select Sport & Match Title -->
              <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                <div>
                  <label class="block text-xs font-medium text-slate-400 mb-1">ประเภทกีฬา *</label>
                  <select id="match-sport-select" required class="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2 text-sm text-slate-200 focus:outline-none focus:border-indigo-500">
                    <option value="">-- กรุณาเลือกกีฬา --</option>
                  </select>
                </div>
                <div>
                  <label class="block text-xs font-medium text-slate-400 mb-1">ชื่อรอบ / คำอธิบายแมตช์</label>
                  <input type="text" id="match-title-input" placeholder="เช่น รอบชิงชนะเลิศ, สาย A แมตช์ 1" class="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2 text-sm text-slate-200 focus:outline-none focus:border-indigo-500">
                </div>
              </div>

              <!-- Team A Details -->
              <div class="bg-slate-950 p-4 rounded-xl border border-slate-800/80 space-y-3">
                <h4 class="text-xs font-bold text-indigo-400 uppercase tracking-wider flex items-center gap-2">
                  <i class="fa-solid fa-users"></i> ข้อมูลทีม A (เจ้าบ้าน / ฝ่ายแรก)
                </h4>
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                  <div>
                    <label class="block text-xs text-slate-400 mb-1">ชื่อทีม A *</label>
                    <input type="text" id="teamA-name-input" required placeholder="ชื่อทีม A" class="w-full bg-slate-900 border border-slate-800 rounded-lg px-3 py-1.5 text-sm text-slate-200 focus:outline-none focus:border-indigo-500">
                  </div>
                  <div>
                    <label class="block text-xs text-slate-400 mb-1">โลโก้ทีม A (ไฟล์ไม่เกิน 10MB)</label>
                    <input type="file" id="teamA-logo-file" accept="image/*" onchange="handleLogoUpload(this, 'teamA-preview')" class="w-full text-xs text-slate-400 file:mr-2 file:py-1 file:px-2 file:rounded-lg file:border-0 file:text-xs file:bg-slate-800 file:text-slate-200 hover:file:bg-slate-700">
                    <input type="hidden" id="teamA-logo-data">
                  </div>
                </div>
                <div id="teamA-preview" class="hidden flex items-center gap-2 text-xs text-slate-400 pt-1">
                  <img src="" class="w-8 h-8 rounded-full object-cover border border-slate-700">
                  <span>แนบไฟล์โลโก้สำเร็จ</span>
                </div>
              </div>

              <!-- Team B Details -->
              <div class="bg-slate-950 p-4 rounded-xl border border-slate-800/80 space-y-3">
                <h4 class="text-xs font-bold text-red-400 uppercase tracking-wider flex items-center gap-2">
                  <i class="fa-solid fa-users"></i> ข้อมูลทีม B (ทีมเยือน / ฝ่ายสอง)
                </h4>
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                  <div>
                    <label class="block text-xs text-slate-400 mb-1">ชื่อทีม B *</label>
                    <input type="text" id="teamB-name-input" required placeholder="ชื่อทีม B" class="w-full bg-slate-900 border border-slate-800 rounded-lg px-3 py-1.5 text-sm text-slate-200 focus:outline-none focus:border-indigo-500">
                  </div>
                  <div>
                    <label class="block text-xs text-slate-400 mb-1">โลโก้ทีม B (ไฟล์ไม่เกิน 10MB)</label>
                    <input type="file" id="teamB-logo-file" accept="image/*" onchange="handleLogoUpload(this, 'teamB-preview')" class="w-full text-xs text-slate-400 file:mr-2 file:py-1 file:px-2 file:rounded-lg file:border-0 file:text-xs file:bg-slate-800 file:text-slate-200 hover:file:bg-slate-700">
                    <input type="hidden" id="teamB-logo-data">
                  </div>
                </div>
                <div id="teamB-preview" class="hidden flex items-center gap-2 text-xs text-slate-400 pt-1">
                  <img src="" class="w-8 h-8 rounded-full object-cover border border-slate-700">
                  <span>แนบไฟล์โลโก้สำเร็จ</span>
                </div>
              </div>

              <!-- Match Status & Initial Scores -->
              <div class="grid grid-cols-1 sm:grid-cols-3 gap-3">
                <div>
                  <label class="block text-xs font-medium text-slate-400 mb-1">สถานะการแข่งขัน</label>
                  <select id="match-status-select" class="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2 text-sm text-slate-200 focus:outline-none focus:border-indigo-500">
                    <option value="UPCOMING">ยังไม่เริ่มแข่ง</option>
                    <option value="LIVE">กำลังแข่งขัน (LIVE)</option>
                    <option value="FINISHED">จบการแข่งขัน</option>
                  </select>
                </div>
                <div>
                  <label class="block text-xs font-medium text-slate-400 mb-1">คะแนนเริ่มต้น ทีม A</label>
                  <input type="number" id="teamA-score-init" value="0" min="0" class="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2 text-sm text-slate-200 focus:outline-none focus:border-indigo-500">
                </div>
                <div>
                  <label class="block text-xs font-medium text-slate-400 mb-1">คะแนนเริ่มต้น ทีม B</label>
                  <input type="number" id="teamB-score-init" value="0" min="0" class="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2 text-sm text-slate-200 focus:outline-none focus:border-indigo-500">
                </div>
              </div>

              <button type="submit" class="w-full py-2.5 bg-emerald-600 hover:bg-emerald-500 text-white rounded-xl font-medium text-sm transition-all shadow-lg shadow-emerald-600/30 flex items-center justify-center gap-2">
                <i class="fa-solid fa-plus-circle"></i>
                <span>บันทึกสร้างแมตช์</span>
              </button>

            </form>
          </div>

        </div>

      </div>

    </section>

  </main>

  <!-- ==================== MODALS ==================== -->

  <!-- Admin Login Password Modal -->
  <div id="modal-admin-login" class="hidden fixed inset-0 z-50 bg-slate-950/80 backdrop-blur-md flex items-center justify-center p-4">
    <div class="bg-slate-900 border border-slate-800 rounded-2xl max-w-md w-full p-6 shadow-2xl relative space-y-4">
      
      <button onclick="closeAdminModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white">
        <i class="fa-solid fa-xmark text-lg"></i>
      </button>

      <div class="text-center space-y-2">
        <div class="w-12 h-12 mx-auto bg-indigo-500/10 text-indigo-400 border border-indigo-500/20 rounded-2xl flex items-center justify-center text-xl">
          <i class="fa-solid fa-shield-halved"></i>
        </div>
        <h3 class="text-lg font-bold text-white">ยืนยันสิทธิ์ผู้ดูแลระบบ (Admin)</h3>
        <p class="text-xs text-slate-400">กรุณากรอกรหัสผ่านผู้ดูแลระบบเพื่อทำการแก้ไขข้อมูลคะแนนและแมตช์</p>
      </div>

      <form onsubmit="handleAdminAuth(event)" class="space-y-4 pt-2">
        <div>
          <label class="block text-xs text-slate-400 mb-1">รหัสผ่านแอดมิน (Admin Passcode)</label>
          <div class="relative">
            <input type="password" id="admin-passcode-input" required placeholder="••••••••••••••••" class="w-full bg-slate-950 border border-slate-800 rounded-xl px-4 py-2.5 text-sm text-slate-200 focus:outline-none focus:border-indigo-500 pr-10">
            <button type="button" onclick="togglePasswordVisibility()" class="absolute right-3 top-1/2 -translate-y-1/2 text-slate-500 hover:text-slate-300">
              <i id="eye-icon" class="fa-solid fa-eye"></i>
            </button>
          </div>
          <p id="admin-auth-error" class="hidden text-xs text-red-400 mt-1.5 flex items-center gap-1">
            <i class="fa-solid fa-circle-exclamation"></i>
            <span>รหัสผ่านไม่ถูกต้อง กรุณาลองใหม่อีกครั้ง</span>
          </p>
        </div>

        <button type="submit" class="w-full py-2.5 bg-indigo-600 hover:bg-indigo-500 text-white font-medium text-sm rounded-xl transition-all shadow-md shadow-indigo-600/30">
          เข้าสู่ระบบ
        </button>
      </form>

    </div>
  </div>

  <!-- Fullscreen Match Details & Scorecard Graphic Modal -->
  <div id="modal-match-fullscreen" class="hidden fixed inset-0 z-50 bg-slate-950/90 backdrop-blur-lg flex items-center justify-center p-4 overflow-y-auto">
    <div class="bg-slate-900 border border-slate-800 rounded-3xl max-w-2xl w-full p-6 shadow-2xl relative space-y-6 my-auto">
      
      <button onclick="closeMatchFullscreenModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white w-8 h-8 rounded-full bg-slate-800 flex items-center justify-center">
        <i class="fa-solid fa-xmark"></i>
      </button>

      <!-- Fullscreen Graphic Container -->
      <div id="fullscreen-card-export" class="bg-gradient-to-br from-slate-950 via-slate-900 to-indigo-950 p-6 rounded-2xl border border-slate-800 space-y-6 text-center">
        
        <!-- Header -->
        <div class="flex items-center justify-between border-b border-slate-800 pb-3">
          <span id="fs-sport-badge" class="px-3 py-1 bg-indigo-500/20 text-indigo-300 text-xs font-semibold rounded-full border border-indigo-500/30">กีฬา</span>
          <span id="fs-status-badge" class="text-xs font-bold px-2.5 py-1 rounded-full">สถานะ</span>
        </div>

        <p id="fs-match-title" class="text-sm text-slate-400 font-medium">คำอธิบายแมตช์</p>

        <!-- Teams Showcase -->
        <div class="grid grid-cols-7 items-center gap-2 py-4">
          
          <!-- Team A -->
          <div class="col-span-3 flex flex-col items-center space-y-3">
            <div class="w-20 h-20 sm:w-24 sm:h-24 rounded-2xl bg-slate-800 p-2 border border-slate-700/80 shadow-xl flex items-center justify-center overflow-hidden">
              <img id="fs-teamA-logo" src="" class="max-w-full max-h-full object-contain">
            </div>
            <h3 id="fs-teamA-name" class="text-base sm:text-lg font-bold text-white max-w-full truncate">ทีม A</h3>
            <span id="fs-teamA-score" class="text-4xl sm:text-5xl font-black text-indigo-400">0</span>
          </div>

          <!-- VS Divider -->
          <div class="col-span-1 flex flex-col items-center justify-center">
            <span class="text-xs font-extrabold tracking-widest text-slate-500 bg-slate-800 px-2 py-1 rounded-md">VS</span>
          </div>

          <!-- Team B -->
          <div class="col-span-3 flex flex-col items-center space-y-3">
            <div class="w-20 h-20 sm:w-24 sm:h-24 rounded-2xl bg-slate-800 p-2 border border-slate-700/80 shadow-xl flex items-center justify-center overflow-hidden">
              <img id="fs-teamB-logo" src="" class="max-w-full max-h-full object-contain">
            </div>
            <h3 id="fs-teamB-name" class="text-base sm:text-lg font-bold text-white max-w-full truncate">ทีม B</h3>
            <span id="fs-teamB-score" class="text-4xl sm:text-5xl font-black text-red-400">0</span>
          </div>

        </div>

        <div class="pt-2 text-xs text-slate-500 border-t border-slate-800/80 flex items-center justify-between">
          <span>LIVE SCORE SYSTEM</span>
          <span id="fs-timestamp">--:--</span>
        </div>

      </div>

      <!-- Action Buttons -->
      <div class="flex flex-wrap gap-3 justify-end">
        <button onclick="downloadMatchScorecard()" class="px-4 py-2 bg-indigo-600 hover:bg-indigo-500 text-white rounded-xl text-sm font-medium transition-all flex items-center gap-2 shadow-md">
          <i class="fa-solid fa-download"></i>
          <span>ดาวน์โหลดรูปการแข่งขัน</span>
        </button>
        <button onclick="closeMatchFullscreenModal()" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 text-slate-300 rounded-xl text-sm font-medium transition-all">
          ปิดหน้าต่าง
        </button>
      </div>

    </div>
  </div>

  <!-- Realtime Score Quick Update Modal (Admin) -->
  <div id="modal-quick-score" class="hidden fixed inset-0 z-50 bg-slate-950/80 backdrop-blur-md flex items-center justify-center p-4">
    <div class="bg-slate-900 border border-slate-800 rounded-2xl max-w-md w-full p-6 shadow-2xl relative space-y-4">
      
      <button onclick="closeQuickScoreModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white">
        <i class="fa-solid fa-xmark text-lg"></i>
      </button>

      <h3 class="text-base font-bold text-white flex items-center gap-2">
        <i class="fa-solid fa-pen-to-square text-indigo-400"></i>
        <span>แก้ไขผลคะแนนสด (Realtime)</span>
      </h3>

      <form id="form-quick-score" onsubmit="handleSaveScoreUpdate(event)" class="space-y-4">
        <input type="hidden" id="qs-match-id">

        <div>
          <label class="block text-xs font-medium text-slate-400 mb-1">สถานะแมตช์</label>
          <select id="qs-status" class="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2 text-sm text-slate-200">
            <option value="UPCOMING">ยังไม่เริ่มแข่ง</option>
            <option value="LIVE">กำลังแข่งขัน (LIVE)</option>
            <option value="FINISHED">จบการแข่งขัน</option>
          </select>
        </div>

        <div class="grid grid-cols-2 gap-4">
          <div class="bg-slate-950 p-3 rounded-xl border border-slate-800 text-center space-y-2">
            <span id="qs-teamA-label" class="text-xs font-bold text-indigo-400 truncate block">ทีม A</span>
            <input type="number" id="qs-teamA-score" min="0" class="w-full text-center text-2xl font-bold bg-slate-900 border border-slate-700 rounded-lg py-1.5 text-white">
          </div>
          <div class="bg-slate-950 p-3 rounded-xl border border-slate-800 text-center space-y-2">
            <span id="qs-teamB-label" class="text-xs font-bold text-red-400 truncate block">ทีม B</span>
            <input type="number" id="qs-teamB-score" min="0" class="w-full text-center text-2xl font-bold bg-slate-900 border border-slate-700 rounded-lg py-1.5 text-white">
          </div>
        </div>

        <button type="submit" class="w-full py-2.5 bg-emerald-600 hover:bg-emerald-500 text-white font-medium text-sm rounded-xl transition-all shadow-md shadow-emerald-600/30">
          อัปเดตคะแนนทันที
        </button>
      </form>

    </div>
  </div>

  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
    import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
    import { getFirestore, doc, collection, onSnapshot, setDoc, addDoc, updateDoc, deleteDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

    // Global App configuration and state
    const appId = typeof __app_id !== 'undefined' ? __app_id : 'live-sports-app-default';
    const firebaseConfig = typeof __firebase_config !== 'undefined' 
      ? JSON.parse(__firebase_config) 
      : { apiKey: "demo", authDomain: "demo.firebaseapp.com", projectId: "demo" };

    const app = initializeApp(firebaseConfig);
    const auth = getAuth(app);
    const db = getFirestore(app);

    // State Variables
    let currentUser = null;
    let isAdminLoggedIn = false;
    let currentSportFilter = 'all';
    
    // In-memory synced collections
    let sportsData = [];
    let matchesData = [];
    let medalThresholds = { gold: 100, silver: 60, bronze: 30 };

    // Passcode stored securely in memory for auth check
    const ADMIN_KEY = "07Poyu_@841lowl[rirjfloe=10kunla2";

    // Fallback default team logo SVG placeholder
    const DEFAULT_LOGO = "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='100' height='100' viewBox='0 0 100 100'><rect width='100' height='100' rx='20' fill='%23334155'/><text x='50%' y='55%' font-family='Arial' font-size='32' fill='%2394a3b8' text-anchor='middle' dominant-baseline='middle'>🏆</text></svg>";

    // Initialize Firebase Auth & Realtime Sync
    async function initFirebase() {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          const userCredential = await signInWithCustomToken(auth, __initial_auth_token);
          currentUser = userCredential.user;
        } else {
          const userCredential = await signInAnonymously(auth);
          currentUser = userCredential.user;
        }

        updateSyncStatus(true, 'เชื่อมต่อเรียบร้อย');
        updateAdminUIState(); // ซ่อนส่วนการจัดการแอดมินสำหรับคนดูทั่วไปทันทีเมื่อเริ่มทำงาน
        subscribeRealtimeData();
      } catch (err) {
        console.error("Auth / Init error:", err);
        updateSyncStatus(false, 'เกิดข้อผิดพลาดในการเชื่อมต่อ');
      }
    }

    // Subscribe to Firestore collections with Strict Rule 1 paths
    function subscribeRealtimeData() {
      if (!currentUser) return;

      // 1. Sports list snapshot
      const sportsCol = collection(db, 'artifacts', appId, 'public', 'data', 'sports');
      onSnapshot(sportsCol, (snapshot) => {
        sportsData = [];
        snapshot.forEach(doc => {
          sportsData.push({ id: doc.id, ...doc.data() });
        });
        renderSportFilters();
        renderAdminSportsList();
        populateMatchSportSelect();
      }, (error) => {
        console.error("Sports snapshot error:", error);
      });

      // 2. Matches snapshot
      const matchesCol = collection(db, 'artifacts', appId, 'public', 'data', 'matches');
      onSnapshot(matchesCol, (snapshot) => {
        matchesData = [];
        snapshot.forEach(doc => {
          matchesData.push({ id: doc.id, ...doc.data() });
        });
        renderMatches();
        renderLeaderboard();
      }, (error) => {
        console.error("Matches snapshot error:", error);
      });

      // 3. Settings / Medal Thresholds snapshot
      const settingsDoc = doc(db, 'artifacts', appId, 'public', 'data', 'settings', 'medal_rules');
      onSnapshot(settingsDoc, (docSnap) => {
        if (docSnap.exists()) {
          medalThresholds = docSnap.data();
          updateThresholdLabels();
          renderLeaderboard();
        }
      }, (error) => {
        console.error("Settings snapshot error:", error);
      });
    }

    // Update Sync indicator dot
    function updateSyncStatus(isOk, text) {
      const dot = document.getElementById('sync-indicator');
      const textElem = document.getElementById('sync-text');
      if (dot && textElem) {
        dot.className = `w-2.5 h-2.5 rounded-full ${isOk ? 'bg-emerald-500 animate-pulse' : 'bg-red-500'}`;
        textElem.innerText = text;
      }
    }

    // Render Sport Filter Buttons in Tab 1
    window.renderSportFilters = function() {
      const container = document.getElementById('sport-filter-buttons');
      if (!container) return;

      let html = `
        <button onclick="filterBySport('all')" class="sport-filter-btn px-3.5 py-1.5 rounded-lg text-xs sm:text-sm font-medium ${currentSportFilter === 'all' ? 'bg-indigo-600 text-white' : 'bg-slate-800 text-slate-400 hover:bg-slate-700'} transition-all whitespace-nowrap" data-sport="all">
          ทั้งหมด
        </button>
      `;

      sportsData.forEach(sp => {
        const active = currentSportFilter === sp.id;
        html += `
          <button onclick="filterBySport('${sp.id}')" class="sport-filter-btn px-3.5 py-1.5 rounded-lg text-xs sm:text-sm font-medium ${active ? 'bg-indigo-600 text-white' : 'bg-slate-800 text-slate-400 hover:bg-slate-700'} transition-all whitespace-nowrap" data-sport="${sp.id}">
            ${escapeHtml(sp.name)}
          </button>
        `;
      });

      container.innerHTML = html;
    };

    window.filterBySport = function(sportId) {
      currentSportFilter = sportId;
      renderSportFilters();
      renderMatches();
    };

    // Render Matches Grid
    window.renderMatches = function() {
      const grid = document.getElementById('matches-grid');
      const emptyState = document.getElementById('matches-empty');
      const searchInput = document.getElementById('search-match-input');
      const searchQuery = searchInput ? searchInput.value.toLowerCase().trim() : '';

      if (!grid) return;

      let filtered = matchesData.filter(m => {
        if (currentSportFilter !== 'all' && m.sportId !== currentSportFilter) return false;
        if (searchQuery) {
          const matchText = `${m.teamA.name} ${m.teamB.name} ${m.title || ''} ${m.sportName || ''}`.toLowerCase();
          if (!matchText.includes(searchQuery)) return false;
        }
        return true;
      });

      // Update live match badge
      const liveCount = matchesData.filter(m => m.status === 'LIVE').length;
      const liveBadge = document.getElementById('live-match-badge');
      if (liveBadge) {
        if (liveCount > 0) {
          liveBadge.innerText = liveCount;
          liveBadge.classList.remove('hidden');
        } else {
          liveBadge.classList.add('hidden');
        }
      }

      if (filtered.length === 0) {
        grid.innerHTML = '';
        emptyState.classList.remove('hidden');
        return;
      }

      emptyState.classList.add('hidden');

      grid.innerHTML = filtered.map(match => {
        const isLive = match.status === 'LIVE';
        const isFinished = match.status === 'FINISHED';
        
        let statusBadgeClass = 'bg-slate-800 text-slate-400 border-slate-700';
        let statusText = 'ยังไม่เริ่ม';
        if (isLive) {
          statusBadgeClass = 'bg-red-500/20 text-red-400 border-red-500/30 pulse-live';
          statusText = '• กำลังแข่ง LIVE';
        } else if (isFinished) {
          statusBadgeClass = 'bg-emerald-500/20 text-emerald-400 border-emerald-500/30';
          statusText = 'จบการแข่งขัน';
        }

        const teamALogo = match.teamA.logo || DEFAULT_LOGO;
        const teamBLogo = match.teamB.logo || DEFAULT_LOGO;

        return `
          <div class="bg-slate-900 border border-slate-800/90 hover:border-indigo-500/40 transition-all rounded-2xl p-5 shadow-lg flex flex-col justify-between space-y-4">
            
            <!-- Match Header Info -->
            <div class="flex items-center justify-between border-b border-slate-800/80 pb-3">
              <span class="px-2.5 py-1 bg-slate-800 text-slate-300 text-xs rounded-lg font-medium">
                ${escapeHtml(match.sportName || 'กีฬา')}
              </span>
              <span class="text-xs font-semibold px-2.5 py-0.5 rounded-full border ${statusBadgeClass}">
                ${statusText}
              </span>
            </div>

            <!-- Match Title / Note -->
            ${match.title ? `<p class="text-xs text-slate-400 text-center font-medium">${escapeHtml(match.title)}</p>` : ''}

            <!-- Match Teams & Scores -->
            <div class="grid grid-cols-7 items-center gap-2 py-2">
              
              <!-- Team A -->
              <div class="col-span-3 flex flex-col items-center text-center space-y-2">
                <div class="w-14 h-14 rounded-xl bg-slate-950 p-1.5 border border-slate-800 flex items-center justify-center overflow-hidden">
                  <img src="${teamALogo}" class="max-w-full max-h-full object-contain" alt="${escapeHtml(match.teamA.name)}">
                </div>
                <span class="text-sm font-semibold text-slate-200 line-clamp-1 w-full">${escapeHtml(match.teamA.name)}</span>
                <span class="text-3xl font-black text-indigo-400">${match.teamA.score}</span>
              </div>

              <!-- VS -->
              <div class="col-span-1 text-center">
                <span class="text-xs font-extrabold text-slate-600 bg-slate-800/60 px-1.5 py-0.5 rounded">VS</span>
              </div>

              <!-- Team B -->
              <div class="col-span-3 flex flex-col items-center text-center space-y-2">
                <div class="w-14 h-14 rounded-xl bg-slate-950 p-1.5 border border-slate-800 flex items-center justify-center overflow-hidden">
                  <img src="${teamBLogo}" class="max-w-full max-h-full object-contain" alt="${escapeHtml(match.teamB.name)}">
                </div>
                <span class="text-sm font-semibold text-slate-200 line-clamp-1 w-full">${escapeHtml(match.teamB.name)}</span>
                <span class="text-3xl font-black text-red-400">${match.teamB.score}</span>
              </div>

            </div>

            <!-- Card Action Footer -->
            <div class="pt-2 border-t border-slate-800/80 flex items-center justify-between gap-2">
              
              <button onclick="openMatchFullscreen('${match.id}')" class="px-3 py-1.5 bg-slate-800 hover:bg-slate-700 text-slate-300 rounded-xl text-xs font-medium transition-all flex items-center gap-1.5">
                <i class="fa-solid fa-expand text-slate-400"></i>
                <span>ดูแบบเต็มจอ</span>
              </button>

              ${isAdminLoggedIn ? `
                <div class="flex items-center gap-1">
                  <button onclick="openQuickScoreModal('${match.id}')" class="px-2.5 py-1.5 bg-indigo-600/20 hover:bg-indigo-600/30 text-indigo-300 border border-indigo-500/30 rounded-xl text-xs font-medium transition-all flex items-center gap-1">
                    <i class="fa-solid fa-pen-to-square"></i>
                    <span>แก้คะแนน</span>
                  </button>
                  <button onclick="deleteMatch('${match.id}')" class="p-1.5 bg-red-500/10 hover:bg-red-500/20 text-red-400 rounded-xl text-xs transition-all">
                    <i class="fa-solid fa-trash-can"></i>
                  </button>
                </div>
              ` : ''}

            </div>

          </div>
        `;
      }).join('');
    };

    // Render Total Leaderboard & Medal Possibilities
    window.renderLeaderboard = function() {
      const tbody = document.getElementById('leaderboard-tbody');
      const emptyState = document.getElementById('leaderboard-empty');
      if (!tbody) return;

      // Aggregating team statistics across all matches
      const teamStats = {};

      matchesData.forEach(match => {
        const teamA = match.teamA.name;
        const teamB = match.teamB.name;
        const scoreA = parseInt(match.teamA.score) || 0;
        const scoreB = parseInt(match.teamB.score) || 0;

        // Ensure teams exist in stats dictionary
        if (!teamStats[teamA]) teamStats[teamA] = { name: teamA, logo: match.teamA.logo, played: 0, wins: 0, draws: 0, losses: 0, totalPoints: 0 };
        if (!teamStats[teamB]) teamStats[teamB] = { name: teamB, logo: match.teamB.logo, played: 0, wins: 0, draws: 0, losses: 0, totalPoints: 0 };

        // Calculate statistics if match is LIVE or FINISHED
        if (match.status === 'LIVE' || match.status === 'FINISHED') {
          teamStats[teamA].played += 1;
          teamStats[teamB].played += 1;

          teamStats[teamA].totalPoints += scoreA;
          teamStats[teamB].totalPoints += scoreB;

          if (scoreA > scoreB) {
            teamStats[teamA].wins += 1;
            teamStats[teamB].losses += 1;
          } else if (scoreB > scoreA) {
            teamStats[teamB].wins += 1;
            teamStats[teamA].losses += 1;
          } else {
            teamStats[teamA].draws += 1;
            teamStats[teamB].draws += 1;
          }
        }
      });

      const teamsArray = Object.values(teamStats);

      if (teamsArray.length === 0) {
        tbody.innerHTML = '';
        if (emptyState) emptyState.classList.remove('hidden');
        return;
      }

      if (emptyState) emptyState.classList.add('hidden');

      // Sort by Total Points (descending), then Wins (descending)
      teamsArray.sort((a, b) => b.totalPoints - a.totalPoints || b.wins - a.wins);

      tbody.innerHTML = teamsArray.map((t, idx) => {
        const rank = idx + 1;
        
        // Calculate Medal Projection based on thresholds
        let medalBadge = '';
        if (t.totalPoints >= medalThresholds.gold) {
          medalBadge = `<span class="px-2.5 py-1 bg-amber-500/20 text-amber-400 border border-amber-500/40 rounded-full text-xs font-bold inline-flex items-center gap-1"><i class="fa-solid fa-medal text-amber-400"></i> เหรียญทอง</span>`;
        } else if (t.totalPoints >= medalThresholds.silver) {
          medalBadge = `<span class="px-2.5 py-1 bg-slate-400/20 text-slate-300 border border-slate-400/40 rounded-full text-xs font-bold inline-flex items-center gap-1"><i class="fa-solid fa-medal text-slate-300"></i> เหรียญเงิน</span>`;
        } else if (t.totalPoints >= medalThresholds.bronze) {
          medalBadge = `<span class="px-2.5 py-1 bg-amber-700/20 text-amber-500 border border-amber-700/40 rounded-full text-xs font-bold inline-flex items-center gap-1"><i class="fa-solid fa-medal text-amber-600"></i> เหรียญทองแดง</span>`;
        } else {
          medalBadge = `<span class="px-2.5 py-1 bg-slate-800 text-slate-500 rounded-full text-xs font-medium">ไม่มีเหรียญ</span>`;
        }

        let rankBadgeClass = 'text-slate-400 font-medium';
        if (rank === 1) rankBadgeClass = 'w-7 h-7 mx-auto rounded-full bg-amber-500 text-slate-950 font-black flex items-center justify-center shadow-lg shadow-amber-500/30';
        else if (rank === 2) rankBadgeClass = 'w-7 h-7 mx-auto rounded-full bg-slate-300 text-slate-950 font-black flex items-center justify-center shadow-md';
        else if (rank === 3) rankBadgeClass = 'w-7 h-7 mx-auto rounded-full bg-amber-700 text-white font-black flex items-center justify-center shadow-md';

        return `
          <tr class="hover:bg-slate-800/40 transition-colors">
            <td class="py-3.5 px-4 text-center">
              <div class="${rankBadgeClass}">${rank}</div>
            </td>
            <td class="py-3.5 px-4 font-semibold text-slate-200 flex items-center gap-3">
              <img src="${t.logo || DEFAULT_LOGO}" class="w-8 h-8 rounded-lg object-contain bg-slate-950 p-1 border border-slate-800" alt="${escapeHtml(t.name)}">
              <span>${escapeHtml(t.name)}</span>
            </td>
            <td class="py-3.5 px-4 text-center text-slate-400">${t.played}</td>
            <td class="py-3.5 px-4 text-center text-emerald-400 font-medium">${t.wins}</td>
            <td class="py-3.5 px-4 text-center text-slate-400">${t.draws}</td>
            <td class="py-3.5 px-4 text-center text-red-400 font-medium">${t.losses}</td>
            <td class="py-3.5 px-4 text-center text-indigo-400 font-black text-base">${t.totalPoints}</td>
            <td class="py-3.5 px-4 text-center">${medalBadge}</td>
          </tr>
        `;
      }).join('');
    };

    function updateThresholdLabels() {
      const g = document.getElementById('thresh-gold-label');
      const s = document.getElementById('thresh-silver-label');
      const b = document.getElementById('thresh-bronze-label');
      if (g) g.innerText = medalThresholds.gold;
      if (s) s.innerText = medalThresholds.silver;
      if (b) b.innerText = medalThresholds.bronze;

      const gIn = document.getElementById('thresh-gold-input');
      const sIn = document.getElementById('thresh-silver-input');
      const bIn = document.getElementById('thresh-bronze-input');
      if (gIn) gIn.value = medalThresholds.gold;
      if (sIn) sIn.value = medalThresholds.silver;
      if (bIn) bIn.value = medalThresholds.bronze;
    }

    // Admin Login logic
    window.handleAdminAuth = function(e) {
      e.preventDefault();
      const pass = document.getElementById('admin-passcode-input').value;
      const errorElem = document.getElementById('admin-auth-error');

      if (pass === ADMIN_KEY) {
        isAdminLoggedIn = true;
        closeAdminModal();
        showToast("เข้าสู่ระบบแอดมินสำเร็จ!", "check");
        updateAdminUIState();
        switchTab('admin');
      } else {
        if (errorElem) errorElem.classList.remove('hidden');
      }
    };

    window.adminLogout = function() {
      isAdminLoggedIn = false;
      updateAdminUIState();
      switchTab('matches');
      showToast("ออกจากระบบผู้ดูแลเรียบร้อย", "info");
    };

    function updateAdminUIState() {
      const container = document.getElementById('admin-auth-container');
      const adminTabBtn = document.getElementById('tab-btn-admin');

      if (isAdminLoggedIn) {
        if (container) {
          container.innerHTML = `
            <div class="flex items-center gap-2">
              <span class="px-2.5 py-1 rounded-lg text-xs bg-emerald-500/20 text-emerald-300 border border-emerald-500/30 font-semibold flex items-center gap-1">
                <i class="fa-solid fa-user-check"></i> แอดมิน
              </span>
              <button onclick="adminLogout()" class="px-2.5 py-1 text-xs bg-slate-800 hover:bg-slate-700 text-slate-300 rounded-lg">
                ออก
              </button>
            </div>
          `;
        }
        if (adminTabBtn) adminTabBtn.classList.remove('hidden');
      } else {
        if (container) {
          container.innerHTML = `
            <button id="btn-admin-login-trigger" onclick="openAdminModal()" class="px-3.5 py-1.5 text-xs sm:text-sm font-medium bg-slate-800 hover:bg-slate-700 text-slate-200 border border-slate-700 rounded-lg transition-all flex items-center gap-2 shadow-sm">
              <i class="fa-solid fa-lock text-slate-400"></i>
              <span>เข้าสู่ระบบแอดมิน</span>
            </button>
          `;
        }
        if (adminTabBtn) adminTabBtn.classList.add('hidden');
      }

      renderMatches();
    }

    // File upload handler (Max 10MB check + base64 compression)
    window.handleLogoUpload = function(inputElem, previewId) {
      const file = inputElem.files[0];
      if (!file) return;

      // 10MB limit in bytes = 10 * 1024 * 1024 = 10,485,760
      const MAX_SIZE = 10 * 1024 * 1024;
      if (file.size > MAX_SIZE) {
        showToast("ขนาดไฟล์ต้องไม่เกิน 10MB!", "exclamation");
        inputElem.value = "";
        return;
      }

      const reader = new FileReader();
      reader.onload = function(e) {
        const base64 = e.target.result;
        const targetHiddenInput = previewId.includes('teamA') ? 'teamA-logo-data' : 'teamB-logo-data';
        document.getElementById(targetHiddenInput).value = base64;

        const previewDiv = document.getElementById(previewId);
        if (previewDiv) {
          previewDiv.querySelector('img').src = base64;
          previewDiv.classList.remove('hidden');
        }
      };
      reader.readAsDataURL(file);
    };

    // Create Sport Handler
    window.handleCreateSport = async function(e) {
      e.preventDefault();
      if (!isAdminLoggedIn) return;

      const input = document.getElementById('sport-name-input');
      const name = input.value.trim();
      if (!name) return;

      try {
        const sportsCol = collection(db, 'artifacts', appId, 'public', 'data', 'sports');
        await addDoc(sportsCol, {
          name: name,
          createdAt: Date.now()
        });
        input.value = '';
        showToast("เพิ่มประเภทกีฬาสำเร็จ", "check");
      } catch (err) {
        console.error("Error creating sport:", err);
        showToast("เกิดข้อผิดพลาดในการบันทึก", "exclamation");
      }
    };

    // Render sports list in Admin panel
    function renderAdminSportsList() {
      const container = document.getElementById('admin-sports-list');
      if (!container) return;

      if (sportsData.length === 0) {
        container.innerHTML = '<span class="text-xs text-slate-500">ยังไม่มีชนิดกีฬา</span>';
        return;
      }

      container.innerHTML = sportsData.map(s => `
        <span class="inline-flex items-center gap-1.5 px-2.5 py-1 bg-slate-950 border border-slate-800 text-slate-300 rounded-lg text-xs font-medium">
          ${escapeHtml(s.name)}
          ${isAdminLoggedIn ? `
            <button onclick="deleteSport('${s.id}')" class="text-slate-500 hover:text-red-400 ml-1">
              <i class="fa-solid fa-xmark"></i>
            </button>
          ` : ''}
        </span>
      `).join('');
    }

    function populateMatchSportSelect() {
      const select = document.getElementById('match-sport-select');
      if (!select) return;

      const currentVal = select.value;
      select.innerHTML = '<option value="">-- กรุณาเลือกกีฬา --</option>' + 
        sportsData.map(s => `<option value="${s.id}">${escapeHtml(s.name)}</option>`).join('');
      select.value = currentVal;
    }

    window.deleteSport = async function(sportId) {
      if (!isAdminLoggedIn) return;
      try {
        const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'sports', sportId);
        await deleteDoc(docRef);
        showToast("ลบประเภทกีฬาสำเร็จ", "check");
      } catch (err) {
        console.error("Error deleting sport:", err);
      }
    };

    // Create Match Handler
    window.handleCreateMatch = async function(e) {
      e.preventDefault();
      if (!isAdminLoggedIn) return;

      const sportSelect = document.getElementById('match-sport-select');
      const sportId = sportSelect.value;
      const selectedSportObj = sportsData.find(s => s.id === sportId);
      const sportName = selectedSportObj ? selectedSportObj.name : 'กีฬาทั่วไป';

      const matchTitle = document.getElementById('match-title-input').value.trim();
      
      const teamAName = document.getElementById('teamA-name-input').value.trim();
      const teamALogo = document.getElementById('teamA-logo-data').value || DEFAULT_LOGO;
      const teamAScore = parseInt(document.getElementById('teamA-score-init').value) || 0;

      const teamBName = document.getElementById('teamB-name-input').value.trim();
      const teamBLogo = document.getElementById('teamB-logo-data').value || DEFAULT_LOGO;
      const teamBScore = parseInt(document.getElementById('teamB-score-init').value) || 0;

      const status = document.getElementById('match-status-select').value;

      try {
        const matchesCol = collection(db, 'artifacts', appId, 'public', 'data', 'matches');
        await addDoc(matchesCol, {
          sportId,
          sportName,
          title: matchTitle,
          status,
          teamA: { name: teamAName, logo: teamALogo, score: teamAScore },
          teamB: { name: teamBName, logo: teamBLogo, score: teamBScore },
          createdAt: Date.now()
        });

        // Reset form
        document.getElementById('form-create-match').reset();
        document.getElementById('teamA-logo-data').value = '';
        document.getElementById('teamB-logo-data').value = '';
        document.getElementById('teamA-preview').classList.add('hidden');
        document.getElementById('teamB-preview').classList.add('hidden');

        showToast("สร้างแมตช์การแข่งขันเรียบร้อย!", "check");
      } catch (err) {
        console.error("Error creating match:", err);
        showToast("ไม่สามารถสร้างแมตช์ได้", "exclamation");
      }
    };

    window.deleteMatch = async function(matchId) {
      if (!isAdminLoggedIn) return;
      try {
        const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'matches', matchId);
        await deleteDoc(docRef);
        showToast("ลบแมตช์การแข่งขันเรียบร้อย", "check");
      } catch (err) {
        console.error("Error deleting match:", err);
      }
    };

    // Save Medal Criteria Thresholds
    window.handleSaveThresholds = async function(e) {
      e.preventDefault();
      if (!isAdminLoggedIn) return;

      const gold = parseInt(document.getElementById('thresh-gold-input').value) || 0;
      const silver = parseInt(document.getElementById('thresh-silver-input').value) || 0;
      const bronze = parseInt(document.getElementById('thresh-bronze-input').value) || 0;

      try {
        const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'settings', 'medal_rules');
        await setDoc(docRef, { gold, silver, bronze, updatedAt: Date.now() });
        showToast("บันทึกเกณฑ์คำนวณเหรียญสำเร็จ", "check");
      } catch (err) {
        console.error("Error saving thresholds:", err);
      }
    };

    // Quick score edit modal
    window.openQuickScoreModal = function(matchId) {
      const match = matchesData.find(m => m.id === matchId);
      if (!match) return;

      document.getElementById('qs-match-id').value = match.id;
      document.getElementById('qs-status').value = match.status;
      document.getElementById('qs-teamA-label').innerText = match.teamA.name;
      document.getElementById('qs-teamA-score').value = match.teamA.score;
      document.getElementById('qs-teamB-label').innerText = match.teamB.name;
      document.getElementById('qs-teamB-score').value = match.teamB.score;

      document.getElementById('modal-quick-score').classList.remove('hidden');
    };

    window.closeQuickScoreModal = function() {
      document.getElementById('modal-quick-score').classList.add('hidden');
    };

    window.handleSaveScoreUpdate = async function(e) {
      e.preventDefault();
      if (!isAdminLoggedIn) return;

      const matchId = document.getElementById('qs-match-id').value;
      const status = document.getElementById('qs-status').value;
      const scoreA = parseInt(document.getElementById('qs-teamA-score').value) || 0;
      const scoreB = parseInt(document.getElementById('qs-teamB-score').value) || 0;

      try {
        const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'matches', matchId);
        await updateDoc(docRef, {
          status,
          'teamA.score': scoreA,
          'teamB.score': scoreB,
          updatedAt: Date.now()
        });
        closeQuickScoreModal();
        showToast("อัปเดตผลคะแนนเรียบร้อย!", "check");
      } catch (err) {
        console.error("Error updating score:", err);
      }
    };

    // Fullscreen View Modal
    let activeFullscreenMatch = null;
    window.openMatchFullscreen = function(matchId) {
      const match = matchesData.find(m => m.id === matchId);
      if (!match) return;

      activeFullscreenMatch = match;

      document.getElementById('fs-sport-badge').innerText = match.sportName || 'กีฬา';
      
      const statusBadge = document.getElementById('fs-status-badge');
      if (match.status === 'LIVE') {
        statusBadge.className = 'text-xs font-bold px-2.5 py-1 rounded-full bg-red-500/20 text-red-400 border border-red-500/30 pulse-live';
        statusBadge.innerText = '• กำลังแข่ง LIVE';
      } else if (match.status === 'FINISHED') {
        statusBadge.className = 'text-xs font-bold px-2.5 py-1 rounded-full bg-emerald-500/20 text-emerald-400 border border-emerald-500/30';
        statusBadge.innerText = 'จบการแข่งขัน';
      } else {
        statusBadge.className = 'text-xs font-bold px-2.5 py-1 rounded-full bg-slate-800 text-slate-400 border border-slate-700';
        statusBadge.innerText = 'ยังไม่เริ่ม';
      }

      document.getElementById('fs-match-title').innerText = match.title || 'แมตช์การแข่งขัน';

      document.getElementById('fs-teamA-logo').src = match.teamA.logo || DEFAULT_LOGO;
      document.getElementById('fs-teamA-name').innerText = match.teamA.name;
      document.getElementById('fs-teamA-score').innerText = match.teamA.score;

      document.getElementById('fs-teamB-logo').src = match.teamB.logo || DEFAULT_LOGO;
      document.getElementById('fs-teamB-name').innerText = match.teamB.name;
      document.getElementById('fs-teamB-score').innerText = match.teamB.score;

      const now = new Date();
      document.getElementById('fs-timestamp').innerText = `อัปเดตเมื่อ: ${now.toLocaleTimeString('th-TH')}`;

      document.getElementById('modal-match-fullscreen').classList.remove('hidden');
    };

    window.closeMatchFullscreenModal = function() {
      document.getElementById('modal-match-fullscreen').classList.add('hidden');
      activeFullscreenMatch = null;
    };

    // Download Scorecard Graphic using Canvas
    window.downloadMatchScorecard = function() {
      if (!activeFullscreenMatch) return;

      const canvas = document.createElement('canvas');
      canvas.width = 800;
      canvas.height = 500;
      const ctx = canvas.getContext('2d');

      // Fill Background
      const gradient = ctx.createLinearGradient(0, 0, 800, 500);
      gradient.addColorStop(0, '#020617');
      gradient.addColorStop(0.5, '#0f172a');
      gradient.addColorStop(1, '#1e1b4b');
      ctx.fillStyle = gradient;
      ctx.fillRect(0, 0, 800, 500);

      // Card Header
      ctx.fillStyle = '#6366f1';
      ctx.font = 'bold 24px Kanit, sans-serif';
      ctx.fillText(activeFullscreenMatch.sportName || 'LIVE SCORE', 50, 60);

      ctx.fillStyle = '#94a3b8';
      ctx.font = '18px Kanit, sans-serif';
      ctx.fillText(activeFullscreenMatch.title || '', 50, 90);

      // Draw Team Names & Scores
      ctx.textAlign = 'center';
      
      // Team A
      ctx.fillStyle = '#ffffff';
      ctx.font = 'bold 36px Kanit, sans-serif';
      ctx.fillText(activeFullscreenMatch.teamA.name, 220, 220);
      ctx.fillStyle = '#818cf8';
      ctx.font = 'bold 90px Kanit, sans-serif';
      ctx.fillText(activeFullscreenMatch.teamA.score, 220, 320);

      // VS
      ctx.fillStyle = '#475569';
      ctx.font = 'bold 32px Kanit, sans-serif';
      ctx.fillText('VS', 400, 260);

      // Team B
      ctx.fillStyle = '#ffffff';
      ctx.font = 'bold 36px Kanit, sans-serif';
      ctx.fillText(activeFullscreenMatch.teamB.name, 580, 220);
      ctx.fillStyle = '#f87171';
      ctx.font = 'bold 90px Kanit, sans-serif';
      ctx.fillText(activeFullscreenMatch.teamB.score, 580, 320);

      // Footer
      ctx.textAlign = 'left';
      ctx.fillStyle = '#64748b';
      ctx.font = '16px Kanit, sans-serif';
      ctx.fillText('LIVE SCORE & LEADERBOARD SYSTEM', 50, 460);

      // Trigger Download
      const link = document.createElement('a');
      link.download = `match-${activeFullscreenMatch.teamA.name}-vs-${activeFullscreenMatch.teamB.name}.png`;
      link.href = canvas.toDataURL('image/png');
      link.click();

      showToast("ดาวน์โหลดรูปภาพสำเร็จ!", "download");
    };

    // Navigation Tabs Switcher
    window.switchTab = function(tabName) {
      const viewMatches = document.getElementById('view-matches');
      const viewLeaderboard = document.getElementById('view-leaderboard');
      const viewAdmin = document.getElementById('view-admin');

      const btnMatches = document.getElementById('tab-btn-matches');
      const btnLeaderboard = document.getElementById('tab-btn-leaderboard');
      const btnAdmin = document.getElementById('tab-btn-admin');

      // Hide all views
      viewMatches.classList.add('hidden');
      viewLeaderboard.classList.add('hidden');
      viewAdmin.classList.add('hidden');

      // Reset button styles
      btnMatches.className = "px-4 py-2 text-sm font-medium rounded-lg transition-all flex items-center gap-2 text-slate-400 hover:bg-slate-800 hover:text-slate-200";
      btnLeaderboard.className = "px-4 py-2 text-sm font-medium rounded-lg transition-all flex items-center gap-2 text-slate-400 hover:bg-slate-800 hover:text-slate-200";
      btnAdmin.className = "px-4 py-2 text-sm font-medium rounded-lg transition-all flex items-center gap-2 text-slate-400 hover:bg-slate-800 hover:text-slate-200";

      if (tabName === 'matches') {
        viewMatches.classList.remove('hidden');
        btnMatches.className = "px-4 py-2 text-sm font-medium rounded-lg transition-all flex items-center gap-2 bg-indigo-600 text-white shadow-md shadow-indigo-600/30";
      } else if (tabName === 'leaderboard') {
        viewLeaderboard.classList.remove('hidden');
        btnLeaderboard.className = "px-4 py-2 text-sm font-medium rounded-lg transition-all flex items-center gap-2 bg-indigo-600 text-white shadow-md shadow-indigo-600/30";
        renderLeaderboard();
      } else if (tabName === 'admin') {
        viewAdmin.classList.remove('hidden');
        btnAdmin.className = "px-4 py-2 text-sm font-medium rounded-lg transition-all flex items-center gap-2 bg-indigo-600 text-white shadow-md shadow-indigo-600/30";
      }
    };

    // Admin Modal triggers
    window.openAdminModal = function() {
      document.getElementById('modal-admin-login').classList.remove('hidden');
      document.getElementById('admin-passcode-input').value = '';
      document.getElementById('admin-auth-error').classList.add('hidden');
    };

    window.closeAdminModal = function() {
      document.getElementById('modal-admin-login').classList.add('hidden');
    };

    window.togglePasswordVisibility = function() {
      const input = document.getElementById('admin-passcode-input');
      const icon = document.getElementById('eye-icon');
      if (input.type === 'password') {
        input.type = 'text';
        icon.className = 'fa-solid fa-eye-slash';
      } else {
        input.type = 'password';
        icon.className = 'fa-solid fa-eye';
      }
    };

    // Toast Notification logic
    function showToast(text, iconType = 'check') {
      const toast = document.getElementById('toast-message');
      const toastText = document.getElementById('toast-text');
      const toastIcon = document.getElementById('toast-icon');

      if (!toast || !toastText) return;

      toastText.innerText = text;
      toastIcon.className = `fa-solid fa-circle-${iconType} text-lg`;

      toast.classList.remove('hidden');
      setTimeout(() => {
        toast.classList.add('hidden');
      }, 3000);
    }

    // HTML Sanitizer
    function escapeHtml(str) {
      if (!str) return '';
      return str.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#039;");
    }

    // Initialize App on window load
    window.onload = function() {
      updateAdminUIState(); // ซ่อนเมนูการจัดการสำหรับผู้ชมทั่วไปทันทีตั้งแต่โหลดหน้าเว็บ
      initFirebase();
    };
  </script>
</body>
</html>
