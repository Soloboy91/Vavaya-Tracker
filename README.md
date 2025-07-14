<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>90-Day Calorie and Weight Tracker</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        #tracker { max-width: 800px; margin: auto; }
        input { margin: 5px; padding: 5px; }
        button { padding: 10px; background: #4CAF50; color: white; border: none; cursor: pointer; }
        button:hover { background: #45a049; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background: #f2f2f2; }
        #weightChart, #calorieChart { margin-top: 20px; }
    </style>
</head>
<body>
    <div id="tracker">
        <h1>90-Day Calorie and Weight Tracker</h1>
        <p>Your daily calorie target: 1990 cal. Track your daily calorie intake and weight. Data is saved locally in your browser.</p>
        
        <label for="date">Date:</label>
        <input type="date" id="date" required>
        
        <label for="calories">Calories Consumed:</label>
        <input type="number" id="calories" min="0" required>
        
        <label for="weight">Weight (in lbs or kg):</label>
        <input type="number" id="weight" min="0" step="0.1" required>
        
        <button onclick="addEntry()">Add Entry</button>
        
        <table id="entriesTable">
            <thead>
                <tr>
                    <th>Date</th>
                    <th>Calories</th>
                    <th>Over/Under Target</th>
                    <th>Weight</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody></tbody>
        </table>
        
        <canvas id="weightChart" width="400" height="200"></canvas>
        <canvas id="calorieChart" width="400" height="200"></canvas>
        
        <button onclick="clearData()" style="background: #f44336; margin-top: 20px;">Clear All Data</button>
    </div>

    <script>
        const TARGET_CALORIES = 1990;
        const STORAGE_KEY = 'trackerData';
        let data = loadData();
        let weightChart, calorieChart;

        function loadData() {
            const stored = localStorage.getItem(STORAGE_KEY);
            return stored ? JSON.parse(stored) : [];
        }

        function saveData() {
            localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
        }

        function addEntry() {
            const date = document.getElementById('date').value;
            const calories = parseInt(document.getElementById('calories').value);
            const weight = parseFloat(document.getElementById('weight').value);

            if (!date || isNaN(calories) || isNaN(weight)) {
                alert('Please fill in all fields correctly.');
                return;
            }

            // Check if entry for this date already exists
            const existingIndex = data.findIndex(entry => entry.date === date);
            if (existingIndex !== -1) {
                data[existingIndex] = { date, calories, weight };
            } else {
                data.push({ date, calories, weight });
            }

            data.sort((a, b) => new Date(a.date) - new Date(b.date)); // Sort by date
            saveData();
            renderTable();
            updateCharts();
        }

        function deleteEntry(index) {
            data.splice(index, 1);
            saveData();
            renderTable();
            updateCharts();
        }

        function renderTable() {
            const tbody = document.querySelector('#entriesTable tbody');
            tbody.innerHTML = '';
            data.forEach((entry, index) => {
                const diff = entry.calories - TARGET_CALORIES;
                const diffText = diff > 0 ? `+${diff}` : diff;
                const row = `
                    <tr>
                        <td>${entry.date}</td>
                        <td>${entry.calories}</td>
                        <td>${diffText}</td>
                        <td>${entry.weight}</td>
                        <td><button onclick="deleteEntry(${index})" style="background: #f44336; color: white; border: none; cursor: pointer;">Delete</button></td>
                    </tr>
                `;
                tbody.innerHTML += row;
            });
        }

        function updateCharts() {
            const labels = data.map(entry => entry.date);
            const weights = data.map(entry => entry.weight);
            const calories = data.map(entry => entry.calories);

            // Weight Line Chart
            if (weightChart) weightChart.destroy();
            weightChart = new Chart(document.getElementById('weightChart'), {
                type: 'line',
                data: {
                    labels: labels,
                    datasets: [{
                        label: 'Weight Over Time',
                        data: weights,
                        borderColor: 'blue',
                        fill: false
                    }]
                },
                options: {
                    scales: { y: { beginAtZero: false } },
                    plugins: { title: { display: true, text: 'Weight Trend (90 Days)' } }
                }
            });

            // Calorie Bar Chart
            if (calorieChart) calorieChart.destroy();
            calorieChart = new Chart(document.getElementById('calorieChart'), {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [{
                        label: 'Calories Consumed',
                        data: calories,
                        backgroundColor: calories.map(c => c > TARGET_CALORIES ? 'red' : 'green')
                    }, {
                        label: 'Target',
                        data: labels.map(() => TARGET_CALORIES),
                        type: 'line',
                        borderColor: 'orange',
                        fill: false
                    }]
                },
                options: {
                    scales: { y: { beginAtZero: true } },
                    plugins: { title: { display: true, text: 'Calories vs Target' } }
                }
            });
        }

        function clearData() {
            if (confirm('Are you sure you want to clear all data?')) {
                data = [];
                saveData();
                renderTable();
                updateCharts();
            }
        }

        // Initial render
        const today = new Date().toISOString().split('T')[0];
        document.getElementById('date').value = today;
        renderTable();
        updateCharts();
    </script>
</body>
</html>
