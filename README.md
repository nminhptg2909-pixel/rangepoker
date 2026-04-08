<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Poker Tools: Range, EV, Pot Odds & Equity (9-Max)</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #1e1e1e; color: #fff; margin: 0; padding: 20px; }
        .container { max-width: 800px; margin: auto; background: #2d2d2d; padding: 20px; border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.5); }
        h1 { text-align: center; color: #f39c12; font-size: 24px; margin-bottom: 10px; }
        
        /* Tabs */
        .tabs { display: flex; margin-bottom: 20px; border-bottom: 2px solid #444; flex-wrap: wrap; }
        .tab { padding: 10px 15px; cursor: pointer; font-weight: bold; color: #ccc; font-size: 15px; transition: 0.2s;}
        .tab:hover { color: #fff; }
        .tab.active { color: #f39c12; border-bottom: 2px solid #f39c12; margin-bottom: -2px; }
        
        .content { display: none; }
        .content.active { display: block; }

        /* Range Matrix Controls */
        .range-controls { display: flex; flex-direction: column; gap: 10px; margin-bottom: 15px; background: #333; padding: 15px; border-radius: 5px; border: 1px solid #555;}
        select { padding: 10px; border: 1px solid #555; border-radius: 4px; background-color: #222; color: white; font-size: 16px; font-weight: bold; cursor: pointer;}
        .info-box { font-size: 14px; color: #3498db; line-height: 1.5; }

        /* Range Matrix */
        #matrix { display: grid; grid-template-columns: repeat(13, 1fr); gap: 2px; }
        .cell { aspect-ratio: 1; display: flex; align-items: center; justify-content: center; font-size: 12px; font-weight: bold; background-color: #444; user-select: none; border-radius: 3px; transition: 0.2s;}
        .pair { border: 1px solid #888; }
        
        /* State Colors */
        .state-0 { background-color: #444; } /* Empty/Default */
        .state-1 { background-color: #e74c3c; color: white;} /* Raise/3-Bet (Red) */
        .state-2 { background-color: #2ecc71; color: black; } /* Call vs Raise (Green) */
        .state-3 { background-color: #7f8c8d; color: white; opacity: 0.3; } /* Fold (Grey) */
        .state-4 { background-color: #3498db; color: white; } /* Limp / Over-limp (Blue) */

        /* Legend */
        .legend { display: flex; flex-wrap: wrap; gap: 10px; justify-content: center; margin-top: 15px; font-size: 14px; }
        .legend span { padding: 5px 10px; border-radius: 3px; font-weight: bold; }

        /* Cheat Sheet Section */
        .cheat-sheet { margin-top: 25px; background: #2c3e50; padding: 15px 20px; border-radius: 6px; border-left: 5px solid #f39c12; font-size: 14px; line-height: 1.6;}
        .cheat-sheet h3 { margin-top: 0; color: #f1c40f; font-size: 18px; border-bottom: 1px solid #555; padding-bottom: 5px;}
        .cheat-sheet h4 { color: #ecf0f1; margin: 10px 0 5px 0; }
        .cheat-sheet ul { margin: 0; padding-left: 20px; }
        .cheat-sheet li { margin-bottom: 5px; color: #bdc3c7;}
        .text-highlight { color: #e74c3c; font-weight: bold; }
        .text-blue { color: #3498db; font-weight: bold; }

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
        
        .divider { height: 1px; background-color: #555; margin: 20px 0; }
    </style>
</head>
<body>

<div class="container">
    <h1>Poker Tools Pro (9-Max)</h1>
    
    <div class="tabs">
        <div class="tab active" onclick="switchTab('range')">1. Preflop Range</div>
        <div class="tab" onclick="switchTab('ev')">2. Pot Odds & EV</div>
        <div class="tab" onclick="switchTab('equity')">3. Equity & Outs</div>
    </div>

    <div id="range" class="content active">
        <div class="range-controls">
            <label for="positionSelect">Chọn vị trí (Bàn 9-Max Full Ring):</label>
            <select id="positionSelect">
                <option value="utg">1. UTG (Đầu bàn 1 - Siêu Tight Top 8%)</option>
                <option value="utg1">2. UTG+1 (Đầu bàn 2 - Top 10%)</option>
                <option value="utg2">3. UTG+2 (Đầu bàn 3 - Top 12%)</option>
                <option value="mp">4. MP / LoJack (Giữa bàn - Bắt đầu Over-Limp)</option>
                <option value="hj">5. HJ / Hijack (Over-limp lấy Set/Sảnh)</option>
                <option value="co">6. CO / Cutoff (Kề cuối - Cướp blind)</option>
                <option value="btn">7. BTN / Button (Vua bàn - Đánh cực rộng)</option>
                <option value="sb">8. SB (Small Blind - Complete vs Limpers)</option>
                <option value="bb">9. BB (Big Blind - Defense vs Raise)</option>
            </select>
            <div id="range-info" class="info-box">Bảng tra cứu Range bài. Đã khóa tính năng click để tránh chạm nhầm.</div>
        </div>

        <div id="matrix"></div>
        
        <div class="legend">
            <span style="background-color: #e74c3c;">Raise / 3-Bet</span>
            <span style="background-color: #2ecc71; color: black;">Call (vs Raise)</span>
            <span style="background-color: #3498db; color: white;">Limp / Over-limp</span>
            <span style="background-color: #7f8c8d; opacity: 0.8;">Fold</span>
        </div>

        <div class="cheat-sheet">
            <h3>⚡ Bí Kíp Thực Chiến 9-Max (Max Profit)</h3>
            
            <h4>1. Trừng phạt kẻ Limp (Iso-Raise)</h4>
            <ul>
                <li>Nếu có người Limp, đừng Limp theo nếu bạn có bài mạnh (AQ+, TT+). Hãy Raise để cách ly.</li>
                <li>Công thức Size: <span class="text-highlight">3 BB + 1 BB</span> (cho mỗi người Limp) <span class="text-highlight">+ 1 BB</span> (nếu bạn OOP).</li>
            </ul>

            <h4>2. Đào mỏ Sám Cô (Set Mining)</h4>
            <ul>
                <li>Dùng đôi nhỏ (22-99) để bẫy khi có nhiều người vào Pot (Family pot).</li>
                <li><span class="text-blue">Quy tắc 15x:</span> Chỉ Call khi Stack của bạn và đối thủ lớn hơn gấp 15 lần số tiền bạn phải Call (để đảm bảo Implied Odds). Trượt Flop = Bỏ ngay.</li>
            </ul>

            <h4>3. Sinh tồn khi OOP (Bất Lợi Vị Trí - SB, BB, UTG)</h4>
            <ul>
                <li><span class="text-highlight">Nguyên tắc 3-Bet or Fold ở SB:</span> Ở SB, đừng Call ngang cú Raise. Bạn sẽ bị BB kẹp thịt và phải đánh OOP cả ván. Hãy 3-Bet mạnh tay hoặc Fold.</li>
                <li><span class="text-blue">Đừng yêu Top Pair yếu:</span> Nếu cầm KQ, QJ trúng 1 đôi ở Flop nhưng bạn phải check trước, hãy kiểm soát Pot (Check/Call) thay vì Bet lớn để tránh bị Value đè.</li>
                <li>Thắt chặt Range (Tighten Up): Nếu ngồi đầu bàn, phải bỏ hoàn toàn các bài dễ bị chi phối như ATo, KJo vì bạn không có thông tin từ 8 người ngồi sau.</li>
            </ul>
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
    // --- Database: Chuyên gia TAG Ranges (9-MAX FULL RING) ---
    const tagRanges = {
        'utg': {
            raise: ['AA','KK','QQ','JJ','TT','99','88','AKs','AQs','AJs','ATs','KQs','AKo','AQo'],
            call: [], limp: [],
            info: "🔥 <b>Chiến lược UTG (Siêu Tight):</b> Có 8 người ngồi sau. Bạn ĐẶC BIỆT dễ chết nếu cầm lá Át yếu. Chỉ Open Raise Top 8% siêu xịn."
        },
        'utg1': {
            raise: ['AA','KK','QQ','JJ','TT','99','88','77','AKs','AQs','AJs','ATs','A9s','A5s','KQs','KJs','AKo','AQo','AJo','KQo'],
            call: [], limp: [],
            info: "🔥 <b>Chiến lược UTG+1:</b> Vẫn là Early Position. Bắt đầu đánh thêm đôi 77 và một số Broadways mạnh. Không Limp."
        },
        'utg2': {
            raise: ['AA','KK','QQ','JJ','TT','99','88','77','66','55','AKs','AQs','AJs','ATs','A9s','A8s','A5s','A4s','KQs','KJs','KTs','QJs','QTs','JTs','T9s','AKo','AQo','AJo','ATo','KQo','KJo'],
            call: [], limp: [],
            info: "🔥 <b>Chiến lược UTG+2:</b> Chuẩn bị vào giữa bàn. Thêm các bài Suited Connectors to (T9s, JTs) và các đôi nhỏ để Set Mining."
        },
        'mp': {
            raise: ['AA','KK','QQ','JJ','TT','99','88','77','66','55','AKs','AQs','AJs','ATs','A9s','A8s','A5s','A4s','KQs','KJs','KTs','K9s','QJs','QTs','Q9s','JTs','J9s','T9s','98s','AKo','AQo','AJo','ATo','KQo','KJo'],
            call: [], 
            limp: ['44','33','22','87s','76s','65s','54s'],
            info: "🔥 <b>Chiến lược MP (LoJack):</b> <span style='color:#3498db'>Vùng Limp mở khóa:</span> Nếu có 1-2 người Limp trước đó, hãy Over-Limp các đôi nhỏ lẻ hoặc bài đồng chất dây để bẫy sập đối thủ."
        },
        'hj': {
            raise: ['AA','KK','QQ','JJ','TT','99','88','77','66','55','44','33','22','AKs','AQs','AJs','ATs','A9s','A8s','A7s','A6s','A5s','A4s','A3s','A2s','KQs','KJs','KTs','K9s','K8s','K7s','K6s','K5s','QJs','QTs','Q9s','Q8s','JTs','J9s','J8s','T9s','T8s','98s','87s','76s','65s','AKo','AQo','AJo','ATo','A9o','A8o','KQo','KJo','KTo','QJo','QTo','JTo'],
            call: [], 
            limp: ['54s','64s','75s','86s'],
            info: "🔥 <b>Chiến lược HJ (Hijack):</b> Nếu trước bạn chưa ai Raise, hãy nã đạn để cướp pot. <span style='color:#3498db'>Limp:</span> Khi cả bàn đánh hiền."
        },
        'co': {
            raise: ['AA','KK','QQ','JJ','TT','99','88','77','66','55','44','33','22','AKs','AQs','AJs','ATs','A9s','A8s','A7s','A6s','A5s','A4s','A3s','A2s','KQs','KJs','KTs','K9s','K8s','K7s','K6s','K5s','K4s','K3s','K2s','QJs','QTs','Q9s','Q8s','Q7s','Q6s','Q5s','JTs','J9s','J8s','J7s','T9s','T8s','T7s','98s','97s','87s','86s','76s','65s','54s','AKo','AQo','AJo','ATo','A9o','A8o','A7o','A5o','KQo','KJo','KTo','K9o','QJo','QTo','Q9o','JTo','J9o','T9o'],
            call: [], 
            limp: ['64s','75s','86s','97s','T8s','J8s','Q8s'],
            info: "🔥 <b>Chiến lược CO (Cutoff):</b> Cực mạnh để Steal. Chỉ còn BTN và Blind phía sau. Ép họ Fold bằng cách mở Raise cực rộng (30-35%)."
        },
        'btn': {
            raise: ['AA','KK','QQ','JJ','TT','99','88','77','66','55','44','33','22','AKs','AQs','AJs','ATs','A9s','A8s','A7s','A6s','A5s','A4s','A3s','A2s','KQs','KJs','KTs','K9s','K8s','K7s','K6s','K5s','K4s','K3s','K2s','QJs','QTs','Q9s','Q8s','Q7s','Q6s','Q5s','Q4s','Q3s','Q2s','JTs','J9s','J8s','J7s','J6s','J5s','T9s','T8s','T7s','T6s','98s','97s','96s','87s','86s','76s','75s','65s','54s','AKo','AQo','AJo','ATo','A9o','A8o','A7o','A6o','A5o','A4o','A3o','A2o','KQo','KJo','KTo','K9o','K8o','QJo','QTo','Q9o','Q8o','JTo','J9o','T9o','98o'],
            call: [], 
            limp: [],
            info: "🔥 <b>Chiến lược BTN (Button):</b> Range Raise phủ nửa bảng. Hãy trừng phạt tất cả những kẻ Limp ở trước bằng cú Raise to, hoặc Steal tiền mù."
        },
        'sb': {
            raise: ['AA','KK','QQ','JJ','TT','99','88','77','AKs','AQs','AJs','ATs','A9s','KQs','KJs','KTs','QJs','QTs','JTs','AKo','AQo','AJo','ATo','KQo'],
            call: [], 
            limp: ['66','55','44','33','22','A8s','A7s','A6s','A5s','A4s','A3s','A2s','K9s','K8s','K7s','K6s','K5s','K4s','Q9s','Q8s','Q7s','J9s','J8s','T9s','T8s','98s','87s','76s','65s','54s','A9o','A8o','A7o','A5o','KJo','KTo','QJo','QTo','JTo','T9o'],
            info: "🔥 <b>Chiến lược SB:</b> <span style='color:#3498db'>Vùng Complete:</span> Nếu có 2-3 người Limp, hãy bỏ thêm 0.5 BB vào xem cho vui. Nhưng tuyệt đối ĐỪNG LIMP khi là First-in."
        },
        'bb': {
            raise: ['AA','KK','QQ','JJ','TT','AKs','AQs','AJs','AKo','AQo','A5s','A4s'], 
            call: ['99','88','77','66','55','44','33','22','ATs','A9s','A8s','A7s','A6s','A3s','A2s','KQs','KJs','KTs','K9s','K8s','K7s','K6s','K5s','K4s','K3s','K2s','QJs','QTs','Q9s','Q8s','Q7s','Q6s','Q5s','Q4s','Q3s','Q2s','JTs','J9s','J8s','J7s','T9s','T8s','T7s','98s','97s','87s','86s','76s','75s','65s','64s','54s','53s','43s','AJo','ATo','A9o','A8o','KQo','KJo','KTo','QJo','QTo','JTo','T9o'],
            limp: [],
            info: "🔥 <b>Chiến lược BB (Defense):</b> <span style='color:#2ecc71'>Màu Xanh lá = Call vs Raise:</span> Vì tiền mù đã nằm trên bàn, bạn có quyền phòng thủ Call rất rộng để không bị bắt nạt."
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
            cell.className = `cell state-3 ${extraClass}`; // Mặc định Fold
            cell.dataset.state = 3;
            cell.dataset.hand = hand;
            cell.textContent = hand;
            matrixEl.appendChild(cell);
        }
    }

    document.getElementById('positionSelect').addEventListener('change', function() {
        const val = this.value;
        const data = tagRanges[val];
        document.getElementById('range-info').innerHTML = data.info;

        const cells = document.querySelectorAll('.cell');
        cells.forEach(cell => {
            const hand = cell.dataset.hand;
            let newState = 3; // Fold
            if (data.raise && data.raise.includes(hand)) newState = 1; 
            else if (data.call && data.call.includes(hand)) newState = 2; 
            else if (data.limp && data.limp.includes(hand)) newState = 4;
            
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
