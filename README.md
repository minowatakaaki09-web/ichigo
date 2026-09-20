<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>イチゴ選別 分太AI (完全完璧版)</title>
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
                    イチゴ減算秤 <span class="bg-emerald-500/20 text-emerald-400 text-xs px-2 py-0.5 rounded-full border border-emerald-500/30">完全完璧版</span>
                </h1>
                <p class="text-xs text-slate-400">リアルタイム直読・即時判定</p>
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
            <div class="bg-slate-800/80 backdrop-blur-md rounded-2xl border border-slate-700/80 p-6 shadow-xl flex flex-col items-center justify-center relative overflow-hidden">
                <div class="absolute top-3 left-4 text-xs font-bold text-slate-400 flex items-center gap-1">
                    <span id="status-indicator" class="w-2.5 h-2.5 rounded-full bg-slate-500 inline-block"></span>
                    <span id="status-text">未接続（模擬テスト可）</span>
                </div>

                <!-- RANK DISPLAY -->
                <div class="mt-4 text-center">
                    <div id="rank-badge" class="inline-block px-6 py-2 rounded-2xl font-black text-3xl md:text-4xl shadow-inner transition-all duration-300 bg-slate-700 text-slate-400 border border-slate-600">
                        ----
                    </div>
                </div>

                <!-- WEIGHT DISPLAY -->
                <div class="my-5 text-center">
                    <div class="text-xs font-semibold text-slate-400 mb-1 uppercase tracking-wider">今回引いたイチゴの重さ</div>
                    <div class="flex items-baseline justify-center gap-2">
                        <span id="removed-weight-display" class="text-6xl md:text-7xl font-black tracking-tight text-white font-mono">0.0</span>
                        <span class="text-2xl font-bold text-slate-400">g</span>
                    </div>
                </div>

                <!-- SUB INFO -->
                <div class="w-full bg-slate-900/60 rounded-xl p-3 flex justify-around text-center border border-slate-700/50 text-xs">
                    <div>
                        <div class="text-slate-500 font-medium">スケール現在値</div>
                        <div class="text-slate-200 font-bold font-mono text-base"><span id="gross-weight">0.0</span> g</div>
                    </div>
                    <div class="border-r border-slate-700/80"></div>
                    <div>
                        <div class="text-slate-500 font-medium">基準重量 (カゴ総重)</div>
                        <div class="text-slate-200 font-bold font-mono text-base"><span id="base-weight">0.0</span> g</div>
                    </div>
                </div>

                <!-- COMMUNICATION DEBUG MONITOR -->
                <div class="w-full mt-3 bg-slate-950 p-3 rounded-xl border border-slate-800 text-xs font-mono text-slate-400 flex flex-col gap-1">
                    <div class="flex justify-between items-center text-[11px] text-slate-400 border-b border-slate-800 pb-1">
                        <span><i class="fa-solid fa-bug text-emerald-400"></i> データモニター (正常直読)</span>
                        <span id="raw-data-status" class="text-emerald-400">待機中</span>
                    </div>
                    <div class="text-slate-300">受信データBytes: <span id="raw-bytes-display" class="text-amber-300 font-bold">[-]</span></div>
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
                    <button onclick="simTakeBerry(17.0)" class="bg-slate-700 hover:bg-slate-600 text-white font-bold py-2 rounded-lg text-xs">17g玉</button>
                    <button onclick="simTakeBerry(52.5)" class="bg-slate-700 hover:bg-slate-600 text-white font-bold py-2 rounded-lg text-xs">大粒(52.5g)</button>
                    <button onclick="simTakeBerry(88.0)" class="bg-slate-700 hover:bg-slate-600 text-white font-bold py-2 rounded-lg text-xs">特大(88g)</button>
                    <button onclick="simTakeBerry(120.0)" class="bg-slate-700 hover:bg-slate-600 text-white font-bold py-2 rounded-lg text-xs">超特大(120g)</button>
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
                <div id="log-container" class="flex-1 overflow-y-auto max-h-[180px] custom-scrollbar space-y-1.5 pr-1 text-xs">
                    <div class="text-slate-500 text-center py-6">選別データはまだありません</div>
                </div>
            </div>

        </section>

    </main>

    <!-- SCRIPT -->
    <script>
        const RanksDef = [
            { key: '6', name: '6玉' }, { key: '7', name: '7玉' }, { key: '8', name: '8玉' },
            { key: '9', name: '9玉' }, { key: '10', name: '10玉' }, { key: '11', name: '11玉' },
            { key: '12', name: '12玉' }, { key: '2L', name: '2Lサイズ' }, { key: 'L', name: 'Lサイズ' },
            { key: 'M', name: 'Mサイズ' }, { key: 'S', name: 'Sサイズ' }, { key: 'out', name: '規格外' }
        ];

        const state = {
            lastGrossWeight: 0.0,
            baseWeight: 0.0,
            isBasketSet: false,
            bluetoothDevice: null,
            stats: {},
            logs: []
        };

        RanksDef.forEach(r => state.stats[r.key] = 0);
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
            const th = getThresholds();
            if (weight >= th['6']) return { name: '6玉', key: '6' };
            if (weight >= th['7']) return { name: '7玉', key: '7' };
            if (weight >= th['8']) return { name: '8玉', key: '8' };
            if (weight >= th['9']) return { name: '9玉', key: '9' };
            if (weight >= th['10']) return { name: '10玉', key: '10' };
            if (weight >= th['11']) return { name: '11玉', key: '11' };
            if (weight >= th['12']) return { name: '12玉', key: '12' };
            if (weight >= th['2L']) return { name: '2Lサイズ', key: '2L' };
            if (weight >= th['L']) return { name: 'Lサイズ', key: 'L' };
            if (weight >= th['M']) return { name: 'Mサイズ', key: 'M' };
            if (weight >= th['S']) return { name: 'Sサイズ', key: 'S' };
            return { name: '規格外', key: 'out' };
        }

        function processGrossWeightUpdate(newGross) {
            state.lastGrossWeight = newGross;
            document.getElementById('gross-weight').innerText = newGross.toFixed(1);

            if (!state.isBasketSet) return;

            const diffWeight = state.baseWeight - newGross;

            // 3g以上の減少を検知したときにつまみ出し（減算）とみなす
            if (diffWeight >= 3.0) {
                finalizePickedBerry(newGross, diffWeight);
            }
        }

        function finalizePickedBerry(finalGross, diffWeight) {
            const rank = evaluateRank(diffWeight);
            document.getElementById('removed-weight-display').innerText = diffWeight.toFixed(1);
            
            const badge = document.getElementById('rank-badge');
            badge.innerText = rank.name;
            badge.className = "inline-block px-6 py-2 rounded-2xl font-black text-3xl md:text-4xl shadow-inner transition-all duration-300 border text-white bg-slate-700 border-slate-600";

            playBeep();
            speakText(`${diffWeight.toFixed(1)}グラム、${rank.name}`);
            
            recordLog(diffWeight, rank, state.baseWeight);
            updateStatsUI();

            state.baseWeight = finalGross;
            document.getElementById('base-weight').innerText = state.baseWeight.toFixed(1);
        }

        function setTareBasket() {
            state.baseWeight = state.lastGrossWeight;
            state.isBasketSet = true;
            document.getElementById('base-weight').innerText = state.baseWeight.toFixed(1);
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
            if (state.stats[rank.key] !== undefined) state.stats[rank.key]++;
            else state.stats.out++;
            state.stats.totalCount++;
            state.stats.totalWeight += weight;
            renderLogsUI();
        }

        function undoLastItem() {
            if (state.logs.length === 0) { alert("取り消す履歴がありません。"); return; }
            const last = state.logs.shift();
            state.stats[last.rankKey]--;
            state.stats.totalCount--;
            state.stats.totalWeight -= last.weight;
            state.baseWeight = last.prevBase;
            document.getElementById('base-weight').innerText = state.baseWeight.toFixed(1);
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
                if (el) el.innerText = state.stats[r.key];
            });
            document.getElementById('total-count-badge').innerText = `合計: ${state.stats.totalCount} 個`;
            document.getElementById('total-weight').innerText = state.stats.totalWeight.toFixed(1);
        }

        function resetStats() {
            if(!confirm("集計をリセットしますか？")) return;
            RanksDef.forEach(r => state.stats[r.key] = 0);
            state.stats.totalCount = 0;
            state.stats.totalWeight = 0.0;
            state.logs = [];
            renderLogsUI();
            updateStatsUI();
        }

        function copyResults() {
            let text = "【イチゴ選別結果】\n" + RanksDef.map(r => `${r.name}: ${state.stats[r.key]}個`).join('\n') + `\n合計: ${state.stats.totalCount}個 (${state.stats.totalWeight.toFixed(1)}g)`;
            navigator.clipboard.writeText(text).then(() => alert("コピーしました！"));
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

                // 切断イベントの監視を常時セット
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

        // 【完璧版パーサー】余計な計算を廃し、インデックス4の数値をダイレクトに総重量として直読
        function parseScaleData(value) {
            let bytes = [];
            for (let i = 0; i < value.byteLength; i++) { bytes.push(value.getUint8(i)); }
            
            document.getElementById('raw-bytes-display').innerText = `[${bytes.join(', ')}]`;
            document.getElementById('raw-data-status').innerText = "受信 " + new Date().toLocaleTimeString();

            let weight = 0;
            const len = value.byteLength;

            if (len >= 5) {
                // インデックス4のバイト値をそのまま重量（g）として採用
                let directWeight = value.getUint8(4);
                if (directWeight >= 0 && directWeight < 10000) {
                    weight = directWeight;
                }
            }

            // 取得した重量を画面に反映
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
