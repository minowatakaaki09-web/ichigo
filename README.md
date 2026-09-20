<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OBESTスケール 診断モニター</title>
    <style>
        body { font-family: sans-serif; padding: 20px; background: #f0f2f5; color: #333; }
        .card { background: white; padding: 20px; border-radius: 12px; margin-bottom: 15px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        button { background: #ff4757; color: white; border: none; padding: 14px 20px; font-size: 16px; border-radius: 8px; cursor: pointer; width: 100%; font-weight: bold; }
        button:active { background: #e84118; }
        pre { background: #2f3640; color: #f5f6fa; padding: 10px; border-radius: 6px; overflow-x: auto; font-size: 11px; max-height: 180px; }
        .big-weight { font-size: 42px; font-weight: bold; color: #2ed573; text-align: center; margin: 10px 0; }
        .status { font-weight: bold; color: #57606f; margin-top: 8px; text-align: center; }
    </style>
</head>
<body>

    <div class="card">
        <h2>🔌 スケール接続診断</h2>
        <button id="connectBtn">スケールに接続する</button>
        <div class="status" id="status">未接続</div>
    </div>

    <div class="card">
        <h2>⚖️ 測定値モニター</h2>
        <div class="big-weight" id="weightDisplay">-- g</div>
    </div>

    <div class="card">
        <h2>📡 受信生データ (Bytes)</h2>
        <pre id="logArea">ここにデータが流れてきます...</pre>
    </div>

    <div class="card">
        <h2>🛠️ 検出された通信路 (キャラクタリスティック)</h2>
        <pre id="charList">接続するとここに一覧が表示されます...</pre>
    </div>

<script>
    let bluetoothDevice = null;

    const connectBtn = document.getElementById('connectBtn');
    const statusEl = document.getElementById('status');
    const weightDisplay = document.getElementById('weightDisplay');
    const logArea = document.getElementById('logArea');
    const charList = document.getElementById('charList');

    function log(text) {
        console.log(text);
        logArea.textContent += text + "\n";
        logArea.scrollTop = logArea.scrollHeight;
    }

    connectBtn.addEventListener('click', async () => {
        try {
            log("🔍 Bluetoothデバイスをスキャン中...");
            statusEl.textContent = "スキャン中...";
            
            bluetoothDevice = await navigator.bluetooth.requestDevice({
                acceptAllDevices: true,
                optionalServices: [
                    '0000ffe0-0000-1000-8000-00805f9b34fb',
                    '0000fff0-0000-1000-8000-00805f9b34fb',
                    '0000181d-0000-1000-8000-00805f9b34fb',
                    '0000180d-0000-1000-8000-00805f9b34fb',
                    '000018f0-0000-1000-8000-00805f9b34fb'
                ]
            });

            log(`✅ 選択: ${bluetoothDevice.name || '不明なデバイス'}`);
            statusEl.textContent = "GATT接続中...";

            const server = await bluetoothDevice.gatt.connect();
            log("✅ サーバー接続成功！");
            statusEl.textContent = "サービス解析中...";

            const services = await server.getPrimaryServices();
            let summaryText = "";
            let notifyCount = 0;

            for (const service of services) {
                summaryText += `[Service] ${service.uuid}\n`;
                try {
                    const characteristics = await service.getCharacteristics();
                    for (const char of characteristics) {
                        let props = [];
                        if (char.properties.read) props.push('Read');
                        if (char.properties.write) props.push('Write');
                        if (char.properties.notify) props.push('Notify');
                        if (char.properties.indicate) props.push('Indicate');

                        summaryText += `  └ [Char] ${char.uuid} [ ${props.join(', ')} ]\n`;

                        // Notify または Indicate が使えるなら片っ端から購読を試みる
                        if (char.properties.notify || char.properties.indicate) {
                            try {
                                await char.startNotifications();
                                char.addEventListener('characteristicvaluechanged', (e) => {
                                    const val = e.target.value;
                                    const bytes = [];
                                    for (let i = 0; i < val.byteLength; i++) {
                                        bytes.push(val.getUint8(i));
                                    }
                                    log(`📦 [${char.uuid.slice(0,8)}...] Bytes: [${bytes.join(', ')}]`);
                                });
                                notifyCount++;
                                log(`🔔 購読成功: ${char.uuid.slice(0,8)}...`);
                            } catch (err) {
                                log(`⚠️ 購読失敗 (${char.uuid.slice(0,8)}...): ${err.message}`);
                            }
                        }
                    }
                } catch (err) {
                    summaryText += `  └ ❌ エラー: ${err.message}\n`;
                }
            }

            charList.textContent = summaryText;
            statusEl.textContent = `待機中 (購読数: ${notifyCount})。上に物を乗せて！`;
            log(`🚀 準備完了！スケールの上に何かを乗せてみてください。`);

        } catch (error) {
            log(`❌ エラー発生: ${error}`);
            statusEl.textContent = "接続エラー";
        }
    });
</script>
</body>
</html>
