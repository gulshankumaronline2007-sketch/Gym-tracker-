<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ultimate Fitness Tracker</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: 'Segoe UI', sans-serif; background: #0f0f0f; color: white; padding: 15px; margin: 0; padding-bottom: 50px; }
        .card { background: #1a1a1a; padding: 15px; border-radius: 15px; margin-bottom: 15px; border: 1px solid #333; }
        h2, h3 { color: #00adb5; margin-top: 0; text-align: center; }
        input, select { width: 100%; padding: 10px; margin: 5px 0; border-radius: 8px; border: 1px solid #333; background: #252525; color: white; box-sizing: border-box; }
        .btn { width: 100%; padding: 12px; background: #00adb5; border: none; border-radius: 8px; color: white; font-weight: bold; cursor: pointer; margin-top: 10px; }
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .stats-box { text-align: center; padding: 10px; background: #252525; border-radius: 10px; border-left: 4px solid #00adb5; }
        #timerDisplay { font-size: 30px; font-weight: bold; color: #ff2e63; text-align: center; margin: 10px 0; }
        .water-btn { background: #3498db; width: 50px; height: 50px; border-radius: 50%; font-size: 20px; }
    </style>
</head>
<body>

    <div class="card">
        <h2>🚀 Gym Pro Tracker</h2>
        <canvas id="myChart" style="height: 180px;"></canvas>
        <div class="grid" style="margin-top: 15px;">
            <div class="stats-box">
                <small>Calories</small>
                <div id="totalCals">0</div>
            </div>
            <div class="stats-box">
                <small>Water (Glasses)</small>
                <div id="waterCount">0</div>
            </div>
        </div>
    </div>

    <div class="card">
        <h3>⏱️ Rest Timer</h3>
        <div id="timerDisplay">01:00</div>
        <button class="btn" style="background: #ff2e63;" onclick="startTimer()">Start 60s Rest</button>
    </div>

    <div class="card">
        <h3>💪 Log Set</h3>
        <select id="muscleGroup">
            <option value="Chest">Chest</option>
            <option value="Back">Back</option>
            <option value="Legs">Legs</option>
            <option value="Arms">Arms</option>
        </select>
        <input type="text" id="exName" placeholder="Exercise Name">
        <div class="grid">
            <input type="number" id="exWeight" placeholder="Weight (kg)">
            <input type="number" id="exReps" placeholder="Reps">
        </div>
        <button class="btn" onclick="addData()">Save Workout</button>
    </div>

    <div class="card" style="text-align: center;">
        <h3>💧 Water Tracker</h3>
        <button class="water-btn" onclick="addWater()">+</button>
        <p><small>Tap to add a glass of water</small></p>
    </div>

    <button class="btn" style="background: #444; opacity: 0.5; font-size: 10px;" onclick="clearData()">Reset App Data</button>

<script>
    const ctx = document.getElementById('myChart').getContext('2d');
    let savedLabels = JSON.parse(localStorage.getItem('gymLabels')) || [];
    let savedData = JSON.parse(localStorage.getItem('gymData')) || [];
    let totalBurned = parseFloat(localStorage.getItem('burnedCals')) || 0;
    let water = parseInt(localStorage.getItem('waterLog')) || 0;

    // Initialize UI
    document.getElementById('totalCals').innerText = totalBurned.toFixed(0) + " kcal";
    document.getElementById('waterCount').innerText = water;

    const myChart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: savedLabels,
            datasets: [{
                label: 'Volume',
                data: savedData,
                borderColor: '#00adb5',
                fill: true,
                backgroundColor: 'rgba(0, 173, 181, 0.1)',
                tension: 0.4
            }]
        },
        options: { responsive: true, maintainAspectRatio: false }
    });

    function addData() {
        const name = document.getElementById('exName').value;
        const weight = parseFloat(document.getElementById('exWeight').value) || 0;
        const reps = parseInt(document.getElementById('exReps').value) || 0;
        const muscle = document.getElementById('muscleGroup').value;

        if (!name) return alert("Kaunsi exercise ki?");

        const volume = weight * reps;
        const calBurned = (6.0 * 3.5 * 70 / 200) * 3; // Standard calculation
        totalBurned += calBurned;

        const time = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
        myChart.data.labels.push(`${time} (${muscle})`);
        myChart.data.datasets[0].data.push(volume);
        myChart.update();

        saveToLocal();
        updateUI();
    }

    function addWater() {
        water++;
        updateUI();
        saveToLocal();
    }

    let timer;
    function startTimer() {
        clearInterval(timer);
        let timeLeft = 60;
        timer = setInterval(() => {
            timeLeft--;
            document.getElementById('timerDisplay').innerText = `00:${timeLeft < 10 ? '0' : ''}${timeLeft}`;
            if(timeLeft <= 0) {
                clearInterval(timer);
                alert("Rest Over! Agla set lagaiye.");
                document.getElementById('timerDisplay').innerText = "01:00";
            }
        }, 1000);
    }

    function updateUI() {
        document.getElementById('totalCals').innerText = totalBurned.toFixed(0) + " kcal";
        document.getElementById('waterCount').innerText = water;
    }

    function saveToLocal() {
        localStorage.setItem('gymLabels', JSON.stringify(myChart.data.labels));
        localStorage.setItem('gymData', JSON.stringify(myChart.data.datasets[0].data));
        localStorage.setItem('burnedCals', totalBurned);
        localStorage.setItem('waterLog', water);
    }

    function clearData() {
        if(confirm("Saara data delete karein?")) {
            localStorage.clear();
            location.reload();
        }
    }
</script>
</body>
</html>
