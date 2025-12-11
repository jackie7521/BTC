# BTC
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SMC 凱利倉位計算機 - GitHub Pages 版</title>
    <style>
        /* CSS 樣式 (深色主題) */
        :root {
            --bg-color: #121212;
            --card-bg: #1e1e1e;
            --text-primary: #e0e0e0;
            --text-secondary: #a0a0a0;
            --accent-color: #00bcd4; /* SMC Blue */
            --bull-color: #00e676;
            --bear-color: #ff5252;
            --input-bg: #2c2c2c;
            --border-color: #333;
        }

        body {
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-primary);
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
        }

        .container {
            background-color: var(--card-bg);
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            width: 100%;
            max-width: 500px;
            border: 1px solid var(--border-color);
        }

        h1 {
            text-align: center;
            color: var(--accent-color);
            font-size: 1.5rem;
            margin-bottom: 20px;
            letter-spacing: 1px;
        }
        
        .header-subtitle {
            text-align: center;
            color: var(--text-secondary);
            font-size: 0.85rem;
            margin-bottom: 25px;
        }

        .input-group {
            margin-bottom: 15px;
        }

        label {
            display: block;
            font-size: 0.9rem;
            color: var(--text-secondary);
            margin-bottom: 5px;
        }

        input[type="number"] {
            width: 100%;
            padding: 12px;
            background-color: var(--input-bg);
            border: 1px solid var(--border-color);
            border-radius: 6px;
            color: var(--text-primary);
            font-size: 1rem;
            box-sizing: border-box;
        }

        input[type="number"]:focus {
            outline: none;
            border-color: var(--accent-color);
        }

        .kelly-options {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }

        .kelly-btn {
            flex: 1;
            padding: 10px;
            background-color: var(--input-bg);
            border: 1px solid var(--border-color);
            color: var(--text-secondary);
            border-radius: 6px;
            cursor: pointer;
            transition: all 0.3s ease;
            font-size: 0.9rem;
        }

        .kelly-btn.active {
            background-color: var(--accent-color);
            color: #000;
            font-weight: bold;
            border-color: var(--accent-color);
        }

        .btn-calculate {
            width: 100%;
            padding: 15px;
            background: linear-gradient(135deg, #00bcd4, #008ba3);
            color: white;
            border: none;
            border-radius: 6px;
            font-size: 1.1rem;
            font-weight: bold;
            cursor: pointer;
            transition: transform 0.1s ease;
            margin-bottom: 15px;
        }

        .btn-calculate:active {
            transform: scale(0.98);
        }

        .result-area {
            margin-top: 25px;
            padding-top: 20px;
            border-top: 1px solid var(--border-color);
            display: none;
        }

        .result-row {
            display: flex;
            justify-content: space-between;
            margin-bottom: 10px;
            font-size: 1rem;
        }

        .result-value {
            font-weight: bold;
            font-family: 'Courier New', monospace;
        }

        .highlight-green { color: var(--bull-color); }
        .highlight-red { color: var(--bear-color); }
        .highlight-blue { color: var(--accent-color); }

        .warning-box {
            background-color: rgba(255, 82, 82, 0.1);
            border: 1px solid var(--bear-color);
            color: var(--bear-color);
            padding: 10px;
            border-radius: 6px;
            margin-top: 15px;
            text-align: center;
            font-size: 0.9rem;
            display: none;
        }

        .info-tag {
            font-size: 0.8rem;
            color: var(--text-secondary);
            text-align: right;
            margin-top: 5px;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>SMC 凱利倉位計算機</h1>
    <p class="header-subtitle">基於盈虧比和勝率計算最佳風險暴露 </p>

    <div class="input-group">
        <label for="capital">本金 (USDT)</label>
        <input type="number" id="capital" placeholder="例如: 300">
    </div>

    <div class="input-group">
        <label for="entry">入場點位 (Entry Price)</label>
        <input type="number" id="entry" placeholder="例如: 96390.4">
    </div>

    <div class="input-group">
        <label for="sl">止損點位 (Stop Loss)</label>
        <input type="number" id="sl" placeholder="例如: 100239.2">
    </div>

    <div class="input-group">
        <label for="tp">止盈點位 (Take Profit)</label>
        <input type="number" id="tp" placeholder="例如: 86565.9">
    </div>

    <div class="input-group">
        <label for="winrate">勝率 (%)</label>
        <input type="number" id="winrate" placeholder="例如: 45">
    </div>

    <label>凱利模式選擇</label>
    <div class="kelly-options">
        <button class="kelly-btn" id="btn-full" onclick="setKelly(1)">全凱利</button>
        <button class="kelly-btn active" id="btn-half" onclick="setKelly(0.5)">半凱利</button>
        <button class="kelly-btn" id="btn-quarter" onclick="setKelly(0.25)">1/4 凱利</button>
    </div>

    <button class="btn-calculate" onclick="calculate()">計算倉位</button>

    <div id="warning" class="warning-box"></div>

    <div id="result" class="result-area">
        <div class="result-row">
            <span>交易方向</span>
            <span id="direction" class="result-value">--</span>
        </div>
        <div class="result-row">
            <span>盈虧比 (R:R)</span>
            <span id="rr_ratio" class="result-value">--</span>
        </div>
        <div class="result-row">
            <span>凱利建議風險比例</span>
            <span id="kelly_risk_percent" class="result-value highlight-blue">--</span>
        </div>
        <hr style="border-color: #333; opacity: 0.3;">
        <div class="result-row">
            <span>止損金額 (最大風險)</span>
            <span id="risk_amt" class="result-value highlight-red">--</span>
        </div>
        <div class="result-row">
            <span>止盈金額 (預期報酬)</span>
            <span id="profit_amt" class="result-value highlight-green">--</span>
        </div>
        <hr style="border-color: #333; opacity: 0.3;">
        <div class="result-row">
            <span>**建議倉位總價值** (USDT)</span>
            <span id="position_value" class="result-value">--</span>
        </div>
        <div class="result-row">
            <span>**建議槓桿倍數**</span>
            <span id="leverage" class="result-value highlight-blue">--</span>
        </div>
        <div class="result-row">
            <span>*資產單位數量 (例如 BTC 顆數)*</span>
            <span id="position_size" class="result-value">--</span>
        </div>
        <div class="info-tag">* 槓桿 = 倉位總價值 / 本金</div>
    </div>
</div>

<script>
    let kellyMultiplier = 0.5; // 預設半凱利

    function setKelly(multiplier) {
        kellyMultiplier = multiplier;
        // 更新按鈕樣式
        const buttons = document.querySelectorAll('.kelly-btn');
        buttons.forEach(btn => btn.classList.remove('active'));
        
        // 找到對應的按鈕並設置 active 樣式
        if (multiplier === 1) document.getElementById('btn-full').classList.add('active');
        if (multiplier === 0.5) document.getElementById('btn-half').classList.add('active');
        if (multiplier === 0.25) document.getElementById('btn-quarter').classList.add('active');
        
        // 如果已經計算過，重新計算
        if(document.getElementById('result').style.display === 'block') {
            calculate();
        }
    }

    function calculate() {
        // 1. 取得輸入值
        const capital = parseFloat(document.getElementById('capital').value);
        const entry = parseFloat(document.getElementById('entry').value);
        const sl = parseFloat(document.getElementById('sl').value);
        const tp = parseFloat(document.getElementById('tp').value);
        const winrate = parseFloat(document.getElementById('winrate').value) / 100;

        const warningBox = document.getElementById('warning');
        const resultArea = document.getElementById('result');

        // 基本驗證
        if (isNaN(capital) || isNaN(entry) || isNaN(sl) || isNaN(tp) || isNaN(winrate)) {
            warningBox.style.display = 'block';
            warningBox.innerHTML = `⚠️ **錯誤：** 請填寫所有數值欄位！`;
            resultArea.style.display = 'none';
            return;
        }

        // 2. 判斷方向和計算價差
        let isLong = tp > entry;
        let directionText = "無效方向";
        let profitDiff = 0;
        let lossDiff = 0;

        if (tp > entry && sl < entry) {
            isLong = true;
            directionText = "做多 (Long)";
            profitDiff = tp - entry;
            lossDiff = entry - sl;
        } else if (tp < entry && sl > entry) {
            isLong = false;
            directionText = "做空 (Short)";
            profitDiff = entry - tp;
            lossDiff = sl - entry;
        } else {
             warningBox.style.display = 'block';
             warningBox.innerHTML = `⚠️ **錯誤：** 止盈與止損必須在入場價的兩側！`;
             resultArea.style.display = 'none';
             return;
        }

        // 3. 盈虧比 (b)
        if (lossDiff === 0) {
            warningBox.style.display = 'block';
            warningBox.innerHTML = `⚠️ **錯誤：** 止損距離不能為零！`;
            resultArea.style.display = 'none';
            return;
        }
        let b = profitDiff / lossDiff;

        // 4. 凱利公式 (f*) = (bp - q) / b
        let q = 1 - winrate;
        let f_star = ((b * winrate) - q) / b;

        // 5. 期望值檢查 (正期望值才能進場)
        if (f_star <= 0) {
            warningBox.style.display = 'block';
            warningBox.innerHTML = `⚠️ <strong>期望值為負或為零！</strong><br>此交易策略長期會虧損，建議**不要進場**。<br>盈虧比: 1:${b.toFixed(2)} | 勝率: ${(winrate*100).toFixed(0)}%`;
            resultArea.style.display = 'none';
            return;
        } else {
            warningBox.style.display = 'none';
        }

        // 6. 應用凱利倍數 (決定實際風險比例)
        let riskPercent = f_star * kellyMultiplier;

        // 7. 計算各項結果
        let riskAmount = capital * riskPercent; // 願意虧損的 USDT 金額
        
        let positionSize = riskAmount / lossDiff; // 倉位數量（顆數）
        
        let positionValue = positionSize * entry; // 倉位總價值 (USDT)
        
        let leverage = positionValue / capital; // 槓桿倍數

        let profitAmount = riskAmount * b; // 預期獲利金額

        // 8. 顯示結果
        resultArea.style.display = 'block';
        
        document.getElementById('direction').textContent = directionText;
        document.getElementById('direction').style.color = isLong ? 'var(--bull-color)' : 'var(--bear-color)';
        
        document.getElementById('rr_ratio').textContent = `1 : ${b.toFixed(2)}`;
        document.getElementById('kelly_risk_percent').textContent = `${(riskPercent * 100).toFixed(2)}% (基於本金)`;
        
        document.getElementById('risk_amt').textContent = `-$${riskAmount.toFixed(2)}`;
        document.getElementById('profit_amt').textContent = `+$${profitAmount.toFixed(2)}`;
        
        document.getElementById('position_value').textContent = `$${positionValue.toFixed(2)}`;
        
        // 槓桿與倉位數量顯示
        let displayLeverage = leverage.toFixed(2);
        document.getElementById('leverage').textContent = `${displayLeverage}x`;
        document.getElementById('position_size').textContent = positionSize.toFixed(6); 
    }
</script>

</body>
</html>
