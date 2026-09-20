<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OBEST 完全覚醒モニター</title>
    <style>
        body { font-family: sans-serif; padding: 20px; background: #1e272e; color: #f5f6fa; }
        .card { background: #2f3640; padding: 20px; border-radius: 12px; margin-bottom: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.3); }
        button { background: #e84118; color: white; border: none; padding: 16px 20px; font-size: 18px; border-radius: 8px; cursor: pointer; width: 100%; font-weight: bold; }
        button:active { background: #c23616; }
        pre { background: #192a56; color: #00d2d3; padding: 12px; border-radius: 6px; overflow-x: auto; font-size: 12px; max-height: 220px; }
        .big-weight { font-size: 48px; font-weight: bold; color: #4cd137; text-align: center; margin: 10px 0; }
        .status { font-weight: bold; color: #fbc531; margin-top: 8px; text-align: center; font-size: 16px; }
    </style>
</head>
<body>

    <div class="card">
        <h2>🔌 OBEST 強制覚醒＆接続</h2>
        <button id="connectBtn">スケールを叩き起こして接続する</button>
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

<script>
    let bluetoothDevice = null;

    const connectBtn = document.getElementById('connectBtn');
    const statusEl = document.getElementById('status');
    const weightDisplay = document.getElementById('weightDisplay');
    const logArea = document.getElementById('logArea');

    function log(text) {
        console.log(text);
        logArea.textContent += text + "\n";
        logArea.scrollTop = logArea.scrollHeight;
    }

    connectBtn.addEventListener('click', async () => {
        try {
            log("🔍 OBESTスケールをスキャン中...");
            statusEl.textContent = "スキャン中...";
            
            // よく使われるサービスUUIDを網羅
            bluetoothDevice = await navigator.bluetooth.requestDevice({
                acceptAllDevices: true,
                optionalServices: [
                    '0000ffe0-0000-1000-8000-00805f9b34fb',
                    '0000fff0-0000-1000-8000-00805f9b34fb',
                    '0000ff00-0000-1000-8000-00805f9b34fb',
                    '0000181d-0000-1000-8000-00805f9b34fb',
                    '0000180d-0000-1000-8000-00805f9b34fb',
                    '000018f0-0000-1000-8000-00805f9b34fb',
                    '0000fff1-0000-1000-8000-00805f9b34fb',
                    '0000ffe1-0000-1000-8000-00805f9b34fb'
                ]
            });

            log(`✅ 選択: ${bluetoothDevice.name || 'OBESTスケール'}`);
            statusEl.textContent = "GATT接続中...";

            const server = await bluetoothDevice.gatt.connect();
            log("✅ サーバー接続成功！");
            statusEl.textContent = "サービス解析＆起動中...";

            const services = await server.getPrimaryServices();
            
            for (const service of services) {
                log(`[Service] ${service.uuid}`);
                try {
                    const characteristics = await service.getCharacteristics();
                    for (const char of characteristics) {
                        let props = [];
                        if (char.properties.read) props.push('Read');
                        if (char.properties.write) props.push('Write');
                        if (char.properties.writeWithoutResponse) props.push('WriteNoResp');
                        if (char.properties.notify) props.push('Notify');
                        if (char.properties.indicate) props.push('Indicate');

                        log(`  └ [Char] ${char.uuid} [ ${props.join(', ')} ]`);

                        // 1. データの受信購読を設定
                        if (char.properties.notify || char.properties.indicate) {
                            try {
                                await char.startNotifications();
                                char.addEventListener('characteristicvaluechanged', (e) => {
                                    const val = e.target.value;
                                    const bytes = [];
                                    for (let i = 0; i < val.byteLength; i++) {
                                        bytes.push(val.getUint8(i));
                                    }
                                    log(`📦 爆速受信! Bytes: [${bytes.join(', ')}]`);
                                    
                                    // 簡易的な数値表示のテスト（仮で配列の長さを出すなど）
                                    weightDisplay.textContent = `${bytes.length} bytes`;
                                });
                                log(`🔔 購読成功: ${char.uuid.slice(0,8)}...`);
                            } catch (err) {
                                log(`⚠️ 購読失敗: ${err.message}`);
                            }
                        }

                        // 2. 書き込みができる場所があれば、スケールを目覚めさせるコマンドを連打する
                        if (char.properties.write || char.properties.writeWithoutResponse) {
                            try {
                                const wakeCmds = [
                                    new Uint8Array([0x03, 0x01, 0x01]),
                                    new Uint8Array([0xAA, 0x01, 0x00]),
                                    new Uint8Array([0x55, 0x01, 0x01]),
                                    new Uint8Array([0x01, 0x03, 0x00, 0x01, 0x00, 0x10])
                                ];
                                for (const cmd of wakeCmds) {
                                    try {
                                        if (char.properties.write) {
                                            await char.writeValue(cmd);
                                        } else {
                                            await char.writeValueWithoutResponse(cmd);
                                        }
                                        log(`⚡ 起動コマンド送信: ${char.uuid.slice(0,8)}...`);
                                    } catch(e) {}
                                }
                            } catch (err) {}
                        }
                    }
                } catch (err) {
                    log(`  └ ❌ エラー: ${err.message}`);
                }
            }

            statusEl.textContent = "起動完了！スケールに乗せてみて";
            log(`🚀 準備完了！スケールの上に何かを乗せてみてください。`);

        } catch (error) {
            log(`❌ エラー発生: ${error}`);
            statusEl.textContent = "接続エラー";
        }
    });
</script>
</body>
</html>
