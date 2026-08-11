<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NitriScan Web Monitor</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; text-align: center; background-color: #f0f2f5; padding: 20px; }
        .card { background: white; padding: 25px; border-radius: 16px; max-width: 380px; margin: 20px auto; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
        button { background-color: #007bff; color: white; border: none; padding: 14px 28px; font-size: 16px; font-weight: bold; border-radius: 10px; cursor: pointer; transition: 0.2s; }
        button:hover { background-color: #0056b3; }
        .conc-title { font-size: 14px; color: #666; margin-top: 20px; }
        .value-box { font-size: 36px; color: #28a745; font-weight: bold; margin: 10px 0; }
        .rgb-box { background: #f8f9fa; padding: 10px; border-radius: 8px; color: #333; font-size: 14px; font-weight: 500; }
        #status { margin-top: 15px; font-size: 13px; color: #888; }
    </style>
</head>
<body>

<div class="card">
    <h2>🧪 NitriScan Monitor</h2>
    <button onclick="connectBLE()">🔗 เชื่อมต่ออุปกรณ์</button>

    <div class="conc-title">ความเข้มข้นที่คำนวณได้</div>
    <div class="value-box" id="concDisplay">0.00 ppm</div>

    <div class="rgb-box" id="rgbDisplay">R: -- | G: -- | B: --</div>
    <p id="status">สถานะ: ยังไม่ได้เชื่อมต่อ</p>
</div>

<script>
    const SERVICE_UUID = "4fafc201-1fb5-459e-8fcc-c5c9c331914b";
    const CHARACTERISTIC_UUID = "beb5483e-36e1-4688-b7f5-ea07361b26a8";

    async function connectBLE() {
        const statusText = document.getElementById("status");
        try {
            statusText.innerText = "กำลังค้นหาอุปกรณ์...";
            
            // เรียกขอสิทธิ์ค้นหา Bluetooth ผ่านเบราว์เซอร์
            const device = await navigator.bluetooth.requestDevice({
                filters: [{ namePrefix: 'NitriScan' }],
                optionalServices: [SERVICE_UUID]
            });

            statusText.innerText = "กำลังเชื่อมต่อกับ " + device.name + "...";
            const server = await device.gatt.connect();
            const service = await server.getPrimaryService(SERVICE_UUID);
            const characteristic = await service.getCharacteristic(CHARACTERISTIC_UUID);

            // เปิดโหมดรับข้อมูลต่อเนื่อง (Notify)
            await characteristic.startNotifications();
            characteristic.addEventListener('characteristicvaluechanged', (event) => {
                const decoder = new TextDecoder('utf-8');
                const dataString = decoder.decode(event.target.value);
                
                // แยกข้อมูล CSV
                const values = dataString.split(',');
                if (values.length >= 4) {
                    document.getElementById("rgbDisplay").innerText = `R: ${values[0]} | G: ${values[1]} | B: ${values[2]}`;
                    document.getElementById("concDisplay").innerText = values[3] + " ppm";
                }
            });

            statusText.innerText = "🟢 เชื่อมต่อสำเร็จแล้ว!";
            statusText.style.color = "green";

        } catch (error) {
            console.error(error);
            statusText.innerText = "🔴 การเชื่อมต่อล้มเหลว";
            statusText.style.color = "red";
        }
    }
</script>

</body>
</html>
