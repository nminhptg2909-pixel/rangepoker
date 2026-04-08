<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Poker Tools: Range, EV, Pot Odds & Equity</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #1e1e1e; color: #fff; margin: 0; padding: 20px; }
        .container { max-width: 800px; margin: auto; background: #2d2d2d; padding: 20px; border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.5); }
        h1 { text-align: center; color: #f39c12; font-size: 24px; }
        
        /* Tabs */
        .tabs { display: flex; margin-bottom: 20px; border-bottom: 2px solid #444; flex-wrap: wrap; }
        .tab { padding: 10px 15px; cursor: pointer; font-weight: bold; color: #ccc; font-size: 15px; }
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

        /* Form Controls */
        .form-group { display: flex; flex-direction: column; gap: 15px; max-width: 450px; margin: auto; }
        label { font-weight: bold; color: #ddd; }
        input, select { padding: 10px; border: 1px solid #555; border-radius: 4px; background-color: #333; color: white; font-size: 16px; }
        input:focus, select:focus { outline: none; border-color: #f39c12; }
        
        button.btn-main { padding: 12px; background-color: #f39c12; border: none; border-radius: 4px; color: white; font-size: 16px; font-weight: bold; cursor: pointer; margin-top: 10px; transition: 0.2s;}
        button.btn-main:hover { background-color: #e67e22; }
        
        .btn-reset { background-color: #c0392b; margin-top: 20px; width: 100%; padding: 10px; border: none; border-radius: 4px; color: white; font-weight: bold; cursor: pointer;}
        .btn-reset:hover { background-color: #e74c3c; }

        /* Quick Buttons */
        .quick-btns { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 5px; }
        .btn-quick { background-color: #444; color: #fff; border: 1px solid #666; padding: 6px 10px; border-radius: 15px; font-size: 13px; cursor: pointer; transition: 0.2s; }
        .btn-quick:hover { background-color: #3498db; border-color: #3498db; }
        .btn-quick.pot-btn { background-color: #2c3e50; border-color: #34495e; font-weight: bold;}
        .btn-quick.pot-btn:hover { background-color: #1abc9c; border-color: #1abc9c; }
        
        /* Result Box */
        .result { margin-top: 20px; padding: 15px; border-radius: 4px; text-align: center; font-size: 16px; background-color: #333; border: 1px solid #555;}
        .highlight-green { color: #2ecc71; font-weight: bold;}
        .highlight-red { color: #e74c3c; font-weight: bold;}
        .highlight-yellow { color: #f1c40f; font-weight: bold;}
        
        .legend { display: flex; gap: 10px; justify-content: center; margin-top: 15px; font-size: 14px; }
        .legend span { padding: 5px 10px; border-radius: 3px; }
        
        .divider { height: 1px; background-color: #555; margin: 20px 0; }
    </style>
</head>
<body>

<div class="container">
    <h1>Poker Tools Pro</h1>
    
    <div class="tabs">
        <div class="tab active" onclick="switchTab('range')">1. Preflop Range</div>
        <div class="tab" onclick="switchTab('ev')">2. Pot Odds & EV</div>
        <div class="tab" onclick="switchTab('equity')">3. Equity & Outs</div>
    </div>

    <div id="range" class="content active">
        <p style="text-align: center; font-size: 14px; color: #aaa;">Click vào ô để đổi màu. Tự động lưu bằng LocalStorage.</p>
        <div id="matrix"></div>
        <div class="legend">
            <span style="background-color: #e74c3c;">Raise</span>
            <span style="background-color: #2ecc71; color: black;">Call</span>
            <span style="background-color: #7f8c8d;">Fold</span>
        </div>
        <button class="btn-reset" onclick="resetRange()">Reset Bảng Range</button>
    </div>

    <div id="ev" class="content">
        <div class="form-group">
            <p style="font-size: 14px; color: #aaa; margin-bottom: 0;">Chọn nhanh Pot Size:</p>
            <div class="quick-btns">
                <button class="btn-quick pot-btn" onclick="setPot(100)">100</button>
                <button class="btn-quick pot-btn" onclick="setPot(200)">200</button>
                <button class="btn-quick pot-btn" onclick="setPot(250)">250</button>
                <button class="btn-quick pot-btn" onclick="setPot(300)">300</button>
                <button class="btn-quick pot-btn" onclick="setPot(350)">350</button>
                <button class="btn-quick pot-btn" onclick="setPot(400)">400</button>
                <button class="btn-quick pot-btn" onclick="setPot(500)">500</button>
                <button class="btn-quick pot-btn" onclick="setPot(750)">750</button>
                <button class="btn-quick pot-btn" onclick="setPot(1000)">1000</button>
            </div>
            
            <label for="pot">Hoặc nhập tay Tổng Pot (Pot hiện tại + Bet của đối thủ) $:</label>
            <input type="number" id="pot" placeholder="Vd: 145">

            <label for="call">Call Amount (Tiền bạn cần call) $:</label>
            <input type="number" id="call" placeholder="Vd: 50">
            
            <button class="btn-main" onclick="calculatePotOdds()">1. Tính Pot Odds (Equity Tối Thiểu Cần Thiết)</button>
            <div id="pot-odds-result" class="result" style="display: none;"></div>

            <div class="divider"></div>

            <label for="equity-input">Equity của bạn hiện có (Tra ở Tab 3) %:</label>
            <input type="number" id="equity-input" placeholder="Vd: 45">

            <button class="btn-main" onclick="calculateEV()">2. Tính Toán EV Cụ Thể</button>
            <div id="ev-result" class="result">Nhập thông số để xem kết quả</div>
        </div>
    </div>

    <div id="equity" class="content">
        <div class="form-group">
            <p style="font-size: 14px; color: #aaa; margin-bottom: 5px;">Chọn nhanh tình huống bài đợi (Draws):</p>
            <div class="quick-btns">
                <button class="btn-quick" onclick="setOuts(3)">Đợi 1 Top Pair (3)</button>
                <button class="btn-quick" onclick="setOuts(4)">Sảnh lọt khe (4)</button>
                <button class="btn-quick" onclick="setOuts(4)">2 Đôi lên Cù Lũ (4)</button>
                <button class="btn-quick" onclick="setOuts(6)">Đợi Top Pair / 2 Overcards (6)</button>
                <button class="btn-quick" onclick="setOuts(7)">Set lên Cù Lũ/Tứ quý (7)</button>
                <button class="btn-quick" onclick="setOuts(8)">Sảnh 2 đầu (8)</button>
                <button class="btn-quick" onclick="setOuts(9)">Đợi Thùng (9)</button>
                <button class="btn-quick" onclick="setOuts(12)">Thùng + Sảnh khe (12)</button>
                <button class="btn-quick" onclick="setOuts(15)">Thùng + Sảnh 2 đầu (15)</button>
            </div>

            <label for="outs" style="margin-top: 10px;">Số lượng Outs của bạn:</label>
            <input type="number" id="outs" placeholder="Vd: 9 (Thùng)...">

            <label for="street">Giai đoạn (Street):</label>
            <select id="street">
                <option value="flop_to_river">Từ Flop đến River (Được xem cả 2 lá Turn & River)</option>
                <option value="flop_to_turn">Từ Flop đến Turn (Chỉ xem 1 lá Turn)</option>
                <option value="turn_to_river">Từ Turn đến River (Chỉ xem 1 lá River)</option>
            </select>

            <button class="btn-main" onclick="calculateEquity()">Tính Ước Lượng Equity</button>

            <div id="equity-result" class="result">Chọn Outs và bấm tính toán</div>
        </div>
    </div>
</div>

<script>
    // --- Tab Logic ---
    function switchTab(tabId) {
        document.querySelectorAll('.content').forEach(c => c.classList.remove('active'));
        document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
        document.getElementById(tabId).classList.add('active');
        event.currentTarget.classList.add('active');
    }

    // --- Tab 1: Range Matrix Logic ---
    const ranks = ['A', 'K', 'Q', 'J', 'T', '9', '8', '7', '6', '5', '4', '3', '2'];
    const matrixEl = document.getElementById('matrix');
    let savedRange = JSON.parse(localStorage.getItem('pokerRange')) || [];

    function saveRange() {
        const cells = document.querySelectorAll('.cell');
        const stateArray = Array.from(cells).map(c => c.dataset.state);
        localStorage.setItem('pokerRange', JSON.stringify(stateArray));
    }

    function resetRange() {
        if(confirm("Bạn có chắc chắn muốn xóa toàn bộ bảng Range?")) {
            localStorage.removeItem('pokerRange');
            location.reload();
        }
    }

    let cellIndex = 0;
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
            let currentState = savedRange[cellIndex] ? savedRange[cellIndex] : 0;
            
            cell.className = `cell state-${currentState} ${extraClass}`;
            cell.dataset.state = currentState;
            cell.textContent = hand;
            
            cell.addEventListener('click', function() {
                let state = parseInt(this.dataset.state);
                let newState = (state + 1) % 4;
                this.dataset.state = newState;
                this.className = `cell state-${newState} ${extraClass}`;
                saveRange();
            });

            matrixEl.appendChild(cell);
            cellIndex++;
        }
    }

    // --- Tab 2: Pot Odds & EV Logic ---
    function setPot(amount) {
        document.getElementById('pot').value = amount;
    }

    function calculatePotOdds() {
        const pot = parseFloat(document.getElementById('pot').value);
        const call = parseFloat(document.getElementById('call').value);
        const resultEl = document.getElementById('pot-odds-result');

        if (isNaN(pot) || isNaN(call) || pot <= 0 || call < 0) {
            resultEl.style.display = 'block';
            resultEl.innerHTML = "Vui lòng nhập Pot và Call hợp lệ!";
            return;
        }

        const requiredEquity = (call / (pot + call)) * 100;
        
        resultEl.style.display = 'block';
        resultEl.innerHTML = `Equity tối thiểu cần để Call: <span class="highlight-yellow" style="font-size: 22px;">${requiredEquity.toFixed(1)}%</span><br><span style="font-size: 13px; color:#aaa;">(Nếu Equity của bạn > con số này, hãy Call)</span>`;
    }

    function calculateEV() {
        const pot = parseFloat(document.getElementById('pot').value);
        const call = parseFloat(document.getElementById('call').value);
        const equityPercent = parseFloat(document.getElementById('equity-input').value);
        const resultEl = document.getElementById('ev-result');

        if (isNaN(pot) || isNaN(call) || isNaN(equityPercent)) {
            resultEl.innerHTML = "Vui lòng nhập đầy đủ các số hợp lệ!";
            return;
        }

        const equity = equityPercent / 100;
        const loseRate = 1 - equity;
        const ev = (equity * pot) - (loseRate * call);

        if (ev > 0) {
            resultEl.innerHTML = `EV = +$${ev.toFixed(2)}<br><span style="font-size: 24px;" class="highlight-green">CALL (Có lãi dài hạn)</span>`;
        } else if (ev < 0) {
            resultEl.innerHTML = `EV = -$${Math.abs(ev).toFixed(2)}<br><span style="font-size: 24px;" class="highlight-red">FOLD (Lỗ dài hạn)</span>`;
        } else {
            resultEl.innerHTML = `EV = $0<br><span class="highlight-yellow">Hòa vốn (Tùy chọn)</span>`;
        }
    }

    // --- Tab 3: Equity (Outs) Logic ---
    function setOuts(num) {
        document.getElementById('outs').value = num;
        calculateEquity();
    }

    function calculateEquity() {
        const outs = parseInt(document.getElementById('outs').value);
        const street = document.getElementById('street').value;
        const resultEl = document.getElementById('equity-result');

        if (isNaN(outs) || outs < 0 || outs > 22) {
            resultEl.innerHTML = "Vui lòng nhập số Outs hợp lệ (Thường từ 1 đến 21)!";
            return;
        }

        let equity = 0;
        let isAdjusted = false;

        if (street === "flop_to_turn" || street === "turn_to_river") {
            equity = outs * 2;
        } else if (street === "flop_to_river") {
            equity = outs * 4;
            if (outs > 8) {
                equity = equity - (outs - 8);
                isAdjusted = true;
            }
        }

        if (equity > 100) equity = 100;

        let extraText = isAdjusted ? "<br><span style='font-size: 12px; color: #aaa;'>(Đã trừ hao để tính chuẩn xác hơn)</span>" : "";
        resultEl.innerHTML = `Ước tính Equity: <span class="highlight-green" style="font-size: 26px;">~${equity}%</span>${extraText}`;
        
        document.getElementById('equity-input').value = equity;
    }
</script>

</body>
</html>
