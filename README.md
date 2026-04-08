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

        /* Range Matrix Controls */
        .range-controls { display: flex; flex-direction: column; gap: 10px; margin-bottom: 15px; background: #333; padding: 15px; border-radius: 5px; border: 1px solid #555;}
        select { padding: 10px; border: 1px solid #555; border-radius: 4px; background-color: #222; color: white; font-size: 16px; font-weight: bold;}
        .info-box { font-size: 14px; color: #3498db; line-height: 1.5; }

        /* Range Matrix */
        #matrix { display: grid; grid-template-columns: repeat(13, 1fr); gap: 2px; }
        .cell { aspect-ratio: 1; display: flex; align-items: center; justify-content: center; font-size: 12px; font-weight: bold; background-color: #444; cursor: pointer; user-select: none; border-radius: 3px; transition: 0.1s;}
        .cell:hover { opacity: 0.8; box-shadow: inset 0 0 0 2px #fff;}
        .pair { border: 1px solid #888; }
        
        /* State Colors */
        .state-0 { background-color: #444; } /* Empty/Default */
        .state-1 { background-color: #e74c3c; color: white;} /* Raise/3-Bet */
        .state-2 { background-color: #2ecc71; color: black; } /* Call */
        .state-3 { background-color: #7f8c8d; color: white; opacity: 0.4; } /* Fold */

        /* Form Controls */
        .form-group { display: flex; flex-direction: column; gap: 15px; max-width: 450px; margin: auto; }
        label { font-weight: bold; color: #ddd; }
        input { padding: 10px; border: 1px solid #555; border-radius: 4px; background-color: #333; color: white; font-size: 16px; }
        
        button.btn-main { padding: 12px; background-color: #f39c12; border: none; border-radius: 4px; color: white; font-size: 16px; font-weight: bold; cursor: pointer; margin-top: 10px; transition: 0.2s;}
        button.btn-main:hover { background-color: #e67e22; }

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
        .legend span { padding: 5px 10px; border-radius: 3px; font-weight: bold; }
        
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
        <div class="range-controls">
            <label for="positionSelect">Chọn vị trí (Bàn 8-Max / Max Profit):</label>
            <select id="positionSelect">
                <option value="custom">-- Bảng tự thiết kế của bạn (Custom) --</option>
                <option value="utg">1. UTG (Đầu bàn - Chơi cực chặt)</option>
                <option value="mp">2. MP (Giữa bàn - Bắt đầu mở rộng)</option>
                <option value="co">3. CO (Cutoff - Kề cuối, cướp chip mạnh)</option>
                <option value="btn">4. BTN (Button - Cướp mù rộng nhất)</option>
                <option value="bb_vs_btn">5. BB Defense (Bảo vệ mù vs BTN)</option>
            </select>
            <div id="range-info" class="info-box">Tự do click vào các ô bài bên dưới để tạo Range của riêng bạn. Hệ thống sẽ tự lưu lại.</div>
        </div>

        <div id="matrix"></div>
        
        <div class="legend">
            <span style="background-color: #e74c3c;">Raise / 3-Bet</span>
            <span style="background-color: #2ecc71; color: black;">Call</span>
            <span style="background-color: #7f8c8d; opacity: 0.8;">Fold</span>
        </div>
    </div>

    <div id="ev" class="content">
        <div class="form-group">
            <p style="font-size: 14px; color: #aaa; margin-bottom: 0;">Chọn nhanh Pot Size:</p>
            <div class="quick-btns">
                <button class="btn-quick pot-btn" onclick="setPot(100)">100</button>
                <button class="btn-quick pot-btn" onclick="setPot(200)">200</button>
                <button class="btn-quick pot-btn" onclick="setPot(250)">250</button>
                <button class="btn-quick pot-btn" onclick="setPot(300)">300</button>
                <button class="btn-quick pot-btn" onclick="setPot(500)">500</button>
                <button class="btn-quick pot-btn" onclick="setPot(1000)">1000</button>
            </div>
            
            <label for="pot">Tổng Pot (Pot hiện tại + Bet của đối thủ) $:</label>
            <input type="number" id="pot" placeholder="Vd: 145">

            <label for="call">Call Amount (Tiền bạn cần call) $:</label>
            <input type="number" id="call" placeholder="Vd: 50">
            
            <button class="btn-main" onclick="calculatePotOdds()">1. Tính Pot Odds (Equity Tối Thiểu)</button>
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
            <p style="font-size: 14px; color: #aaa; margin-bottom: 5px;">Chọn nhanh bài đợi (Draws):</p>
            <div class="quick-btns">
                <button class="btn-quick" onclick="setOuts(3)">1 Top Pair (3)</button>
                <button class="btn-quick" onclick="setOuts(4)">Sảnh lọt khe (4)</button>
                <button class="btn-quick" onclick="setOuts(4)">2 Đôi lên Cù Lũ (4)</button>
                <button class="btn-quick" onclick="setOuts(6)">Top Pair / 2 Overcards (6)</button>
                <button class="btn-quick" onclick="setOuts(7)">Set lên Cù Lũ/Tứ quý (7)</button>
                <button class="btn-quick" onclick="setOuts(8)">Sảnh 2 đầu (8)</button>
                <button class="btn-quick" onclick="setOuts(9)">Đợi Thùng (9)</button>
                <button class="btn-quick" onclick="setOuts(12)">Thùng + Sảnh khe (12)</button>
                <button class="btn-quick" onclick="setOuts(15)">Thùng + Sảnh 2 đầu (15)</button>
            </div>

            <label for="outs" style="margin-top: 10px;">Số lượng Outs của bạn:</label>
            <input type="number" id="outs" placeholder="Vd: 9 (Thùng)...">

            <label for="street">Giai đoạn (Street):</label>
            <select id="street" style="background-color: #333; color: white; padding: 10px; border: 1px solid #555;">
                <option value="flop_to_river">Từ Flop đến River (Xem 2 lá Turn & River)</option>
                <option value="flop_to_turn">Từ Flop đến Turn (Chỉ xem 1 lá)</option>
                <option value="turn_to_river">Từ Turn đến River (Chỉ xem 1 lá)</option>
            </select>

            <button class="btn-main" onclick="calculateEquity()">Tính Ước Lượng Equity</button>

            <div id="equity-result" class="result">Chọn Outs và bấm tính toán</div>
        </div>
    </div>
</div>

<script>
    // --- Database: Chuyên gia TAG Ranges (8-Max Focus) ---
    const tagRanges = {
        'utg': {
            raise: ['AA','KK','QQ','JJ','TT','99','88','77','AKs','AQs','AJs','ATs','A9s','A5s','KQs','KJs','KTs','QJs','QTs','JTs','T9s','AKo','AQo','AJo','KQo'],
            call: [],
            info: "🔥 <b>Chiến lược UTG (Early):</b> Rất dễ bị ép sân sau Flop. Chỉ Open Raise với top 12% bài mạnh nhất. Bỏ hoàn toàn các lá Ax yếu hoặc bài Suit Connectors nhỏ.<br>👉 <b>Hành động:</b> Raise 2.5x - 3x BB. Bị 3-Bet thì Fold nếu không cầm AA, KK, QQ, AK."
        },
        'mp': {
            raise: ['AA','KK','QQ','JJ','TT','99','88','77','66','55','AKs','AQs','AJs','ATs','A9s','A8s','A5s','A4s','KQs','KJs','KTs','K9s','QJs','QTs','Q9s','JTs','J9s','T9s','98s','87s','AKo','AQo','AJo','ATo','KQo','KJo'],
            call: [],
            info: "🔥 <b>Chiến lược MP (Middle):</b> Vị trí trung bình. Bắt đầu mở rộng đánh các đôi nhỏ (55, 66) và các bài Suit Connectors (87s, 98s) để bẫy đối thủ.<br>👉 <b>Hành động:</b> Raise 2.5x BB. Hạn chế Call hờ, hãy là người nổ súng đầu tiên."
        },
        'co': {
            raise: ['AA','KK','QQ','JJ','TT','99','88','77','66','55','44','33','22','AKs','AQs','AJs','ATs','A9s','A8s','A7s','A6s','A5s','A4s','A3s','A2s','KQs','KJs','KTs','K9s','K8s','QJs','QTs','Q9s','JTs','J9s','T9s','98s','87s','76s','65s','AKo','AQo','AJo','ATo','A9o','KQo','KJo','KTo','QJo','QTo','JTo'],
            call: [],
            info: "🔥 <b>Chiến lược CO (Cutoff):</b> Đây là lúc in tiền! Tấn công mạnh mẽ để cướp Blinds. Đánh mọi đôi, mọi lá Át đồng chất và Broadways.<br>👉 <b>Hành động:</b> Raise 2.5x BB. Nếu Button là người chơi hiền (Tight), bạn có thể Raise rác hơn nữa."
        },
        'btn': {
            raise: ['AA','KK','QQ','JJ','TT','99','88','77','66','55','44','33','22','AKs','AQs','AJs','ATs','A9s','A8s','A7s','A6s','A5s','A4s','A3s','A2s','KQs','KJs','KTs','K9s','K8s','K7s','K6s','K5s','K4s','QJs','QTs','Q9s','Q8s','Q7s','JTs','J9s','J8s','T9s','T8s','98s','97s','87s','86s','76s','75s','65s','54s','AKo','AQo','AJo','ATo','A9o','A8o','A7o','A5o','KQo','KJo','KTo','K9o','QJo','QTo','Q9o','JTo','J9o','T9o','98o'],
            call: [],
            info: "🔥 <b>Chiến lược BTN (Button):</b> Vua của bàn poker. Raise cực rộng (gần 50% hand) để gây áp lực tuyệt đối lên Blind. Luôn có lợi thế đánh sau ở Flop.<br>👉 <b>Hành động:</b> Raise 2.5x BB. Đừng cho Blinds thở."
        },
        'bb_vs_btn': {
            raise: ['AA','KK','QQ','JJ','TT','AKs','AQs','AJs','AKo','AQo','A5s','A4s','A3s','A2s','KQs'], 
            call: ['99','88','77','66','55','44','33','22','ATs','A9s','A8s','A7s','A6s','KJs','KTs','K9s','K8s','K7s','K6s','K5s','K4s','K3s','K2s','QJs','QTs','Q9s','Q8s','Q7s','Q6s','Q5s','Q4s','Q3s','Q2s','JTs','J9s','J8s','J7s','T9s','T8s','T7s','98s','97s','87s','86s','76s','65s','54s','AJo','ATo','A9o','A8o','KQo','KJo','KTo','QJo','QTo','JTo'],
            info: "🔥 <b>Chiến lược BB Defense:</b> Khi Button cố cướp Blind, hãy mạnh dạn Call nhiều hơn vì được giảm giá (Pot Odds tốt). <br>👉 <b>Hành động:</b> Cầm hàng khủng (AA, KK, AK) hoặc hàng Bluff tốt (A2s-A5s), hãy <b>3-Bet x4 lần</b>. Các bài trung bình thì Call xem Flop."
        }
    };

    // --- Tab Logic ---
    function switchTab(tabId) {
        document.querySelectorAll('.content').forEach(c => c.classList.remove('active'));
        document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
        document.getElementById(tabId).classList.add('active');
        event.currentTarget.classList.add('active');
    }

    // --- Range Matrix Setup ---
    const ranks = ['A', 'K', 'Q', 'J', 'T', '9', '8', '7', '6', '5', '4', '3', '2'];
    const matrixEl = document.getElementById('matrix');
    let customRange = JSON.parse(localStorage.getItem('customPokerRange')) || Array(169).fill(3); 

    // Build DOM Matrix
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
            cell.className = `cell state-3 ${extraClass}`; 
            cell.dataset.state = 3;
            cell.dataset.hand = hand;
            cell.dataset.index = cellIndex;
            cell.textContent = hand;
            
            // Handle User Clicks
            cell.addEventListener('click', function() {
                document.getElementById('positionSelect').value = 'custom';
                document.getElementById('range-info').innerHTML = "Bảng tự thiết kế của bạn (Custom). Tự động lưu vào trình duyệt.";
                
                let state = parseInt(this.dataset.state);
                let newState = state === 3 ? 1 : (state === 1 ? 2 : 3);
                
                this.dataset.state = newState;
                this.className = `cell state-${newState} ${extraClass}`;
                
                const cells = document.querySelectorAll('.cell');
                customRange = Array.from(cells).map(c => parseInt(c.dataset.state));
                localStorage.setItem('customPokerRange', JSON.stringify(customRange));
            });

            matrixEl.appendChild(cell);
            cellIndex++;
        }
    }

    // Load Ranges Dropdown Logic
    document.getElementById('positionSelect').addEventListener('change', function() {
        const val = this.value;
        const cells = document.querySelectorAll('.cell');
        
        if (val === 'custom') {
            document.getElementById('range-info').innerHTML = "Bảng tự thiết kế của bạn (Custom). Tự động lưu vào trình duyệt.";
            cells.forEach((cell, idx) => {
                let state = customRange[idx];
                cell.dataset.state = state;
                cell.className = `cell state-${state} ${cell.classList.contains('pair')?'pair':''}`;
            });
            return;
        }

        const data = tagRanges[val];
        document.getElementById('range-info').innerHTML = data.info;

        cells.forEach(cell => {
            const hand = cell.dataset.hand;
            let newState = 3; 
            if (data.raise.includes(hand)) newState = 1; 
            else if (data.call.includes(hand)) newState = 2; 
            
            cell.dataset.state = newState;
            cell.className = `cell state-${newState} ${cell.classList.contains('pair')?'pair':''}`;
        });
    });

    document.getElementById('positionSelect').dispatchEvent(new Event('change'));


    // --- Tab 2 & 3 Logic ---
    function setPot(amount) { document.getElementById('pot').value = amount; }

    function calculatePotOdds() {
        const pot = parseFloat(document.getElementById('pot').value);
        const call = parseFloat(document.getElementById('call').value);
        const resultEl = document.getElementById('pot-odds-result');
        if (isNaN(pot) || isNaN(call) || pot <= 0 || call < 0) {
            resultEl.style.display = 'block'; resultEl.innerHTML = "Vui lòng nhập Pot và Call hợp lệ!"; return;
        }
        const requiredEquity = (call / (pot + call)) * 100;
        resultEl.style.display = 'block';
        resultEl.innerHTML = `Equity tối thiểu cần để Call: <span class="highlight-yellow" style="font-size: 22px;">${requiredEquity.toFixed(1)}%</span><br><span style="font-size: 13px; color:#aaa;">(Nếu Equity > số này = Cứ Call)</span>`;
    }

    function calculateEV() {
        const pot = parseFloat(document.getElementById('pot').value);
        const call = parseFloat(document.getElementById('call').value);
        const equityPercent = parseFloat(document.getElementById('equity-input').value);
        const resultEl = document.getElementById('ev-result');
        if (isNaN(pot) || isNaN(call) || isNaN(equityPercent)) {
            resultEl.innerHTML = "Vui lòng nhập đầy đủ các số hợp lệ!"; return;
        }
        const equity = equityPercent / 100;
        const ev = (equity * pot) - ((1 - equity) * call);
        if (ev > 0) resultEl.innerHTML = `EV = +$${ev.toFixed(2)}<br><span style="font-size: 24px;" class="highlight-green">CALL (Có lãi dài hạn)</span>`;
        else if (ev < 0) resultEl.innerHTML = `EV = -$${Math.abs(ev).toFixed(2)}<br><span style="font-size: 24px;" class="highlight-red">FOLD (Lỗ dài hạn)</span>`;
        else resultEl.innerHTML = `EV = $0<br><span class="highlight-yellow">Hòa vốn</span>`;
    }

    function setOuts(num) { document.getElementById('outs').value = num; calculateEquity(); }

    function calculateEquity() {
        const outs = parseInt(document.getElementById('outs').value);
        const street = document.getElementById('street').value;
        const resultEl = document.getElementById('equity-result');
        if (isNaN(outs) || outs < 0 || outs > 22) { resultEl.innerHTML = "Vui lòng nhập số Outs hợp lệ!"; return; }
        let equity = 0;
        if (street === "flop_to_turn" || street === "turn_to_river") { equity = outs * 2; }
        else if (street === "flop_to_river") { equity = outs * 4; if (outs > 8) equity = equity - (outs - 8); }
        if (equity > 100) equity = 100;
        resultEl.innerHTML = `Ước tính Equity: <span class="highlight-green" style="font-size: 26px;">~${equity}%</span>`;
        document.getElementById('equity-input').value = equity;
    }
</script>

</body>
</html>
