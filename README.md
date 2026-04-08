<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Poker Range & EV Calculator</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #1e1e1e; color: #fff; margin: 0; padding: 20px; }
        .container { max-width: 800px; margin: auto; background: #2d2d2d; padding: 20px; border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.5); }
        h1 { text-align: center; color: #f39c12; }
        
        /* Tabs */
        .tabs { display: flex; margin-bottom: 20px; border-bottom: 2px solid #444; }
        .tab { padding: 10px 20px; cursor: pointer; font-weight: bold; color: #ccc; }
        .tab:hover { color: #fff; }
        .tab.active { color: #f39c12; border-bottom: 2px solid #f39c12; margin-bottom: -2px; }
        
        .content { display: none; }
        .content.active { display: block; }

        /* Range Matrix */
        #matrix { display: grid; grid-template-columns: repeat(13, 1fr); gap: 2px; margin-top: 20px; }
        .cell { aspect-ratio: 1; display: flex; align-items: center; justify-content: center; font-size: 12px; font-weight: bold; background-color: #444; cursor: pointer; user-select: none; border-radius: 3px; }
        .cell:hover { opacity: 0.8; }
        .pair { border: 1px solid #777; }
        
        /* State Colors */
        .state-0 { background-color: #444; } /* Default */
        .state-1 { background-color: #e74c3c; color: white;} /* Raise */
        .state-2 { background-color: #2ecc71; color: black; } /* Call */
        .state-3 { background-color: #7f8c8d; color: white; } /* Fold */

        /* EV Calculator */
        .ev-form { display: flex; flex-direction: column; gap: 15px; max-width: 400px; margin: auto; }
        label { font-weight: bold; }
        input { padding: 10px; border: none; border-radius: 4px; background-color: #444; color: white; font-size: 16px; }
        button { padding: 10px; background-color: #f39c12; border: none; border-radius: 4px; color: white; font-size: 16px; font-weight: bold; cursor: pointer; }
        button:hover { background-color: #e67e22; }
        .result { margin-top: 20px; padding: 15px; border-radius: 4px; text-align: center; font-size: 18px; font-weight: bold; background-color: #444; }
        .ev-positive { color: #2ecc71; }
        .ev-negative { color: #e74c3c; }
        
        .legend { display: flex; gap: 10px; justify-content: center; margin-top: 15px; font-size: 14px; }
        .legend span { padding: 5px 10px; border-radius: 3px; }
    </style>
</head>
<body>

<div class="container">
    <h1>Poker Tools</h1>
    
    <div class="tabs">
        <div class="tab active" onclick="switchTab('range')">Preflop Range</div>
        <div class="tab" onclick="switchTab('ev')">Tính EV</div>
    </div>

    <div id="range" class="content active">
        <p style="text-align: center; font-size: 14px; color: #aaa;">Click vào ô để đổi màu: Mặc định -> Raise -> Call -> Fold</p>
        <div id="matrix"></div>
        <div class="legend">
            <span style="background-color: #e74c3c;">Raise</span>
            <span style="background-color: #2ecc71; color: black;">Call</span>
            <span style="background-color: #7f8c8d;">Fold</span>
        </div>
    </div>

    <div id="ev" class="content">
        <div class="ev-form">
            <label for="pot">Pot Size (Tổng tiền Pot hiện tại + Bet của đối thủ) $:</label>
            <input type="number" id="pot" placeholder="Vd: 100">

            <label for="call">Call Amount (Tiền bạn cần bỏ ra) $:</label>
            <input type="number" id="call" placeholder="Vd: 50">

            <label for="equity">Equity (Tỉ lệ thắng của bạn) %:</label>
            <input type="number" id="equity" placeholder="Vd: 45">

            <button onclick="calculateEV()">Tính Toán EV</button>

            <div id="ev-result" class="result">Nhập thông số để xem kết quả</div>
        </div>
    </div>
</div>

<script>
    // Tab switching logic
    function switchTab(tabId) {
        document.querySelectorAll('.content').forEach(c => c.classList.remove('active'));
        document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
        document.getElementById(tabId).classList.add('active');
        event.currentTarget.classList.add('active');
    }

    // Generate 13x13 Matrix
    const ranks = ['A', 'K', 'Q', 'J', 'T', '9', '8', '7', '6', '5', '4', '3', '2'];
    const matrixEl = document.getElementById('matrix');

    for (let i = 0; i < 13; i++) {
        for (let j = 0; j < 13; j++) {
            let hand = '';
            let extraClass = '';
            if (i === j) {
                hand = ranks[i] + ranks[j];
                extraClass = 'pair';
            } else if (i < j) {
                hand = ranks[i] + ranks[j] + 's';
            } else {
                hand = ranks[j] + ranks[i] + 'o';
            }

            const cell = document.createElement('div');
            cell.className = `cell state-0 ${extraClass}`;
            cell.dataset.state = 0;
            cell.textContent = hand;
            
            // Click to cycle states (0: default, 1: raise, 2: call, 3: fold)
            cell.addEventListener('click', function() {
                let currentState = parseInt(this.dataset.state);
                let newState = (currentState + 1) % 4;
                this.dataset.state = newState;
                this.className = `cell state-${newState} ${extraClass}`;
            });

            matrixEl.appendChild(cell);
        }
    }

    // EV Calculation
    function calculateEV() {
        const pot = parseFloat(document.getElementById('pot').value);
        const call = parseFloat(document.getElementById('call').value);
        const equityPercent = parseFloat(document.getElementById('equity').value);
        const resultEl = document.getElementById('ev-result');

        if (isNaN(pot) || isNaN(call) || isNaN(equityPercent)) {
            resultEl.innerHTML = "Vui lòng nhập đầy đủ các số hợp lệ!";
            resultEl.className = "result";
            return;
        }

        const equity = equityPercent / 100;
        const loseRate = 1 - equity;
        const ev = (equity * pot) - (loseRate * call);

        if (ev > 0) {
            resultEl.innerHTML = `EV = +$${ev.toFixed(2)}<br><span style="font-size: 24px;">CALL (Có lãi dài hạn)</span>`;
            resultEl.className = "result ev-positive";
        } else if (ev < 0) {
            resultEl.innerHTML = `EV = -$${Math.abs(ev).toFixed(2)}<br><span style="font-size: 24px;">FOLD (Lỗ dài hạn)</span>`;
            resultEl.className = "result ev-negative";
        } else {
            resultEl.innerHTML = `EV = $0<br>Hòa vốn (Tùy chọn)`;
            resultEl.className = "result";
        }
    }
</script>

</body>
</html>
