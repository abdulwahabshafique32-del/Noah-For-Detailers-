<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Tactical Lineup & Position Performance Predictor</title>
  
  <!-- Google Fonts: Inter & Outfit -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&family=Outfit:wght@400;500;600;700;800;900&family=JetBrains+Mono:wght@400;600;700;800&display=swap" rel="stylesheet">
  
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          fontFamily: {
            sans: ['Inter', 'sans-serif'],
            display: ['Outfit', 'sans-serif'],
            mono: ['JetBrains Mono', 'monospace'],
          },
          colors: {
            pitch: {
              top: '#347551',
              mid: '#286343',
              dark: '#1b4a32',
              line: 'rgba(255, 255, 255, 0.45)',
            }
          }
        }
      }
    }
  </script>

  <!-- Confetti library for game wins -->
  <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.9.4/dist/confetti.browser.min.js"></script>

  <style>
    body {
      background-color: #0b0f19;
      color: #f1f5f9;
      font-family: 'Inter', sans-serif;
      overflow-x: hidden;
    }

    /* Pitch Grass Texture & Markings */
    .pitch-container {
      background: linear-gradient(180deg, #377a54 0%, #296846 50%, #1c4e34 100%);
      box-shadow: inset 0 0 40px rgba(0,0,0,0.5), 0 20px 40px rgba(0,0,0,0.6);
      position: relative;
    }

    .pitch-stripes {
      background-image: repeating-linear-gradient(
        0deg,
        rgba(0, 0, 0, 0.08),
        rgba(0, 0, 0, 0.08) 38px,
        transparent 38px,
        transparent 76px
      );
    }

    /* Custom Scrollbar */
    ::-webkit-scrollbar {
      width: 6px;
      height: 6px;
    }
    ::-webkit-scrollbar-track {
      background: #0f172a;
    }
    ::-webkit-scrollbar-thumb {
      background: #334155;
      border-radius: 3px;
    }
    ::-webkit-scrollbar-thumb:hover {
      background: #475569;
    }

    /* Pulse Glow */
    @keyframes pulseGlow {
      0%, 100% { opacity: 0.6; transform: scale(1); }
      50% { opacity: 1; transform: scale(1.05); }
    }
    .pulse-glow {
      animation: pulseGlow 2.5s infinite ease-in-out;
    }

    .card-modal-blur {
      backdrop-filter: blur(16px);
      -webkit-backdrop-filter: blur(16px);
    }

    /* Sofascore Player Node Tombstone Card */
    .player-card-tombstone {
      background: linear-gradient(180deg, #8fa3b5 0%, #68798a 100%);
      border: 1.5px solid rgba(255, 255, 255, 0.35);
      border-radius: 14px 14px 6px 6px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.45);
      position: relative;
    }
  </style>
</head>
<body class="bg-[#0c101a] text-slate-100 min-h-screen flex flex-col items-center">

  <!-- Main Container (Easily embeddable in WordPress) -->
  <div id="tactical-app-root" class="w-full max-w-5xl mx-auto px-2 sm:px-4 py-3 sm:py-6">
    
    <!-- Top Action Banner -->
    <div class="w-full bg-[#131a29]/95 border border-slate-800 rounded-2xl p-3 mb-4 shadow-xl backdrop-blur flex flex-wrap items-center justify-between gap-3">
      <div class="flex items-center space-x-2.5">
        <span class="flex items-center text-xs font-bold text-emerald-400 bg-emerald-500/10 px-2.5 py-1 rounded-full border border-emerald-500/30">
          <span class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse mr-1.5"></span>
          LIVE 74'
        </span>
        <span class="text-xs text-slate-400 font-medium hidden sm:inline">UEFA Champions League • Quarter-Final 2nd Leg</span>
      </div>

      <!-- Quick Action Buttons -->
      <div class="flex items-center flex-wrap gap-2">
        <button id="btn-open-predictor" class="px-3.5 py-1.5 bg-gradient-to-r from-cyan-600 to-blue-600 hover:from-cyan-500 hover:to-blue-500 text-white rounded-xl text-xs font-bold shadow-md shadow-cyan-900/40 transition hover:scale-105 flex items-center space-x-1.5 cursor-pointer">
          <span>✨</span>
          <span>AI Position Predictor</span>
        </button>

        <button id="btn-open-game" class="px-3.5 py-1.5 bg-gradient-to-r from-amber-600 to-orange-600 hover:from-amber-500 hover:to-orange-500 text-white rounded-xl text-xs font-bold shadow-md shadow-amber-900/40 transition hover:scale-105 flex items-center space-x-1.5 cursor-pointer">
          <span>🎮</span>
          <span>Manager Challenge</span>
        </button>

        <button id="btn-open-sim" class="px-3.5 py-1.5 bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-500 hover:to-teal-500 text-white rounded-xl text-xs font-bold shadow-md shadow-emerald-900/40 transition hover:scale-105 flex items-center space-x-1.5 cursor-pointer">
          <span>⚡</span>
          <span>Simulate 15'</span>
        </button>

        <button id="btn-wordpress-code" class="px-2.5 py-1.5 bg-slate-800 hover:bg-slate-700 text-slate-300 hover:text-white rounded-xl text-xs font-semibold border border-slate-700 transition flex items-center space-x-1 cursor-pointer" title="Get WordPress Embed Code">
          <span>📋</span>
          <span class="hidden md:inline">WordPress Embed</span>
        </button>

        <button id="btn-toggle-sound" class="p-1.5 bg-slate-800 hover:bg-slate-700 text-cyan-400 rounded-xl text-sm border border-slate-700 transition cursor-pointer" title="Toggle Sound">
          🔊
        </button>

        <button id="btn-reset-lineup" class="p-1.5 bg-slate-800 hover:bg-slate-700 text-slate-300 rounded-xl text-sm border border-slate-700 transition cursor-pointer" title="Reset Starting Lineups">
          🔄
        </button>
      </div>
    </div>

    <!-- Match Scoreboard Header -->
    <div class="w-full bg-[#131926] border border-slate-800 rounded-3xl p-4 sm:p-5 mb-4 shadow-2xl relative overflow-hidden">
      <div class="flex items-center justify-between">
        
        <!-- Home: Bayern -->
        <div class="flex items-center space-x-3 sm:space-x-4 flex-1">
          <div class="w-12 h-12 sm:w-14 sm:h-14 rounded-full bg-slate-900 border-2 border-red-500/60 p-1 flex items-center justify-center shadow-lg shadow-red-900/40 shrink-0">
            <img src="https://images.fotmob.com/image_resources/logo/teamlogo/9823.png" alt="Bayern" class="w-full h-full object-contain" onerror="this.onerror=null; this.src='https://ui-avatars.com/api/?name=Bayern&background=dc2626&color=fff';" />
          </div>
          <div>
            <div class="flex items-center space-x-2 flex-wrap">
              <h2 class="text-base sm:text-xl font-black text-white font-display">Bayern München</h2>
              <span class="text-[11px] font-bold px-2 py-0.5 rounded-full bg-red-950/80 text-red-300 border border-red-800 font-mono">4-2-3-1</span>
            </div>
            <p class="text-xs text-slate-400 mt-0.5">Manager: <span class="text-slate-300 font-medium">Vincent Kompany</span></p>
          </div>
        </div>

        <!-- Center Score -->
        <div class="px-4 sm:px-6 py-2 bg-slate-950/90 rounded-2xl border border-slate-800 flex flex-col items-center mx-2 shrink-0">
          <div class="flex items-center space-x-3 text-2xl sm:text-3xl font-black text-white font-mono tracking-widest">
            <span id="score-home">1</span>
            <span class="text-slate-600 text-lg font-light">-</span>
            <span id="score-away">1</span>
          </div>
          <span class="text-[10px] sm:text-[11px] font-bold text-emerald-400 uppercase tracking-wider mt-0.5">74' Minute</span>
        </div>

        <!-- Away: PSG -->
        <div class="flex items-center justify-end space-x-3 sm:space-x-4 flex-1 text-right">
          <div>
            <div class="flex items-center justify-end space-x-2 flex-wrap">
              <span class="text-[11px] font-bold px-2 py-0.5 rounded-full bg-blue-950/80 text-blue-300 border border-blue-800 font-mono">4-3-3</span>
              <h2 class="text-base sm:text-xl font-black text-white font-display">Paris Saint-Germain</h2>
            </div>
            <p class="text-xs text-slate-400 mt-0.5">Manager: <span class="text-slate-300 font-medium">Luis Enrique</span></p>
          </div>
          <div class="w-12 h-12 sm:w-14 sm:h-14 rounded-full bg-slate-900 border-2 border-blue-500/60 p-1 flex items-center justify-center shadow-lg shadow-blue-900/40 shrink-0">
            <img src="https://images.fotmob.com/image_resources/logo/teamlogo/9847.png" alt="PSG" class="w-full h-full object-contain" onerror="this.onerror=null; this.src='https://ui-avatars.com/api/?name=PSG&background=1d4ed8&color=fff';" />
          </div>
        </div>

      </div>

      <!-- Sofascore View Switcher Tabs (Performance | Age | Country | Heatmap | Chemistry) -->
      <div class="flex items-center justify-center space-x-1 sm:space-x-3 border-t border-slate-800/80 mt-4 pt-3 overflow-x-auto">
        <button data-tab="performance" class="tab-btn active-tab px-4 py-1.5 rounded-xl text-xs font-bold transition flex items-center space-x-1.5 bg-cyan-600 text-white shadow-md cursor-pointer">
          <span>📊</span>
          <span>Performance</span>
        </button>
        <button data-tab="age" class="tab-btn px-4 py-1.5 rounded-xl text-xs font-bold text-slate-400 hover:text-slate-200 hover:bg-slate-800/80 transition flex items-center space-x-1.5 cursor-pointer">
          <span>📅</span>
          <span>Age</span>
        </button>
        <button data-tab="country" class="tab-btn px-4 py-1.5 rounded-xl text-xs font-bold text-slate-400 hover:text-slate-200 hover:bg-slate-800/80 transition flex items-center space-x-1.5 cursor-pointer">
          <span>🌍</span>
          <span>Country</span>
        </button>
        <button data-tab="heatmap" class="tab-btn px-4 py-1.5 rounded-xl text-xs font-bold text-slate-400 hover:text-slate-200 hover:bg-slate-800/80 transition flex items-center space-x-1.5 cursor-pointer">
          <span>🔥</span>
          <span>Heatmap</span>
        </button>
        <button data-tab="chemistry" class="tab-btn px-4 py-1.5 rounded-xl text-xs font-bold text-slate-400 hover:text-slate-200 hover:bg-slate-800/80 transition flex items-center space-x-1.5 cursor-pointer">
          <span>🔗</span>
          <span>Passing & Links</span>
        </button>
      </div>
    </div>

    <!-- Main Grid: Tactical Pitch & Side Bench / Quick Stats -->
    <div class="grid grid-cols-1 lg:grid-cols-12 gap-5 items-start">
      
      <!-- Center Soccer Pitch (Exact Visuals as in Reference Image) -->
      <div class="lg:col-span-8 flex flex-col items-center">
        
        <div class="w-full max-w-[540px] rounded-3xl overflow-hidden shadow-2xl border border-emerald-800/50 bg-emerald-950 relative">
          
          <!-- Top Pitch Team Tag: Bayern 4-2-3-1 -->
          <div class="bg-[#182030]/95 px-4 py-2.5 flex items-center justify-between border-b border-slate-700/80 relative z-20 backdrop-blur">
            <div class="flex items-center space-x-2">
              <div class="w-5 h-5 rounded-full bg-red-600 flex items-center justify-center text-[10px] font-bold text-white shadow">B</div>
              <span class="font-bold text-xs sm:text-sm text-white">Bayern</span>
            </div>
            <span class="text-[11px] px-2.5 py-0.5 rounded-full bg-emerald-600 text-emerald-50 font-bold font-mono border border-emerald-400/40">4-2-3-1</span>
          </div>

          <!-- Soccer Pitch Graphic -->
          <div id="soccer-pitch" class="pitch-container relative w-full aspect-[9/13.8] pitch-stripes overflow-hidden select-none">
            
            <!-- SVG Pitch Markings -->
            <svg class="absolute inset-0 w-full h-full pointer-events-none stroke-white/50 stroke-[1.5]" fill="none">
              <!-- Outer Line -->
              <rect x="4%" y="2.5%" width="92%" height="95%" rx="6" />
              <!-- Halfway Line -->
              <line x1="4%" y1="50%" x2="96%" y2="50%" />
              <!-- Center Circle -->
              <circle cx="50%" cy="50%" r="13%" />
              <circle cx="50%" cy="50%" r="1.5" fill="white" fill-opacity="0.8" />
              <!-- Top Box (Bayern) -->
              <rect x="22%" y="2.5%" width="56%" height="16%" />
              <rect x="36%" y="2.5%" width="28%" height="6%" />
              <circle cx="50%" cy="13.5%" r="1.5" fill="white" fill-opacity="0.8" />
              <path d="M 39% 18.5% A 9 9 0 0 0 61% 18.5%" />
              <!-- Bottom Box (PSG) -->
              <rect x="22%" y="81.5%" width="56%" height="16%" />
              <rect x="36%" y="91.5%" width="28%" height="6%" />
              <circle cx="50%" cy="86.5%" r="1.5" fill="white" fill-opacity="0.8" />
              <path d="M 39% 81.5% A 9 9 0 0 1 61% 81.5%" />
              <!-- Corner Arcs -->
              <path d="M 4% 4% A 12 12 0 0 0 6% 2.5%" />
              <path d="M 96% 4% A 12 12 0 0 1 94% 2.5%" />
              <path d="M 4% 96% A 12 12 0 0 1 6% 97.5%" />
              <path d="M 96% 96% A 12 12 0 0 0 94% 97.5%" />
            </svg>

            <!-- Interactive Chemistry Link Overlay -->
            <svg id="chemistry-layer" class="absolute inset-0 w-full h-full pointer-events-none z-10 hidden">
              <!-- Dynamic links injected via JS -->
            </svg>

            <!-- Heatmap Layer -->
            <div id="heatmap-layer" class="absolute inset-0 pointer-events-none z-10 mix-blend-screen opacity-75 hidden">
              <div class="absolute top-[42%] left-[44%] w-32 h-32 rounded-full bg-red-500/50 blur-2xl animate-pulse"></div>
              <div class="absolute top-[20%] left-[16%] w-24 h-24 rounded-full bg-amber-500/40 blur-xl"></div>
              <div class="absolute top-[60%] left-[45%] w-28 h-28 rounded-full bg-emerald-400/40 blur-2xl"></div>
              <div class="absolute top-[62%] left-[18%] w-32 h-32 rounded-full bg-red-500/50 blur-2xl"></div>
              <div class="absolute top-[74%] left-[48%] w-36 h-36 rounded-full bg-amber-400/40 blur-2xl"></div>
            </div>

            <!-- Player Nodes Container (Rendered dynamically) -->
            <div id="pitch-players-container" class="absolute inset-0 z-20"></div>

          </div>

          <!-- Bottom Pitch Team Tag: PSG 4-3-3 -->
          <div class="bg-[#182030]/95 px-4 py-2.5 flex items-center justify-between border-t border-slate-700/80 relative z-20 backdrop-blur">
            <div class="flex items-center space-x-2">
              <div class="w-5 h-5 rounded-full bg-blue-700 flex items-center justify-center text-[10px] font-bold text-white shadow">P</div>
              <span class="font-bold text-xs sm:text-sm text-white">Paris Saint-Germain</span>
            </div>
            <span class="text-[11px] px-2.5 py-0.5 rounded-full bg-blue-600 text-blue-50 font-bold font-mono border border-blue-400/40">4-3-3</span>
          </div>

        </div>

        <p class="text-xs text-slate-400 mt-2.5 text-center flex items-center space-x-1">
          <span>💡</span>
          <span>Click any player for Sofascore deep stats, or drag & drop to predict position ratings!</span>
        </p>

      </div>

      <!-- Right Column: Interactive Bench & Fast Prediction Sidebar -->
      <div class="lg:col-span-4 space-y-4">
        
        <!-- Live Quick Predictor Widget -->
        <div class="bg-[#131926] border border-slate-800 rounded-3xl p-4 shadow-xl">
          <div class="flex items-center justify-between mb-3 border-b border-slate-800 pb-2">
            <div class="flex items-center space-x-2">
              <span class="text-cyan-400 text-base">🔮</span>
              <h3 class="text-sm font-bold text-white font-display">Instant Position Predictor</h3>
            </div>
            <span class="text-[10px] px-2 py-0.5 rounded-full bg-cyan-500/10 text-cyan-300 font-mono">Live AI</span>
          </div>

          <div class="space-y-3">
            <div>
              <label class="block text-[11px] font-semibold text-slate-400 uppercase tracking-wider mb-1">Select Player</label>
              <select id="quick-player-select" class="w-full bg-slate-950 border border-slate-700 rounded-xl px-3 py-2 text-xs text-white font-medium focus:outline-none focus:border-cyan-400"></select>
            </div>

            <div>
              <label class="block text-[11px] font-semibold text-slate-400 uppercase tracking-wider mb-1">Test In Position</label>
              <div id="quick-position-chips" class="grid grid-cols-5 gap-1 text-[11px] font-mono"></div>
            </div>

            <!-- Prediction Result Mini-Card -->
            <div id="quick-prediction-card" class="bg-gradient-to-b from-slate-900 to-slate-950 border border-slate-800 rounded-2xl p-3 shadow-inner">
              <div class="flex items-center justify-between mb-2">
                <span class="text-xs text-slate-300">Predicted Match Rating</span>
                <span id="quick-predicted-rating" class="px-2.5 py-0.5 rounded-full text-xs font-black font-mono bg-emerald-600 text-white shadow">8.2 ⭐</span>
              </div>
              <div class="flex items-center justify-between text-[11px] text-slate-400 mb-1">
                <span>Role Suitability:</span>
                <span id="quick-suitability" class="font-bold text-cyan-400 font-mono">92%</span>
              </div>
              <div class="w-full h-1.5 bg-slate-800 rounded-full overflow-hidden mb-2">
                <div id="quick-suitability-bar" class="h-full bg-gradient-to-r from-cyan-500 to-emerald-400" style="width: 92%"></div>
              </div>
              <p id="quick-insight" class="text-[11px] text-slate-300 italic leading-snug line-clamp-2">
                "Harry Kane as CAM generates elite vision (90) and pinpoint distribution behind speed runners."
              </p>
              <button id="btn-apply-quick-pos" class="w-full mt-2.5 py-2 bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-500 hover:to-teal-500 text-white text-xs font-bold rounded-xl shadow-md transition flex items-center justify-center space-x-1.5 cursor-pointer">
                <span>✨ Apply to Pitch</span>
              </button>
            </div>
          </div>
        </div>

        <!-- Substitutes Bench Drawer -->
        <div class="bg-[#131926] border border-slate-800 rounded-3xl p-4 shadow-xl">
          <div class="flex items-center justify-between mb-2.5 border-b border-slate-800 pb-2">
            <div class="flex items-center space-x-2">
              <span class="text-amber-400 text-base">👥</span>
              <h3 class="text-sm font-bold text-white font-display">Substitutes Bench</h3>
            </div>
            <div class="flex space-x-1 text-xs">
              <button id="btn-bench-bayern" class="px-2 py-0.5 rounded-lg bg-red-900/60 text-red-200 border border-red-700 font-bold text-[10px] cursor-pointer">Bayern</button>
              <button id="btn-bench-psg" class="px-2 py-0.5 rounded-lg bg-slate-800 text-slate-400 hover:text-white font-bold text-[10px] cursor-pointer">PSG</button>
            </div>
          </div>

          <!-- Bench list -->
          <div id="bench-list-container" class="space-y-2 max-h-[300px] overflow-y-auto pr-1"></div>
        </div>

      </div>

    </div>

  </div>

  <!-- Modal: AI Position Performance Predictor Full Lab -->
  <div id="modal-predictor" class="fixed inset-0 z-50 bg-black/80 card-modal-blur hidden items-center justify-center p-3 sm:p-5 overflow-y-auto">
    <div class="bg-[#0f1420] border border-slate-800 rounded-3xl p-5 sm:p-6 shadow-2xl max-w-3xl w-full text-slate-100 my-auto animate-fadeIn">
      <div class="flex items-center justify-between border-b border-slate-800 pb-3 mb-4">
        <div class="flex items-center space-x-2.5">
          <span class="text-xl text-cyan-400">✨</span>
          <div>
            <h3 class="text-lg font-bold text-white font-display">Tactical Position & Performance Predictor</h3>
            <p class="text-xs text-slate-400">AI prediction model evaluates attributes, fatigue, form streak, and opponent duels.</p>
          </div>
        </div>
        <button class="btn-close-modal p-2 rounded-xl text-slate-400 hover:text-white hover:bg-slate-800 transition cursor-pointer">✕</button>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
        <div>
          <label class="block text-xs font-semibold text-slate-400 uppercase mb-1">Choose Player</label>
          <select id="modal-player-select" class="w-full bg-slate-950 border border-slate-700 rounded-xl px-3 py-2 text-sm text-white font-medium"></select>
        </div>
        <div>
          <label class="block text-xs font-semibold text-slate-400 uppercase mb-1">Target Position</label>
          <div id="modal-position-grid" class="grid grid-cols-5 gap-1.5 max-h-28 overflow-y-auto pr-1"></div>
        </div>
      </div>

      <div id="modal-prediction-details" class="space-y-4"></div>

      <div class="flex items-center justify-end space-x-3 border-t border-slate-800 pt-4 mt-4">
        <button class="btn-close-modal px-4 py-2 text-xs font-semibold text-slate-400 hover:text-white transition cursor-pointer">Close</button>
        <button id="btn-modal-apply" class="px-5 py-2.5 bg-gradient-to-r from-emerald-600 to-teal-600 text-white text-xs font-bold rounded-xl shadow-lg transition hover:scale-105 cursor-pointer">
          Deploy to Starting XI
        </button>
      </div>
    </div>
  </div>

  <!-- Modal: Sofascore Player Deep Stats Card -->
  <div id="modal-player-details" class="fixed inset-0 z-50 bg-black/80 card-modal-blur hidden items-center justify-center p-3 sm:p-5 overflow-y-auto">
    <div class="bg-[#0f1420] border border-slate-800 rounded-3xl p-5 sm:p-6 shadow-2xl max-w-2xl w-full text-slate-100 my-auto animate-fadeIn">
      <div class="flex items-center justify-between border-b border-slate-800 pb-3 mb-4">
        <div class="flex items-center space-x-3">
          <div id="detail-avatar" class="w-12 h-12 rounded-full overflow-hidden border-2 border-cyan-400 bg-slate-800"></div>
          <div>
            <div class="flex items-center space-x-2">
              <h3 id="detail-name" class="text-lg font-bold text-white font-display">Manuel Neuer</h3>
              <span id="detail-pos" class="text-xs px-2 py-0.5 rounded bg-slate-800 text-cyan-300 font-mono font-bold">GK</span>
            </div>
            <p id="detail-sub" class="text-xs text-slate-400 mt-0.5">38 yrs • Germany • Bayern München</p>
          </div>
        </div>
        <button class="btn-close-modal p-2 rounded-xl text-slate-400 hover:text-white hover:bg-slate-800 transition cursor-pointer">✕</button>
      </div>

      <div id="player-modal-content" class="space-y-4"></div>

      <div class="flex items-center justify-between border-t border-slate-800 pt-4 mt-4">
        <button id="btn-open-predictor-for-selected" class="px-4 py-2 bg-gradient-to-r from-cyan-600 to-blue-600 text-white text-xs font-bold rounded-xl shadow transition cursor-pointer flex items-center space-x-1.5">
          <span>🔮</span>
          <span>Predict Positions for this Player</span>
        </button>
        <button class="btn-close-modal px-4 py-2 text-xs font-semibold text-slate-400 hover:text-white transition cursor-pointer">Close</button>
      </div>
    </div>
  </div>

  <!-- Modal: Tactical Manager Game & Scenario Challenge Mode -->
  <div id="modal-game" class="fixed inset-0 z-50 bg-black/80 card-modal-blur hidden items-center justify-center p-3 sm:p-5 overflow-y-auto">
    <div class="bg-[#0f1420] border border-slate-800 rounded-3xl p-5 sm:p-6 shadow-2xl max-w-3xl w-full text-slate-100 my-auto animate-fadeIn">
      <div class="flex items-center justify-between border-b border-slate-800 pb-3 mb-4">
        <div class="flex items-center space-x-2.5">
          <span class="text-2xl text-amber-400">🎮</span>
          <div>
            <h3 class="text-lg font-bold text-white font-display">Tactical Manager Challenge & Rating Quiz</h3>
            <p class="text-xs text-slate-400">Solve real-time match crises, substitute struggling players, or predict position ratings.</p>
          </div>
        </div>
        <button class="btn-close-modal p-2 rounded-xl text-slate-400 hover:text-white hover:bg-slate-800 transition cursor-pointer">✕</button>
      </div>

      <!-- Challenge Tabs -->
      <div class="flex space-x-2 border-b border-slate-800 pb-3 mb-4">
        <button id="btn-game-tab-scenario" class="px-3.5 py-1.5 rounded-xl text-xs font-bold bg-amber-600 text-white shadow cursor-pointer">Manager Crises (3)</button>
        <button id="btn-game-tab-quiz" class="px-3.5 py-1.5 rounded-xl text-xs font-bold text-slate-400 hover:text-white hover:bg-slate-800 cursor-pointer">Prediction Quiz (3)</button>
      </div>

      <div id="game-modal-body" class="space-y-4 max-h-[60vh] overflow-y-auto pr-1"></div>

      <div class="flex items-center justify-between border-t border-slate-800 pt-4 mt-4">
        <div class="text-xs text-slate-400">
          Manager Score: <span id="game-manager-score" class="font-bold text-amber-400 font-mono text-sm">0 pts</span>
        </div>
        <button class="btn-close-modal px-4 py-2 text-xs font-semibold text-slate-400 hover:text-white transition cursor-pointer">Close</button>
      </div>
    </div>
  </div>

  <!-- Modal: Live 15-Minute Match Simulator -->
  <div id="modal-sim" class="fixed inset-0 z-50 bg-black/80 card-modal-blur hidden items-center justify-center p-3 sm:p-5 overflow-y-auto">
    <div class="bg-[#0f1420] border border-slate-800 rounded-3xl p-5 sm:p-6 shadow-2xl max-w-2xl w-full text-slate-100 my-auto animate-fadeIn">
      <div class="flex items-center justify-between border-b border-slate-800 pb-3 mb-4">
        <div class="flex items-center space-x-2.5">
          <span class="text-2xl text-emerald-400">⚡</span>
          <div>
            <h3 class="text-lg font-bold text-white font-display">Live Match Simulation (74' to 90'+)</h3>
            <p class="text-xs text-slate-400">Watch simulated match events and live rating adjustments based on your tactical lineup!</p>
          </div>
        </div>
        <button class="btn-close-modal p-2 rounded-xl text-slate-400 hover:text-white hover:bg-slate-800 transition cursor-pointer">✕</button>
      </div>

      <div class="space-y-4">
        <!-- Progress Bar & Clock -->
        <div class="bg-slate-900 p-4 rounded-2xl border border-slate-800 text-center">
          <div class="flex justify-between items-center text-xs text-slate-400 mb-2">
            <span>74' Min</span>
            <span id="sim-clock" class="text-base font-black text-emerald-400 font-mono">74'</span>
            <span>90'+ Min</span>
          </div>
          <div class="w-full h-3 bg-slate-950 rounded-full overflow-hidden border border-slate-800">
            <div id="sim-progress-bar" class="h-full bg-gradient-to-r from-emerald-500 to-cyan-400 transition-all duration-300" style="width: 0%"></div>
          </div>
        </div>

        <!-- Event Log / Commentary Feed -->
        <div class="bg-slate-950 p-4 rounded-2xl border border-slate-800">
          <h4 class="text-xs font-bold text-slate-400 uppercase tracking-wider mb-2">Match Ticker</h4>
          <div id="sim-event-feed" class="space-y-2 max-h-44 overflow-y-auto text-xs font-medium text-slate-300">
            <div class="p-2 rounded-lg bg-slate-900 border border-slate-800 text-slate-400">
              Click "Start 15' Simulation" to calculate live match outcomes.
            </div>
          </div>
        </div>

        <div class="flex items-center justify-end space-x-3 pt-2">
          <button id="btn-start-simulation" class="px-5 py-2.5 bg-gradient-to-r from-emerald-600 to-teal-600 text-white text-xs font-bold rounded-xl shadow-lg transition hover:scale-105 flex items-center space-x-1.5 cursor-pointer">
            <span>▶</span>
            <span>Start 15' Simulation</span>
          </button>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal: WordPress Embed Helper Code -->
  <div id="modal-wordpress" class="fixed inset-0 z-50 bg-black/80 card-modal-blur hidden items-center justify-center p-3 sm:p-5 overflow-y-auto">
    <div class="bg-[#0f1420] border border-slate-800 rounded-3xl p-5 sm:p-6 shadow-2xl max-w-2xl w-full text-slate-100 my-auto animate-fadeIn">
      <div class="flex items-center justify-between border-b border-slate-800 pb-3 mb-4">
        <div class="flex items-center space-x-2.5">
          <span class="text-2xl text-blue-400">📋</span>
          <div>
            <h3 class="text-lg font-bold text-white font-display">Embed in WordPress</h3>
            <p class="text-xs text-slate-400">How to add this interactive tactical predictor game to your WordPress site.</p>
          </div>
        </div>
        <button class="btn-close-modal p-2 rounded-xl text-slate-400 hover:text-white hover:bg-slate-800 transition cursor-pointer">✕</button>
      </div>

      <div class="space-y-4 text-xs text-slate-300">
        <div class="bg-slate-900 p-3 rounded-xl border border-slate-800">
          <h4 class="font-bold text-white mb-1">Option A: Gutenberg Custom HTML Block (Easiest)</h4>
          <p class="mb-2 text-slate-400">In your WordPress Post or Page editor, add a <strong>Custom HTML</strong> block and paste an iframe pointing to your hosted HTML page, or paste this self-contained HTML:</p>
          <div class="relative">
            <textarea id="wp-iframe-code" readonly class="w-full h-24 bg-slate-950 border border-slate-700 rounded-lg p-2 font-mono text-[11px] text-cyan-300 resize-none">&lt;iframe src="https://your-domain.com/tactical-predictor.html" width="100%" height="950" frameborder="0" style="border-radius: 16px; box-shadow: 0 10px 30px rgba(0,0,0,0.5);" allow="autoplay"&gt;&lt;/iframe&gt;</textarea>
            <button onclick="navigator.clipboard.writeText(document.getElementById('wp-iframe-code').value); alert('Iframe code copied to clipboard!');" class="absolute top-2 right-2 px-2.5 py-1 bg-cyan-600 hover:bg-cyan-500 text-white rounded text-[10px] font-bold cursor-pointer">Copy Iframe</button>
          </div>
        </div>

        <div class="bg-slate-900 p-3 rounded-xl border border-slate-800">
          <h4 class="font-bold text-white mb-1">Option B: Elementor / Divi HTML Widget</h4>
          <p class="text-slate-400">Drag an HTML Widget into Elementor or Divi and insert the embed code above. Set section width to "Full Width" or "Boxed 1000px" for best responsive visual layout.</p>
        </div>
      </div>

      <div class="flex justify-end pt-4 border-t border-slate-800 mt-4">
        <button class="btn-close-modal px-4 py-2 text-xs font-semibold text-slate-400 hover:text-white transition cursor-pointer">Done</button>
      </div>
    </div>
  </div>

  <!-- Load Core Modular Javascript Engine -->
  <script src="/src/standalone-app.js"></script>
</body>
</html>
