# webxr-map1
WebXR用の日本地図可視化
<!-- ブロック1 -->
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>都道府県データ 3D可視化システム</title>
    <script src="https://aframe.io/releases/1.7.0/aframe.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        #controls {
            position: absolute;
            top: 10px;
            left: 10px;
            z-index: 999;
            background: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 10px;
            border-radius: 5px;
            font-family: Arial, sans-serif;
            max-width: 300px;
        }
        #info {
            position: absolute;
            bottom: 10px;
            left: 10px;
            z-index: 999;
            background: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 10px;
            border-radius: 5px;
            font-family: Arial, sans-serif;
        }
        #legend {
            position: absolute;
            top: 10px;
            right: 10px;
            z-index: 999;
            background: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 10px;
            border-radius: 5px;
            font-family: Arial, sans-serif;
        }
        #correlation {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 1000;
            background: rgba(0, 0, 0, 0.9);
            color: white;
            padding: 20px;
            border-radius: 5px;
            font-family: Arial, sans-serif;
            max-width: 800px;
            display: none;
            text-align: center;
        }
        #correlation-close {
            position: absolute;
            top: 10px;
            right: 10px;
            background: none;
            border: none;
            color: white;
            font-size: 18px;
            cursor: pointer;
        }
        #selectors {
            position: absolute;
            bottom: 10px;
            right: 10px;
            z-index: 999;
            background: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 10px;
            border-radius: 5px;
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            gap: 10px;
            max-width: 500px;
        }
        .selector-container {
            display: flex;
            flex-direction: column;
            gap: 5px;
        }
        .selector-title {
            font-weight: bold;
            margin-bottom: 5px;
        }
        .button-container {
            display: flex;
            flex-wrap: wrap;
            gap: 5px;
            max-height: 100px;
            overflow-y: auto;
        }
        .legend-item {
            display: flex;
            align-items: center;
            margin: 5px 0;
        }
        .legend-color {
            width: 20px;
            height: 20px;
            margin-right: 10px;
        }
        .button {
            background-color: #4CAF50;
            border: none;
            color: white;
            padding: 8px 16px;
            text-align: center;
            text-decoration: none;
            display: inline-block;
            font-size: 14px;
            margin: 4px 2px;
            cursor: pointer;
            border-radius: 4px;
        }
        .select-button {
            background-color: #2196F3;
            border: none;
            color: white;
            padding: 5px 10px;
            margin: 2px;
            cursor: pointer;
            border-radius: 3px;
            font-size: 12px;
        }
        .select-button.active {
            background-color: #FF5722;
        }
        .slider-container {
            margin: 10px 0;
        }
        .data-selector {
            margin: 10px 0;
        }
        .tab-container {
            display: flex;
            margin-bottom: 10px;
        }
        .tab {
            flex: 1;
            padding: 8px;
            text-align: center;
            background-color: #555;
            cursor: pointer;
        }
        .tab.active {
            background-color: #007BFF;
        }
        .correlation-result {
            margin: 15px 0;
            font-size: 16px;
        }
        .correlation-value {
            font-weight: bold;
            font-size: 24px;
            color: #4CAF50;
        }
        .correlation-interpretation {
            font-style: italic;
            margin-top: 10px;
        }
        .p-value {
            margin-top: 5px;
            font-size: 14px;
        }
        .chart-container {
            width: 100%;
            max-width: 700px;
            height: 400px;
            margin: 15px auto;
            background-color: rgba(255, 255, 255, 0.9);
            border-radius: 5px;
            padding: 10px;
        }
        .chart-tabs {
            display: flex;
            justify-content: center;
            margin-bottom: 10px;
        }
        .chart-tab {
            padding: 8px 15px;
            margin: 0 5px;
            background-color: #555;
            color: white;
            border: none;
            border-radius: 3px;
            cursor: pointer;
        }
        .chart-tab.active {
            background-color: #007BFF;
        }
    </style>
</head>

