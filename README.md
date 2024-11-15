<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>에너지 & 물 절약 프로그램</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            background-color: #f4f4f4;
        }
        header {
            background-color: #4CAF50;
            color: white;
            padding: 20px;
            text-align: center;
        }
        .container {
            display: flex;
            flex-wrap: wrap;
            margin: 20px;
        }
        .section {
            flex: 1;
            margin: 10px;
            padding: 20px;
            background-color: white;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        .left, .right {
            flex-basis: 45%;
        }
        .bottom {
            width: 100%;
            margin-top: 20px;
        }
        h2 {
            color: #333;
            border-bottom: 1px solid #ddd;
            padding-bottom: 10px;
        }
        label, input, button {
            display: block;
            width: 100%;
            margin-bottom: 10px;
        }
        input {
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 5px;
        }
        .temp-buttons, .season-buttons {
            display: flex;
            gap: 10px;
        }
        .temp-buttons button, .season-buttons button {
            flex: 1;
            padding: 10px;
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
        }
        .temp-buttons button:hover, .season-buttons button:hover {
            background-color: #45a049;
        }
        .result {
            margin-top: 20px;
            padding: 10px;
            background-color: #f9f9f9;
            border: 1px solid #ddd;
            border-radius: 5px;
        }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
</head>
<body>

<header>
    <h1>에너지 & 물 절약 프로그램</h1>
</header>

<div class="container">
    <div class="section left">
        <h2>가전제품 사용 분석</h2>
        <label for="aircon">에어컨 사용 시간 (시간):</label>
        <input type="number" id="aircon" placeholder="에어컨 사용 시간 입력">

        <label for="fan">선풍기 사용 시간 (시간):</label>
        <input type="number" id="fan" placeholder="선풍기 사용 시간 입력">

        <label for="laptop">노트북 충전 시간 (시간):</label>
        <input type="number" id="laptop" placeholder="노트북 충전 시간 입력">

        <label for="phone">핸드폰 충전 시간 (시간):</label>
        <input type="number" id="phone" placeholder="핸드폰 충전 시간 입력">

        <label for="coffeePot">커피포트 사용 시간 (분):</label>
        <input type="number" id="coffeePot" placeholder="커피포트 사용 시간 입력">

        <label for="hairDryer">헤어드라이어 사용 시간 (분):</label>
        <input type="number" id="hairDryer" placeholder="헤어드라이어 사용 시간 입력">

        <button onclick="analyzeUsage()">가전제품 사용 분석</button>
        <div id="usageResult" class="result"></div>
    </div>

    <div class="section right">
        <h2>에어컨/히터 사용 제안</h2>
        <label for="temperature">실내 온도 입력:</label>
        <input type="number" id="temperature" placeholder="온도 입력">

        <label>온도 단위 선택:</label>
        <div class="temp-buttons">
            <button onclick="setUnit('C')">섭씨 (°C)</button>
            <button onclick="setUnit('F')">화씨 (°F)</button>
            <button onclick="setUnit('K')">켈빈 (K)</button>
        </div>

        <label>현재 계절 선택:</label>
        <div class="season-buttons">
            <button onclick="setSeason('봄')">봄</button>
            <button onclick="setSeason('여름')">여름</button>
            <button onclick="setSeason('가을')">가을</button>
            <button onclick="setSeason('겨울')">겨울</button>
        </div>

        <button onclick="suggestTemperatureControl()">온도 제안</button>
        <div id="tempSuggestion" class="result"></div>
    </div>
</div>

<div class="container">
    <div class="section bottom">
        <h2>물 사용 분석</h2>
        <label for="shower">샤워 시간 (분):</label>
        <input type="number" id="shower" placeholder="샤워 시간 입력">

        <label for="wash">설거지 시간 (분):</label>
        <input type="number" id="wash" placeholder="설거지 시간 입력">

        <button onclick="analyzeWaterUsage()">물 사용 분석</button>
        <div id="waterResult" class="result"></div>
    </div>
</div>

<!-- 에어컨/히터 사용 제안 밑에 엑셀 저장 버튼 추가 -->
<div style="text-align: center; margin-top: 20px;">
    <button onclick="downloadExcel()">보고서 엑셀로 저장</button>
</div>

<script>
    let temperatureUnit = 'C';
    let selectedSeason = '';
    let usageFeedback = ['', '', '', '', '', ''];
    let temperatureFeedback = '';
    let waterFeedback = ['', ''];

    function setUnit(unit) {
        temperatureUnit = unit;
        alert(`온도 단위가 ${unit}로 설정되었습니다.`);
    }

    function setSeason(season) {
        selectedSeason = season;
        alert(`현재 계절이 ${season}로 설정되었습니다.`);
    }

    // 가전제품 사용 분석
    function analyzeUsage() {
        const airconTime = parseFloat(document.getElementById('aircon').value) || 0;
        const fanTime = parseFloat(document.getElementById('fan').value) || 0;
        const laptopTime = parseFloat(document.getElementById('laptop').value) || 0;
        const phoneTime = parseFloat(document.getElementById('phone').value) || 0;
        const coffeePotTime = parseFloat(document.getElementById('coffeePot').value) || 0;
        const hairDryerTime = parseFloat(document.getElementById('hairDryer').value) || 0;

        const usageResult = document.getElementById('usageResult');
        usageResult.innerHTML = '';
        usageFeedback = ['', '', '', '', '', ''];  // 피드백을 초기화

        if (airconTime > 5) {
            const msg = '에어컨을 많이 사용합니다. 온도를 조금 높여 에너지 절약을 고려하세요.';
            usageResult.innerHTML += msg + '<br>';
            usageFeedback[0] = msg;
        }
        if (fanTime > 5) {
            const msg = '선풍기를 오래 사용 중입니다. 필요할 때만 사용해보세요.';
            usageResult.innerHTML += msg + '<br>';
            usageFeedback[1] = msg;
        }
        if (laptopTime > 1) {
            const msg = '노트북 충전 시간이 길어요. 배터리 충전 후 충전기를 빼는 것이 좋습니다.';
            usageResult.innerHTML += msg + '<br>';
            usageFeedback[2] = msg;
        }
        if (phoneTime > 6) {
            const msg = '핸드폰 충전 시간이 길어요. 완충 후 충전기를 빼세요.';
            usageResult.innerHTML += msg + '<br>';
            usageFeedback[3] = msg;
        }
        if (coffeePotTime > 15) {
            const msg = '커피포트 사용 시간이 길어요. 사용 시간을 줄여보세요.';
            usageResult.innerHTML += msg + '<br>';
            usageFeedback[4] = msg;
        }
        if (hairDryerTime > 7) {
            const msg = '헤어드라이어 사용 시간이 길어요. 짧게 사용해 전기를 절약하세요.';
            usageResult.innerHTML += msg + '<br>';
            usageFeedback[5] = msg;
        }
    }

    // 에어컨/히터 사용 제안
    function convertTemperature(temp, unit) {
        if (unit === 'F') {
            return (temp - 32) * 5 / 9;
        } else if (unit === 'K') {
            return temp - 273.15;
        } else {
            return temp;
        }
    }

    function suggestTemperatureControl() {
        const temperature = parseFloat(document.getElementById('temperature').value);
        const tempSuggestion = document.getElementById('tempSuggestion');
        const celsiusTemp = convertTemperature(temperature, temperatureUnit);

        tempSuggestion.innerHTML = '';
        temperatureFeedback = '';  // 피드백 초기화

        if (selectedSeason === '여름' && celsiusTemp < 23) {
            const msg = '온도가 너무 낮습니다. 에어컨 설정 온도를 높여보세요.';
            tempSuggestion.innerHTML = msg + '<br>';
            temperatureFeedback = msg;
        } else if (selectedSeason === '겨울' && celsiusTemp > 25) {
            const msg = '온도가 너무 높습니다. 히터 설정 온도를 낮춰보세요.';
            tempSuggestion.innerHTML = msg + '<br>';
            temperatureFeedback = msg;
        } else {
            const msg = '현재 온도는 적절합니다.';
            tempSuggestion.innerHTML = msg;
            temperatureFeedback = msg;
        }
    }

    // 물 사용 분석
    const avgWaterUsage = {
        shower: 15,
        wash: 15
    };

    function analyzeWaterUsage() {
        const showerTime = parseFloat(document.getElementById('shower').value) || 0;
        const washTime = parseFloat(document.getElementById('wash').value) || 0;

        const waterResult = document.getElementById('waterResult');
        waterResult.innerHTML = '';
        waterFeedback = ['', ''];  // 피드백 초기화

        if (showerTime > avgWaterUsage.shower) {
            const msg = '샤워 시간이 길어요. 물 절약을 위해 시간을 줄여보세요.';
            waterResult.innerHTML += msg + '<br>';
            waterFeedback[0] = msg;
        } else {
            const msg = '샤워 시간이 적정합니다.';
            waterResult.innerHTML += msg + '<br>';
            waterFeedback[0] = msg;
        }

        if (washTime > avgWaterUsage.wash) {
            const msg = '설거지 시간이 길어요. 물을 절약할 방법을 고려하세요.';
            waterResult.innerHTML += msg + '<br>';
            waterFeedback[1] = msg;
        } else {
            const msg = '설거지 시간이 적정합니다.';
            waterResult.innerHTML += msg + '<br>';
            waterFeedback[1] = msg;
        }
    }

    // 엑셀로 보고서 저장
    function downloadExcel() {
        const data = [
            ["항목", "값", "피드백"],
            ["에어컨 사용 시간 (시간)", document.getElementById('aircon').value || 0, usageFeedback[0]],
            ["선풍기 사용 시간 (시간)", document.getElementById('fan').value || 0, usageFeedback[1]],
            ["노트북 충전 시간 (시간)", document.getElementById('laptop').value || 0, usageFeedback[2]],
            ["핸드폰 충전 시간 (시간)", document.getElementById('phone').value || 0, usageFeedback[3]],
            ["커피포트 사용 시간 (분)", document.getElementById('coffeePot').value || 0, usageFeedback[4]],
            ["헤어드라이어 사용 시간 (분)", document.getElementById('hairDryer').value || 0, usageFeedback[5]],
            ["실내 온도 입력", document.getElementById('temperature').value || 0, temperatureFeedback],
            ["온도 단위", temperatureUnit, ""],
            ["계절", selectedSeason, ""],
            ["샤워 시간 (분)", document.getElementById('shower').value || 0, waterFeedback[0]],
            ["설거지 시간 (분)", document.getElementById('wash').value || 0, waterFeedback[1]]
        ];

        const worksheet = XLSX.utils.aoa_to_sheet(data);
        const workbook = XLSX.utils.book_new();
        XLSX.utils.book_append_sheet(workbook, worksheet, "보고서");

        XLSX.writeFile(workbook, "에너지_물_절약_보고서.xlsx");
    }
</script>

</body>
</html>
