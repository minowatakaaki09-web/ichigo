<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>イチゴ選別 分太AI (パック換算・平パック対応版)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;900&display=swap');
        body { font-family: 'Inter', sans-serif; touch-action: manipulation; }
        .custom-scrollbar::-webkit-scrollbar { width: 6px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #475569; border-radius: 4px; }
    </style>
</head>
<body class="bg-slate-900 text-slate-100 min-h-screen flex flex-col font-sans">

    <!-- HEADER -->
    <header class="bg-slate-800 border-b border-slate-700 px-4 py-3 flex justify-between items-center sticky top-0 z-50 shadow-md">
        <div class="flex items-center space-x-3">
            <div class="bg-gradient-to-r from-red-500 to-rose-600 text-white p-2 rounded-xl shadow-lg text-xl">
                🍓
            </div>
            <div>
                <h1 class="text-xl font-bold tracking-tight text-white flex items-center gap-2">
                    イチゴ減算秤 <span class="bg-emerald-500/20 text-emerald-400 text-xs px-2 py-0.5 rounded-full border border-emerald-500/30">パック集計版</span>
                </h1>
                <p class="text-xs text-slate-400">リアルタイム直読・完全安定判定</p>
            </div>
        </div>
        <div class="flex items-center gap-2">
            <button id="btn-connect" onclick="connectScale()" class="bg-emerald-600 hover:bg-emerald-500 active:scale-95 text-white text-xs font-bold px-3 py-2 rounded-lg transition duration-200 flex items-center gap-2 shadow-md">
                <i class="fa-brands fa-bluetooth-b"></i> <span id="conn-text">スケール接続</span>
            </button>
        </div>
    </header>

    <!-- MAIN CONTENT -->
    <main class="flex-1 max-w-5xl w-full mx-auto p-3 md:p-6 grid grid-cols-1 md:grid-cols-12 gap-4">

        <!-- LEFT PANEL -->
        <section class="md:col-span-7 flex flex-col gap-4">

            <!-- MAIN DISPLAY -->
            <div class="bg-slate-800/80 backdrop-blur-md rounded-2xl border border-slate-700/80 p-5 shadow-xl flex flex-col items-center justify-center relative overflow-hidden">
                <div class="absolute top-3 left-4 text-xs font-bold text-slate-400 flex items-center gap-1">
                    <span id="status-indicator" class="w-2.5 h-2.5 rounded-full bg-slate-500 inline-block"></span>
                    <span id="status-text">未接続（模擬テスト可）</span>
                </div>

                <!-- RANK DISPLAY -->
                <div class="mt-3 text-center">
                    <div id="rank-badge" class="inline-block px-6 py-2 rounded-2xl font-black text-3xl md:text-4xl shadow-inner transition-all duration-300 bg-slate-700 text-slate-400 border border-slate-600">
                        ----
                    </div>
                </div>

                <!-- WEIGHT & PREV TOTAL DISPLAY (SIDE BY SIDE) -->
                <div class="w-full my-4 grid grid-cols-2 gap-3 bg-slate-900/60 p-4 rounded-xl border border-slate-700/60 text-center">
                    <div class="border-r border-slate-700/80 pr-2">
                        <div class="text-[11px] font-semibold text-slate-400 mb-1 uppercase tracking-wider">今回引いた重さ</div>
                        <div class="flex items-baseline justify-center gap-1">
                            <span id="removed-weight-display" class="text-4xl md:text-5xl font-black tracking-tight text-white font-mono">0.0</span>
                            <span class="text-lg font-bold text-slate-400">g</span>
                        </div>
                    </div>
                    <div class="pl-2">
                        <div class="text-[11px] font-semibold text-slate-400 mb-1 uppercase tracking-wider">直前の総量</div>
                        <div class="flex items-baseline justify-center gap-1">
                            <span id="prev-base-weight" class="text-3xl md:text-4xl font-black tracking-tight text-amber-400 font-mono">0.0</span>
                            <span class="text-base font-bold text-slate-400">g</span>
                        </div>
                    </div>
                </div>

                <!-- SUB INFO -->
                <div class="w-full bg-slate-900/40 rounded-xl p-2.5 grid grid-cols-2 gap-2 text-center border border-slate-700/40 text-xs">
                    <div>
                        <div class="text-slate-500 font-medium">スケール現在値</div>
                        <div class="text-slate-200 font-bold font-mono text-sm"><span id="gross-weight">0.0</span> g</div>
                    </div>
                    <div class="border-l border-slate-700/80 pl-1">
                        <div class="text-slate-500 font-medium">現在の基準重量</div>
                        <div class="text-slate-200 font-bold font-mono text-sm"><span id="base-weight">0.0</span> g</div>
                    </div>
                </div>

                <!-- COMMUNICATION DEBUG MONITOR -->
                <div class="w-full mt-3 bg-slate-950 p-2.5 rounded-xl border border-slate-800 text-xs font-mono text-slate-400 flex flex-col gap-1">
                    <div class="flex justify-between items-center text-[11px] text-slate-400 border-b border-slate-800 pb-1">
                        <span><i class="fa-solid fa-bug text-emerald-400"></i> データモニター (インデックス3-4固定)</span>
                        <span id="raw-data-status" class="text-emerald-400">待機中</span>
                    </div>
                    <div class="text-slate-300 truncate">受信Bytes: <span id="raw-bytes-display" class="text-amber-300 font-bold">[-]</span></div>
                </div>

                <!-- ACTION BUTTONS -->
                <div class="grid grid-cols-3 gap-2 w-full mt-4">
                    <button onclick="setTareBasket()" class="bg-red-600 hover:bg-red-500 active:scale-95 text-white font-bold py-3 px-2 rounded-xl shadow-lg transition duration-150 text-xs md:text-sm flex flex-col items-center justify-center gap-1">
                        <i class="fa-solid fa-basket-shopping text-base"></i> カゴセット
                    </button>
                    <button onclick="undoLastItem()" class="bg-amber-600 hover:bg-amber-500 active:scale-95 text-white font-bold py-3 px-2 rounded-xl shadow transition duration-150 text-xs md:text-sm flex flex-col items-center justify-center gap-1">
                        <i class="fa-solid fa-rotate-left text-base"></i> 直前を取り消し
                    </button>
                    <button onclick="toggleSimPanel()" class="bg-slate-700 hover:bg-slate-600 active:scale-95 text-slate-200 font-bold py-3 px-2 rounded-xl shadow transition duration-150 text-xs md:text-sm flex flex-col items-center justify-center gap-1">
                        <i class="fa-solid fa-gamepad text-base"></i> テスト入力
                    </button>
                </div>
            </div>

            <!-- SIMULATOR PANEL -->
            <div id="sim-panel" class="bg-slate-800/90 rounded-2xl border border-amber-500/30 p-4 shadow-lg hidden">
                <div class="text-xs font-bold text-amber-400 mb-2 flex items-center justify-between">
                    <span><i class="fa-solid fa-flask"></i> 模擬テスト</span>
                    <button onclick="toggleSimPanel()" class="text-slate-400 hover:text-white"><i class="fa-solid fa-xmark"></i></button>
                </div>
                <div class="grid grid-cols-4 gap-2">
                    <button onclick="simTakeBerry(4.0)" class="bg-slate-700 hover:bg-slate-600 text-white font-bold py-2 rounded-lg text-xs">4g玉(スルー確認)</button>
                    <button onclick="simTakeBerry(17.0)" class="bg-slate-700 hover:bg-slate-600 text-white font-bold py-2 rounded-lg text-xs">17g玉</button>
                    <button onclick="simTakeBerry(35.0)" class="bg-slate-700 hover:bg-slate-600 text-white font-bold py-2 rounded-lg text-xs">8玉サイズ(35g)</button>
                    <button onclick="simTakeBerry(100.0)" class="bg-slate-700 hover:bg-slate-600 text-white font-bold py-2 rounded-lg text-xs">大粒(100g)</button>
                </div>
            </div>

            <!-- RANK CONFIG -->
            <div class="bg-slate-800/80 backdrop-blur-md rounded-2xl border border-slate-700/80 p-4 shadow-lg">
                <h3 class="text-sm font-bold text-slate-300 mb-3 flex items-center justify-between">
                    <span class="flex items-center gap-2"><i class="fa-solid fa-sliders text-red-400"></i> 階級・閾値設定 (下限値 g)</span>
                </h3>
                <div class="grid grid-cols-4 sm:grid-cols-6 gap-2 text-center text-xs">
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700"><span class="block text-slate-400 font-bold mb-1">6玉</span><input type="number" id="th-6" value="47.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600"></div>
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700"><span class="block text-slate-400 font-bold mb-1">7玉</span><input type="number" id="th-7" value="41.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600"></div>
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700"><span class="block text-slate-400 font-bold mb-1">8玉</span><input type="number" id="th-8" value="35.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600"></div>
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700"><span class="block text-slate-400 font-bold mb-1">9玉</span><input type="number" id="th-9" value="30.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600"></div>
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700"><span class="block text-slate-400 font-bold mb-1">10玉</span><input type="number" id="th-10" value="27.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600"></div>
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700"><span class="block text-slate-400 font-bold mb-1">11玉</span><input type="number" id="th-11" value="25.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600"></div>
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700"><span class="block text-slate-400 font-bold mb-1">12玉</span><input type="number" id="th-12" value="23.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600"></div>
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700"><span class="block text-slate-400 font-bold mb-1">2L</span><input type="number" id="th-2l" value="18.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600"></div>
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700"><span class="block text-slate-400 font-bold mb-1">L</span><input type="number" id="th-l" value="12.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600"></div>
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700"><span class="block text-slate-400 font-bold mb-1">M</span><input type="number" id="th-m" value="9.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600"></div>
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700"><span class="block text-slate-400 font-bold mb-1">S</span><input type="number" id="th-s" value="6.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600"></div>
                </div>
            </div>

        </section>

        <!-- RIGHT PANEL -->
        <section class="md:col-span-5 flex flex-col gap-4">

            <!-- STATS SUMMARY -->
            <div class="bg-slate-800/80 backdrop-blur-md rounded-2xl border border-slate-700/80 p-4 shadow-lg">
                <div class="flex justify-between items-center mb-3">
                    <h3 class="text-sm font-bold text-slate-300 flex items-center gap-2">
                        <i class="fa-solid fa-chart-pie text-emerald-400"></i> 本日の選別集計
                    </h3>
                    <span id="total-count-badge" class="bg-emerald-500/20 text-emerald-300 text-xs font-bold px-2.5 py-0.5 rounded-full border border-emerald-500/30">
                        合計: 0 個
                    </span>
                </div>
                <div class="grid grid-cols-4 gap-1.5 text-center text-xs mb-3" id="stats-grid"></div>
                
                <!-- PACK SUMMARY (1パック 280g) -->
                <div class="bg-slate-900/80 p-3.5 rounded-xl border border-amber-500/30 mb-3 space-y-2.5 shadow-inner">
                    <div class="text-xs font-bold text-amber-300 flex items-center justify-between border-b border-slate-800 pb-1.5">
                        <span><i class="fa-solid fa-box-open text-amber-400"></i> パック合計 (1パック 280g基準)</span>
                    </div>
                    <div class="grid grid-cols-2 gap-2 text-xs">
                        <div class="bg-slate-800/90 p-2.5 rounded-lg border border-slate-700">
                            <div class="text-slate-400 text-[10px]">レギュラーパック <span class="text-slate-500">(S~2L)</span></div>
                            <div class="text-slate-200 font-bold mt-0.5"><span id="pack-reg-weight">0.0</span>g</div>
                            <div class="text-emerald-400 font-extrabold text-sm mt-0.5"><span id="pack-reg-count">0.00</span> パック</div>
                        </div>
                        <div class="bg-slate-800/90 p-2.5 rounded-lg border border-slate-700">
                            <div class="text-slate-400 text-[10px]">平パック <span class="text-slate-500">(6〜12玉)</span></div>
                            <div class="text-slate-200 font-bold mt-0.5"><span id="pack-flat-weight">0.0</span>g</div>
                            <div class="text-rose-400 font-extrabold text-sm mt-0.5"><span id="pack-flat-count">0.00</span> パック</div>
                        </div>
                    </div>
                </div>

                <div class="pt-2 border-t border-slate-700/80 flex justify-between items-center text-xs text-slate-400">
                    <span>総重量: <strong id="total-weight" class="text-slate-200">0.0</strong>g</span>
                    <div class="flex gap-2">
                        <button onclick="copyResults()" class="text-blue-400 hover:text-blue-300 transition"><i class="fa-solid fa-copy"></i> コピー</button>
                        <button onclick="resetStats()" class="text-slate-500 hover:text-red-400 transition"><i class="fa-solid fa-rotate-right"></i> リセット</button>
                    </div>
                </div>
            </div>

            <!-- LOG HISTORY -->
            <div class="bg-slate-800/80 backdrop-blur-md rounded-2xl border border-slate-700/80 p-4 shadow-lg flex-1 flex flex-col min-h-[160px]">
                <h3 class="text-sm font-bold text-slate-300 mb-2 flex items-center gap-2">
                    <i class="fa-solid fa-list-ol text-blue-400"></i> 選別履歴ログ
                </h3>
                <div id="log-container" class="flex-1 overflow-y-auto max-h-[160px] custom-scrollbar space-y-1.5 pr-1 text-xs">
                    <div class="text-slate-500 text-center py-6">選別データはまだありません</div>
                </div>
            </div>

        </section>

    </main>

    <!-- SCRIPT -->
    <script>
        const RanksDef = [
            { key: '6', name: '6玉', group: 'flat' }, 
            { key: '7', name: '7玉', group: 'flat' }, 
            { key: '8', name: '8玉', group: 'flat' },
            { key: '9', name: '9玉', group: 'flat' }, 
            { key: '10', name: '10玉', group: 'flat' }, 
            { key: '11', name: '11玉', group: 'flat' },
            { key: '12', name: '12玉', group: 'flat' }, 
            { key: '2L', name: '2Lサイズ', group: 'regular' }, 
            { key: 'L', name: 'Lサイズ', group: 'regular' },
            { key: 'M', name: 'Mサイズ', group: 'regular' }, 
            { key: 'S', name: 'Sサイズ', group: 'regular' }, 
            { key: 'out', name: '規格外', group: 'out' }
        ];

        const state = {
            lastGrossWeight: 0.0,
            baseWeight: 0.0,
            prevBaseWeight: 0.0,
            isBasketSet: false,
            bluetoothDevice: null,
            stats: {},
            logs: []
        };

        // 完全安定待ち判定用の変数
        let lastStableWeight = -1;
        let stableCount = 0;
        const STABLE_THRESHOLD_COUNT = 4; // 約300〜400ms間、値が完全に静止したら確定

        RanksDef.forEach(r => {
            state.stats[r.key] = { count: 0, weight: 0.0 };
        });
        state.stats.totalCount = 0;
        state.stats.totalWeight = 0.0;
        initStatsUI();

        function initStatsUI() {
            const grid = document.getElementById('stats-grid');
            grid.innerHTML = RanksDef.map(r => `
                <div class="bg-slate-900/60 p-2 rounded-lg border border-slate-700">
                    <div class="text-slate-400 text-[10px]">${r.name}</div>
                    <div id="count-${r.key}" class="text-base font-bold text-slate-200">0</div>
                </div>
            `).join('');
        }

        const synth = window.speechSynthesis;
        function playBeep() {
            try {
                const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                const osc = audioCtx.createOscillator();
                const gainNode = audioCtx.createGain();
                osc.type = 'sine';
                osc.frequency.value = 880; 
                gainNode.gain.setValueAtTime(0.1, audioCtx.currentTime);
                gainNode.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.15);
                osc.connect(gainNode);
                gainNode.connect(audioCtx.destination);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.15);
            } catch(e) {}
        }

        function speakText(text) {
            if (!synth) return;
            if (synth.speaking) synth.cancel();
            const utter = new SpeechSynthesisUtterance(text);
            utter.lang = 'ja-JP';
            utter.rate = 1.4;
            synth.speak(utter);
        }

        function getThresholds() {
            return {
                '6': parseFloat(document.getElementById('th-6').value) || 47.0,
                '7': parseFloat(document.getElementById('th-7').value) || 41.0,
                '8': parseFloat(document.getElementById('th-8').value) || 35.0,
                '9': parseFloat(document.getElementById('th-9').value) || 30.0,
                '10': parseFloat(document.getElementById('th-10').value) || 27.0,
                '11': parseFloat(document.getElementById('th-11').value) || 25.0,
                '12': parseFloat(document.getElementById('th-12').value) || 23.0,
                '2L': parseFloat(document.getElementById('th-2l').value) || 18.0,
                'L': parseFloat(document.getElementById('th-l').value) || 12.0,
                'M': parseFloat(document.getElementById('th-m').value) || 9.0,
                'S': parseFloat(document.getElementById('th-s').value) || 6.0,
            };
        }

        function evaluateRank(weight) {
            if (weight <= 5.0) return { name: '規格外', key: 'out', group: 'out' };

            const th = getThresholds();
            if (weight >= th['6']) return { name: '6玉', key: '6', group: 'flat' };
            if (weight >= th['7']) return { name: '7玉', key: '7', group: 'flat' };
            if (weight >= th['8']) return { name: '8玉', key: '8', group: 'flat' };
            if (weight >= th['9']) return { name: '9玉', key: '9', group: 'flat' };
            if (weight >= th['10']) return { name: '10玉', key: '10', group: 'flat' };
            if (weight >= th['11']) return { name: '11玉', key: '11', group: 'flat' };
            if (weight >= th['12']) return { name: '12玉', key: '12', group: 'flat' };
            if (weight >= th['2L']) return { name: '2Lサイズ', key: '2L', group: 'regular' };
            if (weight >= th['L']) return { name: 'Lサイズ', key: 'L', group: 'regular' };
            if (weight >= th['M']) return { name: 'Mサイズ', key: 'M', group: 'regular' };
            if (weight >= th['S']) return { name: 'Sサイズ', key: 'S', group: 'regular' };
            return { name: '規格外', key: 'out', group: 'out' };
        }

        function processGrossWeightUpdate(newGross) {
            state.lastGrossWeight = newGross;
            document.getElementById('gross-weight').innerText = newGross.toFixed(1);

            if (!state.isBasketSet) return;

            // 基準重量より重くなった場合（イチゴの補充や乗せ直し）
            if (newGross > state.baseWeight + 5.0) {
                lastStableWeight = -1;
                stableCount = 0;
                state.prevBaseWeight = state.baseWeight;
                state.baseWeight = newGross;
                document.getElementById('base-weight').innerText = state.baseWeight.toFixed(1);
                document.getElementById('prev-base-weight').innerText = state.prevBaseWeight.toFixed(1);
                document.getElementById('removed-weight-display').innerText = "0.0";
                
                const badge = document.getElementById('rank-badge');
                badge.innerText = "補充/更新";
                badge.className = "inline-block px-6 py-2 rounded-2xl font-black text-3xl md:text-4xl shadow-inner transition-all duration-300 bg-blue-900/80 text-blue-300 border border-blue-500/50";
                return;
            }

            // --- 完全安定判定ロジック ---
            // 前回受信した値とほぼ同じ（±0.5g以内）であれば、静止しているとみなす
            if (Math.abs(newGross - lastStableWeight) <= 0.5) {
                stableCount++;
            } else {
                // 動いている最中はカウントをリセットして、新しい値を追う
                lastStableWeight = newGross;
                stableCount = 0;
                
                // 動いている最中でも、画面のプレビュー表示だけはリアルタイムに更新
                const currentDiff = state.baseWeight - newGross;
                if (currentDiff >= 5.0) {
                    const tempRank = evaluateRank(currentDiff);
                    document.getElementById('removed-weight-display').innerText = currentDiff.toFixed(1);
                    const badge = document.getElementById('rank-badge');
                    badge.innerText = tempRank.name + " (計測中...)";
                    badge.className = "inline-block px-6 py-2 rounded-2xl font-black text-2xl md:text-3xl shadow-inner transition-all duration-300 border text-slate-300 bg-slate-800 border-slate-600";
                }
            }

            // 静止状態が一定回数（約300〜400ms）続いたら、ここで初めてスパッと確定！
            if (stableCount >= STABLE_THRESHOLD_COUNT) {
                const diffWeight = state.baseWeight - newGross;
                if (diffWeight >= 5.0) {
                    finalizePickedBerry(newGross, diffWeight);
                    // 確定したら安定カウンターをリセットして次の動作に備える
                    stableCount = 0;
                    lastStableWeight = -1;
                }
            }
        }

        function finalizePickedBerry(finalGross, diffWeight) {
            const rank = evaluateRank(diffWeight);
            document.getElementById('removed-weight-display').innerText = diffWeight.toFixed(1);
            
            const badge = document.getElementById('rank-badge');
            badge.innerText = rank.name;
            badge.className = "inline-block px-6 py-2 rounded-2xl font-black text-3xl md:text-4xl shadow-inner transition-all duration-300 border text-white bg-slate-700 border-slate-600";

            playBeep();
            // 数字だけをスパッと読み上げ
            speakText(`${diffWeight.toFixed(1)}`);
            
            state.prevBaseWeight = state.baseWeight;
            document.getElementById('prev-base-weight').innerText = state.prevBaseWeight.toFixed(1);

            recordLog(diffWeight, rank, state.baseWeight);
            updateStatsUI();

            state.baseWeight = finalGross;
            document.getElementById('base-weight').innerText = state.baseWeight.toFixed(1);
        }

        function setTareBasket() {
            if (state.lastGrossWeight <= 0) {
                alert("スケールの重量が0gです。カゴを乗せてからセットしてください。");
                return;
            }
            lastStableWeight = -1;
            stableCount = 0;
            state.baseWeight = state.lastGrossWeight;
            state.prevBaseWeight = state.baseWeight;
            state.isBasketSet = true;
            document.getElementById('base-weight').innerText = state.baseWeight.toFixed(1);
            document.getElementById('prev-base-weight').innerText = state.prevBaseWeight.toFixed(1);
            document.getElementById('removed-weight-display').innerText = "0.0";
            
            const badge = document.getElementById('rank-badge');
            badge.innerText = "準備完了";
            badge.className = "inline-block px-6 py-2 rounded-2xl font-black text-3xl md:text-4xl shadow-inner transition-all duration-300 bg-emerald-900/80 text-emerald-300 border border-emerald-500/50";

            playBeep();
            speakText("カゴを設定しました。");
        }

        function recordLog(weight, rank, prevBase) {
            const timeStr = new Date().toLocaleTimeString('ja-JP', { hour: '2-digit', minute: '2-digit', second: '2-digit' });
            state.logs.unshift({ time: timeStr, weight: weight, rankKey: rank.key, rankName: rank.name, prevBase: prevBase });
            
            if (state.stats[rank.key]) {
                state.stats[rank.key].count++;
                state.stats[rank.key].weight += weight;
            } else {
                state.stats['out'].count++;
                state.stats['out'].weight += weight;
            }
            state.stats.totalCount++;
            state.stats.totalWeight += weight;
            renderLogsUI();
        }

        function undoLastItem() {
            if (state.logs.length === 0) { alert("取り消す履歴がありません。"); return; }
            lastStableWeight = -1;
            stableCount = 0;
            const last = state.logs.shift();
            
            if (state.stats[last.rankKey]) {
                state.stats[last.rankKey].count--;
                state.stats[last.rankKey].weight -= last.weight;
            }
            state.stats.totalCount--;
            state.stats.totalWeight -= last.weight;
            
            state.baseWeight = last.prevBase;
            state.prevBaseWeight = last.prevBase;
            document.getElementById('base-weight').innerText = state.baseWeight.toFixed(1);
            document.getElementById('prev-base-weight').innerText = state.prevBaseWeight.toFixed(1);
            document.getElementById('removed-weight-display').innerText = `(-${last.weight.toFixed(1)}) 取消`;
            
            speakText("取り消しました。");
            updateStatsUI();
            renderLogsUI();
        }

        function renderLogsUI() {
            const container = document.getElementById('log-container');
            if (state.logs.length === 0) {
                container.innerHTML = '<div class="text-slate-500 text-center py-6">選別データはまだありません</div>';
                return;
            }
            container.innerHTML = state.logs.map(l => `
                <div class="flex justify-between items-center bg-slate-900/50 px-3 py-1.5 rounded border border-slate-700/50">
                    <span class="text-slate-500 font-mono">${l.time}</span>
                    <span class="font-bold text-slate-200">${l.rankName}</span>
                    <span class="font-mono text-emerald-400 font-bold">${l.weight.toFixed(1)} g</span>
                </div>
            `).join('');
        }

        function updateStatsUI() {
            RanksDef.forEach(r => {
                const el = document.getElementById(`count-${r.key}`);
                if (el) el.innerText = state.stats[r.key].count;
            });

            let regWeight = 0;
            let flatWeight = 0;

            RanksDef.forEach(r => {
                if (r.group === 'regular') regWeight += state.stats[r.key].weight;
                if (r.group === 'flat') flatWeight += state.stats[r.key].weight;
            });

            const regPacks = regWeight / 280.0;
            const flatPacks = flatWeight / 280.0;

            document.getElementById('pack-reg-weight').innerText = regWeight.toFixed(1);
            document.getElementById('pack-reg-count').innerText = regPacks.toFixed(2);
            document.getElementById('pack-flat-weight').innerText = flatWeight.toFixed(1);
            document.getElementById('pack-flat-count').innerText = flatPacks.toFixed(2);

            document.getElementById('total-count-badge').innerText = `合計: ${state.stats.totalCount} 個`;
            document.getElementById('total-weight').innerText = state.stats.totalWeight.toFixed(1);
        }

        function resetStats() {
            if(!confirm("集計をリセットしますか？")) return;
            lastStableWeight = -1;
            stableCount = 0;
            RanksDef.forEach(r => {
                state.stats[r.key].count = 0;
                state.stats[r.key].weight = 0.0;
            });
            state.stats.totalCount = 0;
            state.stats.totalWeight = 0.0;
            state.logs = [];
            state.prevBaseWeight = state.baseWeight;
            document.getElementById('prev-base-weight').innerText = state.prevBaseWeight.toFixed(1);
            renderLogsUI();
            updateStatsUI();
        }

        function copyResults() {
            let regW = 0, flatW = 0;
            RanksDef.forEach(r => {
                if (r.group === 'regular') regW += state.stats[r.key].weight;
                if (r.group === 'flat') flatW += state.stats[r.key].weight;
            });
            
            let text = "【イチゴ選別結果】\n" + 
                RanksDef.map(r => `${r.name}: ${state.stats[r.key].count}個 (${state.stats[r.key].weight.toFixed(1)}g)`).join('\n') + 
                `\n\n--- パック合計 (280g/パック) ---\n` +
                `レギュラーパック (S~2L): ${(regW/280).toFixed(2)}パック (${regW.toFixed(1)}g)\n` +
                `平パック (6~12玉): ${(flatW/280).toFixed(2)}パック (${flatW.toFixed(1)}g)\n\n` +
                `総計: ${state.stats.totalCount}個 (${state.stats.totalWeight.toFixed(1)}g)`;
            
            navigator.clipboard.writeText(text).then(() => alert("結果をコピーしました！"));
        }

        async function connectScale() {
            try {
                document.getElementById('conn-text').innerText = "接続中...";
                state.bluetoothDevice = await navigator.bluetooth.requestDevice({
                    acceptAllDevices: true,
                    optionalServices: [
                        '0000ffe0-0000-1000-8000-00805f9b34fb',
                        '0000fff0-0000-1000-8000-00805f9b34fb',
                        '0000ff00-0000-1000-8000-00805f9b34fb',
                        '0000181d-0000-1000-8000-00805f9b34fb'
                    ]
                });

                state.bluetoothDevice.addEventListener('gattserverdisconnected', onDisconnected);

                const server = await state.bluetoothDevice.gatt.connect();
                const services = await server.getPrimaryServices();
                for (const service of services) {
                    const characteristics = await service.getCharacteristics();
                    for (const char of characteristics) {
                        if (char.properties.notify || char.properties.indicate) {
                            await char.startNotifications();
                            char.addEventListener('characteristicvaluechanged', (e) => {
                                parseScaleData(e.target.value);
                            });
                        }
                    }
                }

                document.getElementById('conn-text').innerText = "接続済み";
                document.getElementById('status-indicator').className = "w-2.5 h-2.5 rounded-full bg-emerald-500 inline-block animate-pulse";
                document.getElementById('status-text').innerText = "スマートスケール接続中";
                playBeep();
                speakText("接続完了");

            } catch (error) {
                document.getElementById('conn-text').innerText = "接続失敗";
                alert("エラー: " + error.message);
            }
        }

        function onDisconnected() {
            document.getElementById('conn-text').innerText = "スケール接続";
            document.getElementById('status-indicator').className = "w-2.5 h-2.5 rounded-full bg-slate-500 inline-block";
            document.getElementById('status-text').innerText = "未接続（切断されました）";
            speakText("スケールが切断されました");
        }

        function parseScaleData(value) {
            let bytes = [];
            for (let i = 0; i < value.byteLength; i++) { bytes.push(value.getUint8(i)); }
            
            document.getElementById('raw-bytes-display').innerText = `[${bytes.join(', ')}]`;
            document.getElementById('raw-data-status').innerText = "受信 " + new Date().toLocaleTimeString();

            let weight = 0;
            if (value.byteLength >= 5) {
                weight = value.getUint16(3, false);
            }

            processGrossWeightUpdate(weight);
        }

        function simTakeBerry(weight) {
            if (!state.isBasketSet) simSetBasket();
            processGrossWeightUpdate(state.lastGrossWeight - weight);
        }
        function simSetBasket() {
            state.lastGrossWeight = 500.0;
            setTareBasket();
        }
        function toggleSimPanel() {
            document.getElementById('sim-panel').classList.toggle('hidden');
        }
    </script>
</body>
</html>
