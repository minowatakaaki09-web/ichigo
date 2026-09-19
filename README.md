<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>イチゴ選別 分太AI Pro</title>
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
                    イチゴ減算秤 <span class="bg-red-500/20 text-red-400 text-xs px-2 py-0.5 rounded-full border border-red-500/30">分太AI Pro</span>
                </h1>
                <p class="text-xs text-slate-400">リアルタイム減算選別・音声ナビゲーション</p>
            </div>
        </div>
        <div class="flex items-center gap-2">
            <button id="btn-connect" onclick="connectScale()" class="bg-emerald-600 hover:bg-emerald-500 active:scale-95 text-white text-xs font-bold px-3 py-2 rounded-lg transition duration-200 flex items-center gap-2 shadow-md">
                <i class="fa-brands fa-bluetooth-b"></i> <span id="conn-text">スケール接続</span>
            </button>
        </div>
    </header>

    <!-- MAIN CONTENT -->
    <main class="flex-1 max-w-4xl w-full mx-auto p-3 md:p-6 grid grid-cols-1 md:grid-cols-12 gap-4">

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
                <div class="my-6 text-center">
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

                <!-- ACTION BUTTONS -->
                <div class="grid grid-cols-2 gap-3 w-full mt-5">
                    <button onclick="setTareBasket()" class="bg-red-600 hover:bg-red-500 active:scale-95 text-white font-bold py-3.5 px-4 rounded-xl shadow-lg transition duration-150 text-sm md:text-base flex items-center justify-center gap-2">
                        <i class="fa-solid fa-basket-shopping"></i> カゴセット (基準更新)
                    </button>
                    <button onclick="toggleSimPanel()" class="bg-slate-700 hover:bg-slate-600 active:scale-95 text-slate-200 font-bold py-3.5 px-4 rounded-xl shadow transition duration-150 text-sm md:text-base flex items-center justify-center gap-2">
                        <i class="fa-solid fa-gamepad"></i> テスト入力
                    </button>
                </div>
            </div>

            <!-- SIMULATOR PANEL -->
            <div id="sim-panel" class="bg-slate-800/90 rounded-2xl border border-amber-500/30 p-4 shadow-lg hidden">
                <div class="text-xs font-bold text-amber-400 mb-2 flex items-center justify-between">
                    <span><i class="fa-solid fa-flask"></i> 模擬テスト (クリックで減算動作・音声テスト)</span>
                    <button onclick="toggleSimPanel()" class="text-slate-400 hover:text-white"><i class="fa-solid fa-xmark"></i></button>
                </div>
                <div class="grid grid-cols-5 gap-2">
                    <button onclick="simTakeBerry(12.5)" class="bg-slate-700 hover:bg-slate-600 text-white font-bold py-2 rounded-lg text-xs">12.5g<br><span class="text-[10px] text-slate-400">S</span></button>
                    <button onclick="simTakeBerry(18.0)" class="bg-slate-700 hover:bg-slate-600 text-white font-bold py-2 rounded-lg text-xs">18.0g<br><span class="text-[10px] text-slate-400">M</span></button>
                    <button onclick="simTakeBerry(24.5)" class="bg-slate-700 hover:bg-slate-600 text-white font-bold py-2 rounded-lg text-xs">24.5g<br><span class="text-[10px] text-slate-400">L</span></button>
                    <button onclick="simTakeBerry(32.0)" class="bg-slate-700 hover:bg-slate-600 text-white font-bold py-2 rounded-lg text-xs">32.0g<br><span class="text-[10px] text-slate-400">2L</span></button>
                    <button onclick="simTakeBerry(45.0)" class="bg-slate-700 hover:bg-slate-600 text-white font-bold py-2 rounded-lg text-xs">45.0g<br><span class="text-[10px] text-slate-400">3L</span></button>
                </div>
            </div>

            <!-- RANK CONFIG -->
            <div class="bg-slate-800/80 backdrop-blur-md rounded-2xl border border-slate-700/80 p-4 shadow-lg">
                <h3 class="text-sm font-bold text-slate-300 mb-3 flex items-center gap-2">
                    <i class="fa-solid fa-sliders text-red-400"></i> 選別階級・閾値設定 (グラム)
                </h3>
                <div class="grid grid-cols-5 gap-2 text-center text-xs">
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700">
                        <span class="block text-slate-400 font-bold mb-1">S</span>
                        <input type="number" id="th-s" value="10.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600 focus:outline-none focus:border-red-500">
                        <span class="text-[10px] text-slate-500">g〜</span>
                    </div>
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700">
                        <span class="block text-slate-400 font-bold mb-1">M</span>
                        <input type="number" id="th-m" value="15.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600 focus:outline-none focus:border-red-500">
                        <span class="text-[10px] text-slate-500">g〜</span>
                    </div>
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700">
                        <span class="block text-slate-400 font-bold mb-1">L</span>
                        <input type="number" id="th-l" value="20.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600 focus:outline-none focus:border-red-500">
                        <span class="text-[10px] text-slate-500">g〜</span>
                    </div>
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700">
                        <span class="block text-slate-400 font-bold mb-1">2L</span>
                        <input type="number" id="th-2l" value="28.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600 focus:outline-none focus:border-red-500">
                        <span class="text-[10px] text-slate-500">g〜</span>
                    </div>
                    <div class="bg-slate-900/50 p-2 rounded-xl border border-slate-700">
                        <span class="block text-slate-400 font-bold mb-1">3L</span>
                        <input type="number" id="th-3l" value="38.0" step="0.5" class="w-full bg-slate-800 text-center font-bold text-white rounded p-1 border border-slate-600 focus:outline-none focus:border-red-500">
                        <span class="text-[10px] text-slate-500">g〜</span>
                    </div>
                </div>
            </div>

        </section>

        <!-- RIGHT PANEL -->
        <section class="md:col-span-5 flex flex-col gap-4">

            <!-- STATS SUMMARY -->
            <div class="bg-slate-800/80 backdrop-blur-md rounded-2xl border border-slate-700/80 p-4 shadow-lg">
                <div class="flex justify-between items-center mb-3">
                    <h3 class="text-sm font-bold text-slate-300 flex items-center gap-2">
                        <i class="fa-solid fa-chart-pie text-emerald-400"></i> 本日の選別リアルタイム集計
                    </h3>
                    <span id="total-count-badge" class="bg-emerald-500/20 text-emerald-300 text-xs font-bold px-2.5 py-0.5 rounded-full border border-emerald-500/30">
                        合計: 0 個
                    </span>
                </div>

                <div class="grid grid-cols-3 gap-2 text-center text-xs">
                    <div class="bg-slate-900/60 p-2 rounded-lg border border-slate-700">
                        <div class="text-slate-400">3L</div>
                        <div id="count-3l" class="text-lg font-bold text-amber-400">0</div>
                    </div>
                    <div class="bg-slate-900/60 p-2 rounded-lg border border-slate-700">
                        <div class="text-slate-400">2L</div>
                        <div id="count-2l" class="text-lg font-bold text-emerald-400">0</div>
                    </div>
                    <div class="bg-slate-900/60 p-2 rounded-lg border border-slate-700">
                        <div class="text-slate-400">L</div>
                        <div id="count-l" class="text-lg font-bold text-blue-400">0</div>
                    </div>
                    <div class="bg-slate-900/60 p-2 rounded-lg border border-slate-700">
                        <div class="text-slate-400">M</div>
                        <div id="count-m" class="text-lg font-bold text-indigo-400">0</div>
                    </div>
                    <div class="bg-slate-900/60 p-2 rounded-lg border border-slate-700">
                        <div class="text-slate-400">S</div>
                        <div id="count-s" class="text-lg font-bold text-purple-400">0</div>
                    </div>
                    <div class="bg-slate-900/60 p-2 rounded-lg border border-slate-700">
                        <div class="text-slate-400">規格外</div>
                        <div id="count-out" class="text-lg font-bold text-slate-400">0</div>
                    </div>
                </div>

                <div class="mt-3 pt-2 border-t border-slate-700/80 flex justify-between items-center text-xs text-slate-400">
                    <span>総選別重量: <strong id="total-weight" class="text-slate-200">0.0</strong> g</span>
                    <button onclick="resetStats()" class="text-slate-500 hover:text-red-400 transition"><i class="fa-solid fa-rotate-right"></i> リセット</button>
                </div>
            </div>

            <!-- LOG HISTORY -->
            <div class="bg-slate-800/80 backdrop-blur-md rounded-2xl border border-slate-700/80 p-4 shadow-lg flex-1 flex flex-col min-h-[180px]">
                <h3 class="text-sm font-bold text-slate-300 mb-2 flex items-center gap-2">
                    <i class="fa-solid fa-list-ol text-blue-400"></i> 選別履歴ログ
                </h3>
                <div id="log-container" class="flex-1 overflow-y-auto max-h-[220px] custom-scrollbar space-y-1.5 pr-1 text-xs">
                    <div class="text-slate-500 text-center py-6">選別データはまだありません</div>
                </div>
            </div>

            <!-- GEMINI AI ANALYSIS INTEGRATION -->
            <div class="bg-slate-800/80 backdrop-blur-md rounded-2xl border border-slate-700/80 p-4 shadow-lg">
                <h3 class="text-sm font-bold text-slate-300 mb-2 flex items-center justify-between">
                    <span class="flex items-center gap-2"><i class="fa-solid fa-wand-magic-sparkles text-purple-400"></i> AI収穫分析</span>
                    <span class="text-[10px] bg-purple-500/20 text-purple-300 px-2 py-0.5 rounded border border-purple-500/30">Gemini</span>
                </h3>
                <input type="password" id="gemini-api-key" placeholder="Gemini API Key を入力 (任意)" class="w-full bg-slate-900 border border-slate-700 rounded-lg p-2 text-xs text-slate-200 mb-2 focus:outline-none focus:border-purple-500">
                <button onclick="generateAiReport()" class="w-full bg-purple-600 hover:bg-purple-500 active:scale-95 text-white font-bold py-2 px-3 rounded-lg text-xs transition duration-150 flex items-center justify-center gap-2">
                    <i class="fa-solid fa-brain"></i> 本日の選別結果をAI分析
                </button>
                <div id="ai-report-output" class="mt-2 p-2.5 bg-slate-900/80 rounded-lg border border-slate-700/50 text-xs text-slate-300 max-h-32 overflow-y-auto custom-scrollbar whitespace-pre-wrap leading-relaxed">
                    APIキーを入力してボタンを押すと、収穫バランスのアドバイスをAIが生成します。
                </div>
            </div>

        </section>

    </main>

    <!-- SCRIPT -->
    <script>
        const state = {
            lastGrossWeight: 0.0,
            baseWeight: 0.0,
            isBasketSet: false,
            bluetoothDevice: null,
            stats: { '3L': 0, '2L': 0, 'L': 0, 'M': 0, 'S': 0, out: 0, totalCount: 0, totalWeight: 0.0 },
            logs: []
        };

        const synth = window.speechSynthesis;
        function speakText(text) {
            if (!synth) return;
            if (synth.speaking) synth.cancel();
            const utter = new SpeechSynthesisUtterance(text);
            utter.lang = 'ja-JP';
            utter.rate = 1.2;
            utter.pitch = 1.0;
            synth.speak(utter);
        }

        function evaluateRank(weight) {
            const thS = parseFloat(document.getElementById('th-s').value) || 10.0;
            const thM = parseFloat(document.getElementById('th-m').value) || 15.0;
            const thL = parseFloat(document.getElementById('th-l').value) || 20.0;
            const th2L = parseFloat(document.getElementById('th-2l').value) || 28.0;
            const th3L = parseFloat(document.getElementById('th-3l').value) || 38.0;

            if (weight < thS) return { name: '規格外', key: 'out', bg: 'bg-slate-700', color: 'text-slate-300', speech: '規格外' };
            if (weight < thM) return { name: 'Sサイズ', key: 'S', bg: 'bg-purple-900/80', color: 'text-purple-300', speech: 'エス' };
            if (weight < thL) return { name: 'Mサイズ', key: 'M', bg: 'bg-indigo-900/80', color: 'text-indigo-300', speech: 'エム' };
            if (weight < th2L) return { name: 'Lサイズ', key: 'L', bg: 'bg-blue-900/80', color: 'text-blue-300', speech: 'エル' };
            if (weight < th3L) return { name: '2Lサイズ', key: '2L', bg: 'bg-emerald-900/80', color: 'text-emerald-300', speech: 'ニエル' };
            return { name: '3Lサイズ', key: '3L', bg: 'bg-amber-900/80', color: 'text-amber-300', speech: 'サンエル' };
        }

        function processGrossWeightUpdate(newGross) {
            state.lastGrossWeight = newGross;
            document.getElementById('gross-weight').innerText = newGross.toFixed(1);

            if (!state.isBasketSet) return;

            const diffWeight = state.baseWeight - newGross;

            if (diffWeight >= 5.0) {
                const rank = evaluateRank(diffWeight);

                document.getElementById('removed-weight-display').innerText = diffWeight.toFixed(1);
                const badge = document.getElementById('rank-badge');
                badge.innerText = rank.name;
                badge.className = `inline-block px-6 py-2 rounded-2xl font-black text-3xl md:text-4xl shadow-inner transition-all duration-300 border text-white ${rank.bg} ${rank.color}`;

                speakText(`${diffWeight.toFixed(0)}グラム、${rank.speech}`);

                recordLog(diffWeight, rank);
                updateStatsUI();

                state.baseWeight = newGross;
                document.getElementById('base-weight').innerText = state.baseWeight.toFixed(1);
            }
        }

        function setTareBasket() {
            state.baseWeight = state.lastGrossWeight;
            state.isBasketSet = true;
            document.getElementById('base-weight').innerText = state.baseWeight.toFixed(1);
            document.getElementById('removed-weight-display').innerText = "0.0";
            
            const badge = document.getElementById('rank-badge');
            badge.innerText = "準備完了";
            badge.className = "inline-block px-6 py-2 rounded-2xl font-black text-3xl md:text-4xl shadow-inner transition-all duration-300 bg-emerald-900/80 text-emerald-300 border border-emerald-500/50";

            speakText("カゴを設定しました。");
        }

        function recordLog(weight, rank) {
            const timeStr = new Date().toLocaleTimeString('ja-JP', { hour: '2-digit', minute: '2-digit', second: '2-digit' });
            state.logs.unshift({ time: timeStr, weight: weight, rankName: rank.name });
            if (state.stats[rank.key] !== undefined) state.stats[rank.key]++;
            else state.stats.out++;
            state.stats.totalCount++;
            state.stats.totalWeight += weight;

            const container = document.getElementById('log-container');
            container.innerHTML = state.logs.map(l => `
                <div class="flex justify-between items-center bg-slate-900/50 px-3 py-1.5 rounded border border-slate-700/50">
                    <span class="text-slate-500 font-mono">${l.time}</span>
                    <span class="font-bold text-slate-200">${l.rankName}</span>
                    <span class="font-mono text-emerald-400 font-bold">${l.weight.toFixed(1)} g</span>
                </div>
            `).join('');
        }

        function updateStatsUI() {
            document.getElementById('count-3l').innerText = state.stats['3L'];
            document.getElementById('count-2l').innerText = state.stats['2L'];
            document.getElementById('count-l').innerText = state.stats['L'];
            document.getElementById('count-m').innerText = state.stats['M'];
            document.getElementById('count-s').innerText = state.stats['S'];
            document.getElementById('count-out').innerText = state.stats.out;
            document.getElementById('total-count-badge').innerText = `合計: ${state.stats.totalCount} 個`;
            document.getElementById('total-weight').innerText = state.stats.totalWeight.toFixed(1);
        }

        function resetStats() {
            if(!confirm("選別集計データをリセットしますか？")) return;
            state.stats = { '3L': 0, '2L': 0, 'L': 0, 'M': 0, 'S': 0, out: 0, totalCount: 0, totalWeight: 0.0 };
            state.logs = [];
            document.getElementById('log-container').innerHTML = '<div class="text-slate-500 text-center py-6">選別データはまだありません</div>';
            updateStatsUI();
        }

        async function connectScale() {
            try {
                document.getElementById('conn-text').innerText = "検索中...";
                state.bluetoothDevice = await navigator.bluetooth.requestDevice({
                    acceptAllDevices: true,
                    optionalServices: ['0000181d-0000-1000-8000-00805f9b34fb', '0000181b-0000-1000-8000-00805f9b34fb']
                });
                await state.bluetoothDevice.gatt.connect();
                document.getElementById('conn-text').innerText = "接続済み";
                document.getElementById('status-indicator').className = "w-2.5 h-2.5 rounded-full bg-emerald-500 inline-block animate-pulse";
                document.getElementById('status-text').innerText = "スケールオンライン";
                speakText("Bluetooth接続完了");
            } catch (error) {
                document.getElementById('conn-text').innerText = "接続失敗";
                alert("接続エラー: " + error.message);
            }
        }

        function simTakeBerry(weight) {
            if (!state.isBasketSet) simSetBasket();
            processGrossWeightUpdate(state.lastGrossWeight - weight);
        }
        function simSetBasket() {
            state.lastGrossWeight = 2000.0;
            setTareBasket();
        }
        function toggleSimPanel() {
            document.getElementById('sim-panel').classList.toggle('hidden');
        }

        async function generateAiReport() {
            const apiKey = document.getElementById('gemini-api-key').value;
            if (!apiKey) { alert("API Key を入力してください。"); return; }
            const output = document.getElementById('ai-report-output');
            output.innerText = "分析中...";
            const prompt = `イチゴ選別データ分析:\n- 3L: ${state.stats['3L']}\n- 2L: ${state.stats['2L']}\n- L: ${state.stats['L']}\n- M: ${state.stats['M']}\n- S: ${state.stats['S']}\n- 規格外: ${state.stats.out}\n- 総重量: ${state.stats.totalWeight.toFixed(1)}g`;
            try {
                const res = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${apiKey}`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ contents: [{ parts: [{ text: prompt }] }] })
                });
                const data = await res.json();
                output.innerText = data.candidates[0].content.parts[0].text;
            } catch (e) {
                output.innerText = "エラー: " + e.message;
            }
        }
    </script>
</body>
</html>