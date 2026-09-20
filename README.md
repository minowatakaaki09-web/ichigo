<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>セルフカット・スマートミラー</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
        }
        body {
            background-color: #000;
            color: #fff;
            overflow: hidden;
            width: 100vw;
            height: 100vh;
        }
        #app-container {
            position: relative;
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        #webcam {
            position: absolute;
            width: 100%;
            height: 100%;
            object-fit: cover;
            transform: scaleX(-1); /* 鏡モード（左右反転） */
        }
        #overlay-canvas {
            position: absolute;
            width: 100%;
            height: 100%;
            pointer-events: none;
            transform: scaleX(-1);
        }
        /* 操作パネル */
        .control-panel {
            position: absolute;
            bottom: 30px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(15, 15, 15, 0.9);
            backdrop-filter: blur(15px);
            padding: 16px 20px;
            border-radius: 24px;
            display: flex;
            flex-direction: column;
            gap: 12px;
            width: 92%;
            max-width: 400px;
            z-index: 10;
            border: 1px solid rgba(255, 255, 255, 0.3);
            box-shadow: 0 10px 30px rgba(0,0,0,0.8);
        }
        .control-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            font-size: 14px;
        }
        .control-row label {
            color: #aaa;
        }
        .control-row span {
            color: #00ffcc;
            font-weight: bold;
            font-size: 15px;
        }
        input[type="range"] {
            width: 100%;
            height: 8px;
            margin-top: 6px;
            accent-color: #00ffcc;
        }
        .btn-group {
            display: flex;
            gap: 8px;
        }
        button.mode-btn {
            flex: 1;
            padding: 12px 0;
            border: none;
            border-radius: 12px;
            font-weight: bold;
            cursor: pointer;
            font-size: 14px;
            transition: 0.2s;
        }
        .btn-secondary {
            background: rgba(255, 255, 255, 0.15);
            color: #fff;
        }
        .btn-active {
            background: #00ffcc;
            color: #000;
        }
        /* カメラ強制起動画面 */
        #start-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: #111;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 30;
            gap: 16px;
            padding: 24px;
            text-align: center;
        }
        #start-btn {
            background: #00ffcc;
            color: #000;
            font-size: 18px;
            padding: 16px 40px;
            border-radius: 30px;
            border: none;
            font-weight: bold;
            box-shadow: 0 4px 20px rgba(0,255,204,0.4);
        }
    </style>
</head>
<body>

<div id="app-container">
    <!-- カメラ起動ボタン -->
    <div id="start-screen">
        <h2>セルフカット・スマートミラー</h2>
        <p style="color: #aaa; font-size: 13px; line-height: 1.5;">下のボタンを押すとカメラが起動します<br>（許可ポップアップが出たら「許可」を押してください）</p>
        <button id="start-btn" onclick="initCamera()">ミラーを開始する</button>
    </div>

    <video id="webcam" autoplay playsinline muted></video>
    <canvas id="overlay-canvas"></canvas>

    <!-- 操作パネル -->
    <div class="control-panel" id="panel" style="display: none;">
        <div class="control-row">
            <label>スタイル</label>
            <span id="mode-label">フェード (ロー)</span>
        </div>
        <div class="btn-group">
            <button class="mode-btn btn-active" id="btn-low" onclick="setMode('fadeLow')">ロー</button>
            <button class="mode-btn btn-secondary" id="btn-high" onclick="setMode('fadeHigh')">ハイ</button>
            <button class="mode-btn btn-secondary" id="btn-two" onclick="setMode('twoblock')">ツーブロ</button>
        </div>

        <div>
            <div class="control-row">
                <label>高さ調整</label>
                <span id="pos-val">50%</span>
            </div>
            <input type="range" id="posSlider" min="10" max="90" value="50" oninput="updatePosition(this.value)">
        </div>
    </div>
</div>

<script>
    const video = document.getElementById('webcam');
    const canvas = document.getElementById('overlay-canvas');
    const ctx = canvas.getContext('2d');
    const startScreen = document.getElementById('start-screen');
    const panel = document.getElementById('panel');
    
    let currentMode = 'fadeLow';
    let linePosition = 0.5;

    async function initCamera() {
        try {
            const constraints = {
                video: {
                    facingMode: 'user',
                    width: { ideal: 1280 },
                    height: { ideal: 720 }
                },
                audio: false
            };
            const stream = await navigator.mediaDevices.getUserMedia(constraints);
            video.srcObject = stream;
            await video.play();
            
            startScreen.style.display = 'none';
            panel.style.display = 'flex';
            
            resizeCanvas();
            requestAnimationFrame(renderLoop);
        } catch (err) {
            alert('カメラの起動に失敗しました。Safariで開いているか、カメラの権限を確認してください。');
            console.error(err);
        }
    }

    function resizeCanvas() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
    }
    window.addEventListener('resize', resizeCanvas);

    function setMode(mode) {
        currentMode = mode;
        const label = document.getElementById('mode-label');
        
        document.getElementById('btn-low').className = 'mode-btn btn-secondary';
        document.getElementById('btn-high').className = 'mode-btn btn-secondary';
        document.getElementById('btn-two').className = 'mode-btn btn-secondary';

        if (mode === 'fadeLow') {
            label.innerText = "フェード (ロー)";
            document.getElementById('btn-low').className = 'mode-btn btn-active';
        } else if (mode === 'fadeHigh') {
            label.innerText = "フェード (ハイ)";
            document.getElementById('btn-high').className = 'mode-btn btn-active';
        } else if (mode === 'twoblock') {
            label.innerText = "ツーブロック分け目";
            document.getElementById('btn-two').className = 'mode-btn btn-active';
        }
    }

    function updatePosition(val) {
        linePosition = val / 100;
        document.getElementById('pos-val').innerText = val + '%';
    }

    function renderLoop() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        const w = canvas.width;
        const h = canvas.height;
        const targetY = h * linePosition;

        ctx.lineWidth = 4;

        if (currentMode === 'fadeLow' || currentMode === 'fadeHigh') {
            ctx.strokeStyle = '#00ffcc';
            ctx.beginPath();
            ctx.moveTo(w * 0.05, targetY);
            ctx.lineTo(w * 0.95, targetY);
            ctx.stroke();

            ctx.strokeStyle = 'rgba(0, 255, 204, 0.45)';
            ctx.beginPath();
            ctx.arc(w / 2, targetY - 45, w * 0.38, 0, Math.PI);
            ctx.stroke();
        } else if (currentMode === 'twoblock') {
            ctx.strokeStyle = '#ff00aa';
            ctx.beginPath();
            ctx.moveTo(w * 0.12, targetY + 30);
            ctx.quadraticCurveTo(w * 0.5, targetY - 70, w * 0.88, targetY + 30);
            ctx.stroke();
        }

        // 耳まわりのセーフティゾーン
        ctx.strokeStyle = '#00ff00';
        ctx.lineWidth = 3;
        ctx.setLineDash([6, 6]);
        ctx.beginPath();
        ctx.arc(w * 0.22, h * 0.52, 55, 0, Math.PI * 2);
        ctx.stroke();
        ctx.beginPath();
        ctx.arc(w * 0.78, h * 0.52, 55, 0, Math.PI * 2);
        ctx.stroke();
        ctx.setLineDash([]);

        requestAnimationFrame(renderLoop);
    }
</script>

</body>
</html>
