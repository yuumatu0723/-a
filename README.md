<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>株攻略事典 - Graph Engine</title>
    <!-- グラフライブラリ読み込み -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --accent: #4a90e2; --bg: #f0f4f8; --card: #ffffff; --star: #f5a623; }
        body { font-family: 'Helvetica Neue', Arial, sans-serif; background: var(--bg); margin: 0; padding-bottom: 100px; color: #333; }
        header { background: var(--accent); color: white; padding: 20px; text-align: center; font-size: 1.2rem; font-weight: bold; position: sticky; top: 0; z-index: 100; }
        
        /* カテゴリナビ */
        .nav-scroll { display: flex; overflow-x: auto; background: white; border-bottom: 1px solid #ddd; position: sticky; top: 60px; z-index: 90; }
        .nav-item { padding: 15px 20px; white-space: nowrap; cursor: pointer; font-size: 0.9rem; color: #666; border-bottom: 3px solid transparent; }
        .nav-item.active { color: var(--accent); border-bottom-color: var(--accent); font-weight: bold; }

        .container { padding: 15px; max-width: 600px; margin: auto; }
        
        /* 単語カード */
        .card { background: var(--card); border-radius: 12px; padding: 15px; margin-bottom: 15px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); }
        .card-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
        .word-name { font-size: 1.1rem; font-weight: bold; }
        .stars { color: var(--star); }
        .meaning { font-size: 0.9rem; line-height: 1.6; color: #555; margin-bottom: 10px; }

        /* グラフエリア */
        .chart-container { background: #f9f9f9; border-radius: 8px; padding: 10px; height: 200px; position: relative; }
        
        /* 追加フォーム */
        .fab { position: fixed; bottom: 25px; right: 25px; background: var(--accent); color: white; width: 60px; height: 60px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 30px; box-shadow: 0 4px 12px rgba(0,0,0,0.3); cursor: pointer; z-index: 1000; }
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.5); z-index: 1001; justify-content: center; align-items: center; }
        .modal-content { background: white; padding: 20px; border-radius: 15px; width: 90%; max-width: 400px; }
        input, select, textarea { width: 100%; margin-bottom: 12px; padding: 10px; box-sizing: border-box; border: 1px solid #ddd; border-radius: 6px; }
        .btn-submit { background: var(--accent); color: white; border: none; width: 100%; padding: 12px; border-radius: 6px; font-weight: bold; }
    </style>
</head>
<body>

<header>📈 株攻略事典 Pro</header>

<div class="nav-scroll" id="nav">
    <div class="nav-item active" onclick="setCategory('すべて')">すべて</div>
    <div class="nav-item" onclick="setCategory('チャート形状')">チャート形状</div>
    <div class="nav-item" onclick="setCategory('テクニカル')">テクニカル/足</div>
    <div class="nav-item" onclick="setCategory('ファンダメンタルズ')">ファンダ</div>
    <div class="nav-item" onclick="setCategory('運用・心理')">心理/ルール</div>
    <div class="nav-item" onclick="setCategory('その他')">その他</div>
</div>

<div class="container" id="app">
    <!-- カードがここに生成される -->
</div>

<div class="fab" onclick="document.getElementById('modal').style.display='flex'">＋</div>

<div class="modal" id="modal">
    <div class="modal-content">
        <h3>単語の追加</h3>
        <select id="f-cat">
            <option>チャート形状</option>
            <option>テクニカル</option>
            <option>ファンダメンタルズ</option>
            <option>運用・心理</option>
            <option>その他</option>
        </select>
        <input type="text" id="f-word" placeholder="単語名">
        <textarea id="f-mean" placeholder="意味・解説" rows="3"></textarea>
        <select id="f-imp">
            <option value="5">重要度：5</option>
            <option value="4">重要度：4</option>
            <option value="3">重要度：3</option>
            <option value="2">重要度：2</option>
            <option value="1">重要度：1</option>
        </select>
        <button class="btn-submit" onclick="addWord()">事典に保存</button>
        <button onclick="document.getElementById('modal').style.display='none'" style="border:none; background:none; width:100%; margin-top:10px; color:#999;">キャンセル</button>
    </div>
</div>

<script>
    // 初期データ（グラフデータ付き）
    const initialWords = [
        { cat: "チャート形状", word: "三尊（ヘッドアンドショルダー）", imp: 5, mean: "中央が一番高い3つの山。上昇が終わって下降に転じる時の強力なサイン。", chart: [10, 15, 12, 20, 12, 15, 10] },
        { cat: "チャート形状", word: "逆三尊", imp: 5, mean: "三尊の逆。底で出現すると強力な上昇サイン。買いたい人が増えている状態。", chart: [20, 15, 18, 10, 18, 15, 20] },
        { cat: "チャート形状", word: "ダブルトップ", imp: 5, mean: "M字の形。2回高値を試してダメだった証拠。天井になりやすい。", chart: [10, 20, 12, 20, 10] },
        { cat: "チャート形状", word: "三角保ち合い", imp: 4, mean: "値動きが収束。エネルギーが溜まっており、抜けた方に大きく飛ぶ。", chart: [10, 18, 12, 16, 13, 15, 14] },
        { cat: "テクニカル", word: "大陽線", imp: 5, mean: "一本の棒。強い買いの意志。これが出た後は上昇が続きやすい。", chart: [10, 11, 12, 13, 14, 25] },
        { cat: "ファンダメンタルズ", word: "CPI", imp: 5, mean: "物価指数。これが高いと金利が上がり、株が下がる要因になる超重要指標。" },
        { cat: "運用・心理", word: "損切り（2%ルール）", imp: 5, mean: "1回の負けを資産の2%に抑える。連敗しても生き残るための絶対ルール。" }
    ];

    let words = JSON.parse(localStorage.getItem('kabu_data')) || initialWords;
    let currentCat = "すべて";

    function setCategory(cat) {
        currentCat = cat;
        document.querySelectorAll('.nav-item').forEach(el => {
            el.classList.toggle('active', el.innerText === cat);
        });
        render();
    }

    function render() {
        const container = document.getElementById('app');
        container.innerHTML = "";

        const filtered = words.filter(w => currentCat === "すべて" || w.cat === currentCat);
        
        filtered.forEach((w, idx) => {
            const card = document.createElement('div');
            card.className = 'card';
            const starStr = "★".repeat(w.imp) + "☆".repeat(5 - w.imp);
            
            let chartHtml = w.chart ? `<div class="chart-container"><canvas id="chart-${idx}"></canvas></div>` : "";
            
            card.innerHTML = `
                <div class="card-header">
                    <span class="word-name">${w.word}</span>
                    <span class="stars">${starStr}</span>
                </div>
                <div class="meaning">${w.mean}</div>
                ${chartHtml}
                <div style="text-align:right; margin-top:10px;">
                    <button onclick="deleteWord(${idx})" style="color:#ccc; border:none; background:none; font-size:0.7rem;">削除</button>
                </div>
            `;
            container.appendChild(card);

            if(w.chart) {
                new Chart(document.getElementById(`chart-${idx}`), {
                    type: 'line',
                    data: {
                        labels: w.chart.map((_, i) => i),
                        datasets: [{
                            data: w.chart,
                            borderColor: '#4a90e2',
                            backgroundColor: 'rgba(74, 144, 226, 0.1)',
                            fill: true,
                            tension: 0.1,
                            pointRadius: 4
                        }]
                    },
                    options: {
                        maintainAspectRatio: false,
                        plugins: { legend: { display: false } },
                        scales: { x: { display: false }, y: { display: false } }
                    }
                });
            }
        });
    }

    function addWord() {
        const word = document.getElementById('f-word').value;
        const mean = document.getElementById('f-mean').value;
        const cat = document.getElementById('f-cat').value;
        const imp = parseInt(document.getElementById('f-imp').value);

        if(!word || !mean) return alert("入力してね！");

        words.push({ cat, word, mean, imp });
        localStorage.setItem('kabu_data', JSON.stringify(words));
        
        document.getElementById('f-word').value = "";
        document.getElementById('f-mean').value = "";
        document.getElementById('modal').style.display = 'none';
        render();
    }

    function deleteWord(idx) {
        if(confirm("消してもいい？")) {
            words.splice(idx, 1);
            localStorage.setItem('kabu_data', JSON.stringify(words));
            render();
        }
    }

    render();
</script>
</body>
</html>