<body>
    <div id="controls">
        <h3>都道府県データ可視化システム</h3>
        <p>CSVファイル選択（県名,年,幸福度順位,日照時間,緯度,経度）</p>
        <input type="file" id="csvFile" accept=".csv">
        <div class="slider-container">
            <p>高さ倍率: <input type="range" id="heightFactor" min="1" max="50" value="10"> <span id="factorValue">10</span></p>
        </div>
        <div class="data-selector">
            <div class="tab-container">
                <div class="tab active" id="tab-happiness">幸福度順位</div>
                <div class="tab" id="tab-sunshine">日照時間</div>
            </div>
        </div>
        <button id="playAnimation" class="button">年ごとに再生</button>
        <button id="showAllPrefectures" class="button">全県表示</button>
        <button id="calculateCorrelation" class="button">相関分析</button>
    </div>

    <div id="legend">
        <h3>凡例</h3>
        <div class="legend-item" id="happiness-legend">
            <div class="legend-color" style="background-color: #4287f5;"></div>
            <span>幸福度順位（低いほど幸福）</span>
        </div>
        <div class="legend-item" id="sunshine-legend" style="display: none;">
            <div class="legend-color" style="background-color: #f5a742;"></div>
            <span>日照時間（高いほど長い）</span>
        </div>
    </div>

    <div id="info">
        <p>操作方法: WASDキーで移動、マウスドラッグで視点変更</p>
    </div>

    <div id="correlation">
        <button id="correlation-close">×</button>
        <h2>相関分析</h2>
        <div class="chart-tabs">
            <button class="chart-tab active" id="tab-sunshine-correlation">幸福度と日照時間</button>
            <button class="chart-tab" id="tab-longitude-correlation">幸福度と経度</button>
            <button class="chart-tab" id="tab-latitude-correlation">幸福度と緯度</button>
        </div>
        <div class="chart-container">
            <canvas id="correlationChart"></canvas>
        </div>
        <div id="correlation-content"></div>
    </div>

    <div id="selectors">
        <div class="selector-container">
            <div class="selector-title">表示年を選択:</div>
            <div id="yearButtons" class="button-container"></div>
        </div>
        <div class="selector-container">
            <div class="selector-title">表示県を選択:</div>
            <div id="prefectureButtons" class="button-container"></div>
        </div>
    </div>

    <a-scene>
        <a-assets>
            <img id="japanMap" src="https://raw.githubusercontent.com/amiyuu/webxr-map/main/japan.png">
        </a-assets>

        <!-- 地図平面 -->
        <a-plane id="map" src="#japanMap" rotation="-90 29.999999999999996 0" position="-1.86863 0 2.00732" width="56.56" height="40"></a-plane>

        <!-- カメラ -->
        <a-entity id="rig" position="0 10 20">
            <a-camera look-controls wasd-controls></a-camera>
        </a-entity>

        <!-- 照明 -->
        <a-light type="ambient" color="#bbbbbb"></a-light>
        <a-light type="directional" position="1 1 1" color="#ffffff" intensity="0.6"></a-light>

        <a-sky color="#87CEEB"></a-sky>
    </a-scene>

    <script>
        // 日本の中心座標
        const centerLat = 36.0;
        const centerLon = 138.0;
        let heightMultiplier = 10; // 高さの倍率
        let currentYear = null; // 現在選択されている年
        let selectedPrefectures = []; // 選択されている県のリスト（空の場合は全県表示）
        let allYears = []; // データに含まれる年のリスト
        let allPrefectures = []; // データに含まれる県のリスト
        let isPlaying = false; // アニメーション再生中かどうか
        let playInterval; // アニメーション再生のためのインターバル
        let currentDataType = 'happiness'; // 表示するデータタイプ（happiness または sunshine）
        let allData = []; // CSVから読み込んだ全データ
        let correlationChart = null; // Chart.jsのインスタンス
        let currentCorrelationType = 'sunshine'; // 現在選択されている相関分析のタイプ（sunshine, longitude, latitude）

        // 高さ倍率スライダーの処理
        const heightSlider = document.getElementById('heightFactor');
        const factorValue = document.getElementById('factorValue');
        
        heightSlider.addEventListener('input', function() {
            heightMultiplier = parseInt(this.value);
            factorValue.textContent = heightMultiplier;
            
            // もし現在データが表示されていれば、再描画
            if (allData.length > 0 && currentYear) {
                updateVisualization();
            }
        });

        // データタブの切り替え
        document.getElementById('tab-happiness').addEventListener('click', function() {
            if (currentDataType !== 'happiness') {
                currentDataType = 'happiness';
                document.getElementById('tab-happiness').classList.add('active');
                document.getElementById('tab-sunshine').classList.remove('active');
                document.getElementById('happiness-legend').style.display = 'flex';
                document.getElementById('sunshine-legend').style.display = 'none';
                
                if (allData.length > 0 && currentYear) {
                    updateVisualization();
                }
            }
        });

        document.getElementById('tab-sunshine').addEventListener('click', function() {
            if (currentDataType !== 'sunshine') {
                currentDataType = 'sunshine';
                document.getElementById('tab-sunshine').classList.add('active');
                document.getElementById('tab-happiness').classList.remove('active');
                document.getElementById('happiness-legend').style.display = 'none';
                document.getElementById('sunshine-legend').style.display = 'flex';
                
                if (allData.length > 0 && currentYear) {
                    updateVisualization();
                }
            }
        });

        // 相関分析タブの切り替え
        document.getElementById('tab-sunshine-correlation').addEventListener('click', function(event) {
            event.stopPropagation();
            if (currentCorrelationType !== 'sunshine') {
                currentCorrelationType = 'sunshine';
                updateCorrelationTabs();
                updateCorrelationChart();
                displayCorrelationResults(); // 相関結果の表示も更新
            }
        });

        document.getElementById('tab-longitude-correlation').addEventListener('click', function(event) {
            event.stopPropagation();
            if (currentCorrelationType !== 'longitude') {
                currentCorrelationType = 'longitude';
                updateCorrelationTabs();
                updateCorrelationChart();
                displayCorrelationResults(); // 相関結果の表示も更新
            }
        });

        document.getElementById('tab-latitude-correlation').addEventListener('click', function(event) {
            event.stopPropagation();
            if (currentCorrelationType !== 'latitude') {
                currentCorrelationType = 'latitude';
                updateCorrelationTabs();
                updateCorrelationChart();
                displayCorrelationResults(); // 相関結果の表示も更新
            }
        });

        // 相関分析タブの更新
        function updateCorrelationTabs() {
            document.querySelectorAll('.chart-tab').forEach(tab => {
                tab.classList.remove('active');
            });
            document.getElementById(`tab-${currentCorrelationType}-correlation`).classList.add('active');
        }

        // 全県表示ボタン
        document.getElementById('showAllPrefectures').addEventListener('click', function() {
            // 全ての県ボタンから active クラスを削除
            const prefButtons = document.querySelectorAll('.pref-button');
            prefButtons.forEach(btn => btn.classList.remove('active'));
            
            // 選択県をリセット
            selectedPrefectures = [];
            
            // 再描画
            if (allData.length > 0 && currentYear) {
                updateVisualization();
            }
        });

        // 再生ボタンの処理
        document.getElementById('playAnimation').addEventListener('click', function() {
            if (isPlaying) {
                stopAnimation();
                this.textContent = "年ごとに再生";
            } else {
                startAnimation();
                this.textContent = "停止";
            }
        });

        // 相関分析ボタンの処理
        document.getElementById('calculateCorrelation').addEventListener('click', function() {
            if (allData.length === 0) {
                alert('データがありません。CSVファイルを読み込んでください。');
                return;
            }
            
            // 相関分析チャートの更新
            updateCorrelationChart();
            
            // 相関分析の結果表示
            displayCorrelationResults();
            
            // 相関分析ダイアログを表示
            document.getElementById('correlation').style.display = 'block';
        });

        // 相関分析ダイアログの閉じるボタン
        document.getElementById('correlation-close').addEventListener('click', function() {
            document.getElementById('correlation').style.display = 'none';
        });

        // CSVファイル読み込み処理
        document.getElementById('csvFile').addEventListener('change', function (e) {
            const file = e.target.files[0];

            if (file) {
                const reader = new FileReader();
                reader.onload = function (e) {
                    const text = e.target.result;
                    const rawData = parseCSV(text);
                    
                    // 緯度経度の欠損値を補完
                    allData = fillMissingCoordinates(rawData);
                    
                    // データから年と県のリストを抽出
                    allYears = [...new Set(allData.map(item => item.year))].sort();
                    allPrefectures = [...new Set(allData.map(item => item.name))].sort();
                    
                    // 初期値として最初の年を設定
                    if (allYears.length > 0) {
                        currentYear = allYears[0];
                    }
                    
                    // 年と県のボタンを作成
                    createYearButtons();
                    createPrefectureButtons();
                    
                    // 現在の年のデータを表示
                    updateVisualization();
                };
                reader.readAsText(file);
            }
        });

        // 相関分析チャートの更新
        function updateCorrelationChart() {
            if (allData.length === 0) return;

            // 選択された都道府県のデータを使用
            let targetData = [...allData];
            if (selectedPrefectures.length > 0) {
                targetData = targetData.filter(item => selectedPrefectures.includes(item.name));
            }

            // 現在の年のデータのみをフィルタリングする場合は以下を使う
            if (currentYear) {
                targetData = targetData.filter(item => item.year === currentYear);
            }

            // 有効なデータポイントのみを使用
            const validData = targetData.filter(d => {
                // 幸福度チェック
                if (isNaN(d.happiness) || d.happiness === null) return false;
                
                // 選択された変数に応じたチェック
                if (currentCorrelationType === 'sunshine') {
                    return !isNaN(d.sunshine) && d.sunshine !== null;
                } else if (currentCorrelationType === 'longitude') {
                    return !isNaN(d.lon) && d.lon !== null;
                } else if (currentCorrelationType === 'latitude') {
                    return !isNaN(d.lat) && d.lat !== null;
                }
                return false;
            });

            // 散布図データの準備
            const scatterData = validData.map(d => {
                let xValue;
                if (currentCorrelationType === 'sunshine') {
                    xValue = d.sunshine;
                } else if (currentCorrelationType === 'longitude') {
                    xValue = d.lon;
                } else if (currentCorrelationType === 'latitude') {
                    xValue = d.lat;
                }
                
                return {
                    x: xValue,
                    y: d.happiness,
                    label: d.name
                };
            });

            // Chart.jsの設定
            const ctx = document.getElementById('correlationChart').getContext('2d');
            
            // 既存のチャートがあれば破棄
            if (correlationChart) {
                correlationChart.destroy();
            }

            // タイトルのテキスト設定
            let titleText;
            switch (currentCorrelationType) {
                case 'sunshine':
                    titleText = '幸福度順位と日照時間の相関';
                    break;
                case 'longitude':
                    titleText = '幸福度順位と経度の相関';
                    break;
                case 'latitude':
                    titleText = '幸福度順位と緯度の相関';
                    break;
            }

            // X軸のラベル設定
            let xAxisLabel;
            switch (currentCorrelationType) {
                case 'sunshine':
                    xAxisLabel = '日照時間 (時間)';
                    break;
                case 'longitude':
                    xAxisLabel = '経度 (度)';
                    break;
                case 'latitude':
                    xAxisLabel = '緯度 (度)';
                    break;
            }

            correlationChart = new Chart(ctx, {
                type: 'scatter',
                data: {
                    datasets: [{
                        label: currentYear ? `${currentYear}年 相関データ` : '全年データ',
                        data: scatterData,
                        backgroundColor: 'rgba(66, 135, 245, 0.7)',
                        borderColor: 'rgba(66, 135, 245, 1)',
                        pointRadius: 6,
                        pointHoverRadius: 8
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    const point = context.raw;
                                    return `${point.label}: (${point.x.toFixed(2)}, ${point.y})`;
                                }
                            }
                        },
                        title: {
                            display: true,
                            text: titleText,
                            font: {
                                size: 16
                            }
                        },
                        legend: {
                            display: true
                        }
                    },
                    scales: {
                        x: {
                            title: {
                                display: true,
                                text: xAxisLabel
                            }
                        },
                        y: {
                            title: {
                                display: true,
                                text: '幸福度順位 (低いほど幸福)'
                            },
                            reverse: false // 高い値ほど幸福度が低い
                        }
                    }
                }
            });
        }

        // スピアマンの順位相関係数を計算
        function calculateSpearmanCorrelation(dataArray, xVariable) {
            // 配列が空の場合
            if (!dataArray || dataArray.length < 2) {
                return { coefficient: null, pvalue: null, n: 0 };
            }
            
            // 有効なデータポイントのみを使用
            const validData = dataArray.filter(d => {
                // 幸福度チェック
                if (isNaN(d.happiness) || d.happiness === null) return false;
                
                // 選択された変数に応じたチェック
                if (xVariable === 'sunshine') {
                    return !isNaN(d.sunshine) && d.sunshine !== null;
                } else if (xVariable === 'longitude') {
                    return !isNaN(d.lon) && d.lon !== null;
                } else if (xVariable === 'latitude') {
                    return !isNaN(d.lat) && d.lat !== null;
                }
                return false;
            });
            
            const n = validData.length;
            
            // データが不十分な場合
            if (n < 5) {
                return { coefficient: null, pvalue: null, n: n };
            }
            
            // 幸福度と変数の値を取得
            const happiness = validData.map(d => d.happiness);
            let xValues;
            
            // 変数タイプに応じて値を抽出
            if (xVariable === 'sunshine') {
                xValues = validData.map(d => d.sunshine);
            } else if (xVariable === 'longitude') {
                xValues = validData.map(d => d.lon);
            } else if (xVariable === 'latitude') {
                xValues = validData.map(d => d.lat);
            } else {
                return { coefficient: null, pvalue: null, n: 0 };
            }
            
            // 幸福度のランキング
            const happinessRanks = getRanks(happiness);
            
            // 変数のランキング
            const xRanks = getRanks(xValues);
            
            // 2つのランキングの差の二乗和を計算
            let sumD2 = 0;
            for (let i = 0; i < n; i++) {
                const d = happinessRanks[i] - xRanks[i];
                sumD2 += d * d;
            }
            
            // スピアマンの相関係数を計算
            const rs = 1 - (6 * sumD2) / (n * (n * n - 1));
            
            // p値の計算（近似計算）
            // t統計量の計算
            const t = rs * Math.sqrt((n - 2) / (1 - rs * rs));
            
            // 自由度n-2のt分布に基づくp値（両側検定）
            let pvalue;
            if (n > 30) {
                // 大標本の場合は正規分布に近似
                pvalue = 2 * (1 - normalcdf(Math.abs(t)));
            } else {
                // 小標本の場合はt分布を使用（近似値）
                pvalue = 2 * (1 - tcdf(Math.abs(t), n - 2));
            }
            
            return { coefficient: rs, pvalue: pvalue, n: n };
        }

        // 標準正規分布の累積分布関数（CDF）
        function normalcdf(x) {
            const a1 = 0.254829592;
            const a2 = -0.284496736;
            const a3 = 1.421413741;
            const a4 = -1.453152027;
            const a5 = 1.061405429;
            const p = 0.3275911;
            
            const sign = (x < 0) ? -1 : 1;
            x = Math.abs(x) / Math.sqrt(2);
            
            const t = 1.0 / (1.0 + p * x);
            const y = 1.0 - (((((a5 * t + a4) * t) + a3) * t + a2) * t + a1) * t * Math.exp(-x * x);
            
            return 0.5 * (1 + sign * y);
        }

        // t分布の累積分布関数の近似
        function tcdf(t, df) {
            // ベータ分布の不完全ベータ関数を使った近似
            // この実装は簡略化されています
            const x = df / (df + t * t);
            if (df < 1) {
                return 0.5;
            } else {
                // 自由度が大きい場合は標準正規分布に近似
                return 1 - normalcdf(t);
            }
        }

        // 配列の順位を計算する関数
        function getRanks(array) {
            // 値とインデックスのペアを作成
            const pairs = array.map((value, index) => ({ value, index }));
            
            // 値に基づいてソート
            pairs.sort((a, b) => a.value - b.value);
            
            // ランクを割り当て（同点の場合は平均ランクを使用）
            const ranks = new Array(array.length);
            let i = 0;
            while (i < pairs.length) {
                const value = pairs[i].value;
                let j = i + 1;
                while (j < pairs.length && pairs[j].value === value) {
                    j++;
                }
                
                // 同点の場合は平均ランクを使用
                const rank = (i + j - 1) / 2 + 1;
                for (let k = i; k < j; k++) {
                    ranks[pairs[k].index] = rank;
                }
                
                i = j;
            }
            
            return ranks;
        }

        // 相関分析を実行する関数
        function performCorrelationAnalysis(xVariable) {
            // まずデータチェック
            if (allData.length === 0) {
                return {
                    success: false,
                    message: "データがありません。"
                };
            }
            
            // 選択された都道府県のデータを使用（全選択がない場合は全データ）
            let targetData = [...allData];
            if (selectedPrefectures.length > 0) {
                targetData = targetData.filter(item => selectedPrefectures.includes(item.name));
            }
            
            // 現在の年のデータのみを使用する場合は以下を使う
            if (currentYear) {
                targetData = targetData.filter(item => item.year === currentYear);
            }
            
            // 相関分析の実行
            const result = calculateSpearmanCorrelation(targetData, xVariable);
            
            if (result.coefficient === null) {
                return {
                    success: false,
                    message: `データが不十分です。（有効データ数: ${result.n}）`
                };
            }
            
            // 相関係数の解釈
            let interpretation;
            const coef = Math.abs(result.coefficient);
            
            if (coef >= 0.8) {
                interpretation = "非常に強い相関";
            } else if (coef >= 0.6) {
                interpretation = "強い相関";
            } else if (coef >= 0.4) {
                interpretation = "中程度の相関";
            } else if (coef >= 0.2) {
                interpretation = "弱い相関";
            } else {
                interpretation = "ほとんど相関なし";
            }
            
            // 相関の方向
            const direction = result.coefficient > 0 ? "正の" : "負の";
            
            // 有意性
            const significant = result.pvalue < 0.05;
            
            return {
                success: true,
                coefficient: result.coefficient,
                pvalue: result.pvalue,
                n: result.n,
                interpretation: interpretation,
                direction: direction,
                significant: significant,
                xVariable: xVariable
            };
        }

        // 相関分析結果を表示する関数
        function displayCorrelationResults() {
            const correlationDialog = document.getElementById('correlation');
            const contentDiv = document.getElementById('correlation-content');
            
            // 現在選択されている相関タイプを使用して分析を実行
            const correlationResult = performCorrelationAnalysis(currentCorrelationType);
            
            if (!correlationResult || !correlationResult.success) {
                contentDiv.innerHTML = `
                    <p class="correlation-result">
                        ${correlationResult ? correlationResult.message : "分析を実行できませんでした。"}
                    </p>
                `;
            } else {
                const coefficient = correlationResult.coefficient.toFixed(3);
                const pvalue = correlationResult.pvalue < 0.001 ? 
                    "< 0.001" : correlationResult.pvalue.toFixed(3);
                
                // 相関の解釈
                let message;
                let variableName;
                
                switch (correlationResult.xVariable) {
                    case 'sunshine':
                        variableName = "日照時間";
                        break;
                    case 'longitude':
                        variableName = "経度（東に行くほど大きい値）";
                        break;
                    case 'latitude':
                        variableName = "緯度（北に行くほど大きい値）";
                        break;
                    default:
                        variableName = correlationResult.xVariable;
                }
                
                if (correlationResult.significant) {
                    message = `幸福度順位と${variableName}の間には、統計的に有意な ${correlationResult.direction}${correlationResult.interpretation} があります。`;
                    
                    // 幸福度順位は低いほど幸福なので、正の相関と負の相関の解釈が逆になる
                    if (correlationResult.coefficient > 0) {
                        message += `
つまり、${variableName}が大きいほど幸福度順位の数値が大きくなる（幸福度が低くなる）傾向があります。`;
                    } else {
                        message += `<br>つまり、${variableName}が大きいほど幸福度順位の数値が小さくなる（幸福度が高くなる）傾向があります。`;
                    }
                } else {
                    message = `幸福度順位と${variableName}の間には、統計的に有意な相関は見られません。`;
                }
                
                contentDiv.innerHTML = `
                    <p class="correlation-result">
                        スピアマンの順位相関係数: <span class="correlation-value">${coefficient}</span>
                    </p>
                    <p class="p-value">
                        p値: ${pvalue} ${correlationResult.significant ? "(有意)" : "(非有意)"}
                    </p>
                    <p class="correlation-interpretation">
                        ${message}
                    </p>
                    <p>
                        サンプル数: ${correlationResult.n} ${currentYear ? `(${currentYear}年)` : "(全年)"}
                    </p>
                `;
            }
            
            correlationDialog.style.display = 'block';
        }

        // 欠損している緯度経度を補完する関数
        function fillMissingCoordinates(data) {
            const coordinatesMap = {};
            
            // 最初に県ごとに緯度経度が存在するデータを記録
            data.forEach(item => {
                if (!isNaN(item.lat) && !isNaN(item.lon) && 
                    item.lat !== null && item.lon !== null) {
                    if (!coordinatesMap[item.name]) {
                        coordinatesMap[item.name] = { lat: item.lat, lon: item.lon };
                    }
                }
            });
            
            // 欠損値を補完
            return data.map(item => {
                const newItem = { ...item };
                
                // 緯度または経度が欠損している場合
                if (isNaN(newItem.lat) || isNaN(newItem.lon) || 
                    newItem.lat === null || newItem.lon === null) {
                    // 同じ県の座標が保存されていれば、それを使用
                    if (coordinatesMap[newItem.name]) {
                        newItem.lat = coordinatesMap[newItem.name].lat;
                        newItem.lon = coordinatesMap[newItem.name].lon;
                    }
                }
                
                return newItem;
            });
        }

        // 年ボタンを作成する関数
        function createYearButtons() {
            const yearButtonsContainer = document.getElementById("yearButtons");
            yearButtonsContainer.innerHTML = "";
            
            allYears.forEach(year => {
                const button = document.createElement("button");
                button.textContent = year;
                button.classList.add("select-button", "year-button");
                if (year === currentYear) {
                    button.classList.add("active");
                }
                
                button.addEventListener("click", function(event) {
                    // イベントバブリングを停止
                    event.stopPropagation();
                    
                    // アニメーション中は年の切り替えができないようにする
                    if (isPlaying) return;
                    
                    currentYear = year;
                    
                    // 全ての年ボタンからactiveクラスを削除
                    const yearButtons = document.querySelectorAll(".year-button");
                    yearButtons.forEach(btn => btn.classList.remove("active"));
                    
                    // クリックされたボタンにactiveクラスを追加
                    this.classList.add("active");
                    
                    // 選択された年のデータを表示
                    updateVisualization();
                });
                
                yearButtonsContainer.appendChild(button);
            });
        }

        // 県ボタンを作成する関数
        function createPrefectureButtons() {
            const prefButtonsContainer = document.getElementById("prefectureButtons");
            prefButtonsContainer.innerHTML = "";
            
            allPrefectures.forEach(prefecture => {
                const button = document.createElement("button");
                button.textContent = prefecture;
                button.classList.add("select-button", "pref-button");
                
                button.addEventListener("click", function(event) {
                    // イベントバブリングを停止
                    event.stopPropagation();
                    
                    // ボタンのアクティブ状態を切り替え
                    this.classList.toggle("active");
                    
                    // 選択された県のリストを更新
                    if (this.classList.contains("active")) {
                        // 追加
                        if (!selectedPrefectures.includes(prefecture)) {
                            selectedPrefectures.push(prefecture);
                        }
                    } else {
                        // 削除
                        const index = selectedPrefectures.indexOf(prefecture);
                        if (index > -1) {
                            selectedPrefectures.splice(index, 1);
                        }
                    }
                    
                    // データを表示
                    updateVisualization();
                });
                
                prefButtonsContainer.appendChild(button);
            });
        }

        // 表示を更新する関数
        function updateVisualization() {
            if (allData.length === 0 || !currentYear) return;
            
            // 指定された年のデータをフィルタリング
            let yearData = allData.filter(item => item.year === currentYear);
            
            // 選択されている県がある場合はフィルタリング
            if (selectedPrefectures.length > 0) {
                yearData = yearData.filter(item => selectedPrefectures.includes(item.name));
            }
            
            // データを地図上に表示
            displayOnMap(yearData);
        }

        // アニメーションを開始する関数
        function startAnimation() {
            if (isPlaying || !allYears || allYears.length === 0) return;
            
            isPlaying = true;
            let currentIndex = allYears.indexOf(currentYear);
            if (currentIndex === -1) currentIndex = 0;
            
            playInterval = setInterval(function() {
                currentIndex = (currentIndex + 1) % allYears.length;
                const nextYear = allYears[currentIndex];
                
                // 現在の年を更新
                currentYear = nextYear;
                
                // 年ボタンのアクティブ状態を更新
                const yearButtons = document.querySelectorAll(".year-button");
                yearButtons.forEach(btn => {
                    if (btn.textContent === nextYear) {
                        btn.classList.add("active");
                    } else {
                        btn.classList.remove("active");
                    }
                });
                
                // データを表示
                updateVisualization();
                
                // 最後の年まで行ったら停止
                if (currentIndex === allYears.length - 1) {
                    setTimeout(stopAnimation, 1000);
                }
            }, 2000); // 2秒ごとに年を切り替え
        }

        // アニメーションを停止する関数
        function stopAnimation() {
            if (!isPlaying) return;
            
            clearInterval(playInterval);
            isPlaying = false;
            document.getElementById("playAnimation").textContent = "年ごとに再生";
        }

        // CSVパース関数 - 修正版
        function parseCSV(text) {
            // カンマで分割するが、文字列内のカンマはスキップする
            const parseCSVLine = (line) => {
                const values = [];
                let insideQuotes = false;
                let currentValue = '';
                
                for (let i = 0; i < line.length; i++) {
                    const char = line[i];
                    
                    if (char === '"' && (i === 0 || line[i-1] !== '\\')) {
                        insideQuotes = !insideQuotes;
                    } else if (char === ',' && !insideQuotes) {
                        values.push(currentValue.trim());
                        currentValue = '';
                    } else {
                        currentValue += char;
                    }
                }
                
                values.push(currentValue.trim());
                return values;
            };
            
            const lines = text.split('\n');
            const data = [];
            
            // ヘッダー行をスキップするためのインデックスを1から開始
            for (let i = 1; i < lines.length; i++) {
                if (lines[i].trim() === '') continue;
                
                const values = parseCSVLine(lines[i]);
                if (values.length < 4) continue;  // 少なくとも4つの値が必要
                
                let lat = null;
                let lon = null;
                
                // 緯度経度の処理
                if (values.length >= 5) {
                    lat = parseFloat(values[4]);
                    if (isNaN(lat) || values[4].includes('同じ')) {
                        lat = null; // 「同じ」の場合は後で補完
                    }
                }
                
                if (values.length >= 6) {
                    lon = parseFloat(values[5]);
                    if (isNaN(lon)) {
                        lon = null; // 不正な値の場合は後で補完
                    }
                }
                
                const item = {
                    name: values[0].trim(),
                    year: values[1].trim(),
                    happiness: parseFloat(values[2]),
                    sunshine: parseFloat(values[3]),
                    lat: lat,
                    lon: lon
                };
                
                // 数値データが正しくパースされていることを確認
                if (!isNaN(item.happiness) && !isNaN(item.sunshine)) {
                    data.push(item);
                }
            }
            
            return data;
        }

       
        // 緯度経度からA-Frame座標へ変換
        function latLonToPosition(lat, lon) {
            // 経度は東西方向(x軸)、緯度は南北方向(z軸)に対応
            // 地図の回転に合わせて座標変換を調整
            const mapRotationDeg = -30; // マイナス30度の回転（時計回り）
            const mapRotation = mapRotationDeg * Math.PI / 180; // ラジアンに変換
            
            // 基本的な座標変換
            const baseX = (lon - centerLon) * 1.5;
            const baseZ = -(lat - centerLat) * 2; // 北が負のz方向
            
            // 沖縄の特別処理 - 沖縄を少し北に移動
            if (lat < 30) { // 沖縄の緯度はおよそ26度
                // 沖縄を北（Z軸のマイナス方向）に3単位移動
                const adjustedZ = baseZ + 17; // Z座標を北に移動（値を小さくする）
                // 沖縄のX座標を縮小して右に移動
                const adjustedX = baseX + 33; // X座標を右に移動

                // その他の調整は行わず、回転のみ適用
                const rotatedX = baseX * Math.cos(mapRotation) - adjustedX * Math.sin(mapRotation);
                const rotatedZ = baseZ * Math.sin(mapRotation) + adjustedZ * Math.cos(mapRotation);

                return { x: rotatedX, z: rotatedZ };
            }

            // 地図の回転を考慮して最終座標を計算（沖縄以外）
            const rotatedX = baseX * Math.cos(mapRotation) - baseZ * Math.sin(mapRotation);
            const rotatedZ = baseX * Math.sin(mapRotation) + baseZ * Math.cos(mapRotation);

            return { x: rotatedX, z: rotatedZ };
        }

        // データを地図上に表示
        function displayOnMap(data) {
            const scene = document.querySelector('a-scene');

            // 既存の要素を削除
            const oldElements = document.querySelectorAll('.datapoint');
            oldElements.forEach(el => el.parentNode.removeChild(el));
            
            if (data.length === 0) return;

            // 値の最大値と最小値を計算
            let minValue = Number.MAX_VALUE;
            let maxValue = Number.MIN_VALUE;
            
            data.forEach(item => {
                const value = currentDataType === 'happiness' ? item.happiness : item.sunshine;
                if (value < minValue) minValue = value;
                if (value > maxValue) maxValue = value;
            });
            
            const valueRange = maxValue - minValue;

            data.forEach(item => {
                // 緯度経度が正しく設定されているかチェック
                if (isNaN(item.lat) || isNaN(item.lon) || item.lat === null || item.lon === null) {
                    console.warn(`座標が不正: ${item.name}, ${item.year}`);
                    return;
                }
                
                const pos = latLonToPosition(item.lat, item.lon);
                
                // 表示するデータ値
                const value = currentDataType === 'happiness' ? item.happiness : item.sunshine;
                
                // 値の正規化（0-1の範囲に）
                let normalizedValue;
                if (currentDataType === 'happiness') {
                    // 幸福度は低い値ほど良いため、逆の正規化を適用
                    normalizedValue = valueRange > 0 ? 1 - ((value - minValue) / valueRange) : 0.5;
                } else {
                    normalizedValue = valueRange > 0 ? (value - minValue) / valueRange : 0.5;
                }
               
                // 値に基づいて高さを計算（最小1、最大はheightMultiplier）
                const height = 1 + normalizedValue * (heightMultiplier - 1);

                // 選択されたデータタイプに応じた色を設定
                const color = currentDataType === 'happiness' ? '#4287f5' : '#f5a742';

                // 円柱（データポイント）の作成
                const cylinder = document.createElement('a-cylinder');
                cylinder.classList.add('datapoint');
                cylinder.setAttribute('position', `${pos.x} ${height / 2} ${pos.z}`);
                cylinder.setAttribute('radius', '0.2');
                cylinder.setAttribute('height', height);
                cylinder.setAttribute('color', color);
                cylinder.setAttribute('shadow', 'cast: true');

                // アニメーションを追加
                const animation = document.createElement('a-animation');
                animation.setAttribute('attribute', 'height');
                animation.setAttribute('from', '0');
                animation.setAttribute('to', height);
                animation.setAttribute('dur', '1000');
                animation.setAttribute('easing', 'ease-out');
                cylinder.appendChild(animation);

                scene.appendChild(cylinder);

                // テキストラベル
                const text = document.createElement('a-text');
                text.classList.add('datapoint');
                text.setAttribute('value', `${item.name}\n${currentDataType === 'happiness' ? '幸福度順位: ' + item.happiness : '日照時間: ' + item.sunshine + '時間'}`);
                text.setAttribute('position', `${pos.x} ${height + 0.5} ${pos.z}`);
                text.setAttribute('color', '#ffffff');
                text.setAttribute('align', 'center');
                text.setAttribute('width', '10');
                text.setAttribute('side', 'double');
                text.setAttribute('look-at', '[camera]');
                scene.appendChild(text);
            });
        }
    </script>
</body>

</html>
