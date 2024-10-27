# missedpay
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Enhanced SIP Calculator</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.7.0/chart.min.js"></script>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: Arial, sans-serif;
        }

        body {
            padding: 20px;
            background-color: #f5f5f5;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            background-color: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }

        h1 {
            color: #333;
            margin-bottom: 20px;
            text-align: center;
        }

        .input-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 20px;
        }

        .input-group {
            display: flex;
            flex-direction: column;
        }

        label {
            margin-bottom: 5px;
            color: #666;
        }

        input {
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 14px;
        }

        button {
            background-color: #2196F3;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
            margin: 20px 0;
        }

        button:hover {
            background-color: #1976D2;
        }

        .back-button {
            background-color: #666;
            color: white;
            border: none;
            padding: 8px 16px;
            border-radius: 4px;
            cursor: pointer;
            margin-bottom: 10px;
            display: none;
        }

        .back-button:hover {
            background-color: #555;
        }

        .chart-header {
            display: flex;
            align-items: center;
            margin-bottom: 20px;
            gap: 10px;
        }

        .chart-title {
            flex-grow: 1;
            text-align: center;
            font-size: 18px;
            color: #333;
        }

        .summary-cards {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }

        .summary-card {
            background-color: #f8f9fa;
            padding: 15px;
            border-radius: 8px;
            text-align: center;
        }

        .summary-card h3 {
            color: #666;
            font-size: 14px;
            margin-bottom: 10px;
        }

        .summary-card p {
            color: #333;
            font-size: 18px;
            font-weight: bold;
        }

        .summary-card .wealth-info {
            font-size: 14px;
            color: #666;
            margin-top: 5px;
        }

        .chart-container {
            height: 400px;
            margin-bottom: 20px;
        }

        .breakdown-table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }

        .breakdown-table th,
        .breakdown-table td {
            padding: 10px;
            border: 1px solid #ddd;
            text-align: right;
        }

        .breakdown-table th {
            background-color: #f5f5f5;
            text-align: center;
        }

        .payment-status {
            text-align: center !important;
        }

        .payment-status input {
            margin: 0;
        }

        .missed-payment {
            background-color: #ffe6e6;
        }

        .potential-value {
            color: #666;
            font-size: 0.9em;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Enhanced SIP Calculator</h1>
        
        <div class="input-grid">
            <div class="input-group">
                <label>Monthly SIP Amount (₹):</label>
                <input type="number" id="monthlyAmount" value="10000">
            </div>
            <div class="input-group">
                <label>Expected Annual Return (%):</label>
                <input type="number" id="expectedReturn" value="12">
            </div>
            <div class="input-group">
                <label>Investment Period (Years):</label>
                <input type="number" id="investmentPeriod" value="10">
            </div>
            <div class="input-group">
                <label>Annual Top-up Rate (%):</label>
                <input type="number" id="topupRate" value="10">
            </div>
        </div>

        <button onclick="calculateSIP()">Calculate Investment</button>

        <div class="summary-cards">
            <div class="summary-card">
                <h3>Total Investment</h3>
                <p id="totalInvestment">₹0</p>
            </div>
            <div class="summary-card">
                <h3>Regular SIP Value</h3>
                <p id="regularSIPTotal">₹0</p>
                <div id="regularWealthGenerated" class="wealth-info">Wealth Generated: ₹0</div>
            </div>
            <div class="summary-card">
                <h3>Top-up SIP Value</h3>
                <p id="topupSIPTotal">₹0</p>
                <div id="topupWealthGenerated" class="wealth-info">Wealth Generated: ₹0</div>
            </div>
            <div class="summary-card">
                <h3> Additional Wealth Generated</h3>
                <p id="additionalWealth">₹0</p>
            </div>
        </div>

        <div class="results">
            <div class="chart-header">
                <button id="backButton" class="back-button">← Back to Yearly View</button>
                <div id="chartTitle" class="chart-title">Yearly Overview</div>
            </div>
            <div class="chart-container">
                <canvas id="sipChart"></canvas>
            </div>
            <div id="breakdownTable"></div>
        </div>
    </div>

    <script>
        let sipChart = null;
        let currentView = 'yearly';
        let selectedYear = null;
        let calculationResults = null;

        function calculateSIP() {
            const monthlyAmount = parseFloat(document.getElementById('monthlyAmount').value);
            const expectedReturn = parseFloat(document.getElementById('expectedReturn').value);
            const investmentPeriod = parseInt(document.getElementById('investmentPeriod').value);
            const topupRate = parseFloat(document.getElementById('topupRate').value);

            const monthlyRate = expectedReturn / 12 / 100;
            const yearlyData = [];
            const monthlyBreakdown = {};

            let currentMonthlyAmount = monthlyAmount;
            let regularSIPTotal = 0;
            let topupSIPTotal = 0;
            let totalInvestment = 0;
            let regularInvestment = 0;
            let topupInvestment = 0;

            for (let year = 1; year <= investmentPeriod; year++) {
                let yearlyRegularInvestment = 0;
                let yearlyTopupInvestment = 0;
                monthlyBreakdown[year] = [];

                for (let month = 1; month <= 12; month++) {
                    yearlyRegularInvestment += monthlyAmount;
                    yearlyTopupInvestment += currentMonthlyAmount;

                    regularInvestment += monthlyAmount;
                    topupInvestment += currentMonthlyAmount;

                    regularSIPTotal = (regularSIPTotal + monthlyAmount) * (1 + monthlyRate);
                    topupSIPTotal = (topupSIPTotal + currentMonthlyAmount) * (1 + monthlyRate);

                    monthlyBreakdown[year].push({
                        period: `Month ${month}`,
                        regularSIP: {
                            investment: monthlyAmount,
                            value: regularSIPTotal
                        },
                        topupSIP: {
                            investment: currentMonthlyAmount,
                            value: topupSIPTotal
                        },
                        paymentStatus: true, // Default to paid
                        potentialValue: topupSIPTotal // Store the potential value if all payments were made
                    });
                }

                totalInvestment += yearlyRegularInvestment;

                yearlyData.push({
                    period: `Year ${year}`,
                    regularSIP: {
                        investment: yearlyRegularInvestment,
                        value: regularSIPTotal
                    },
                    topupSIP: {
                        investment: yearlyTopupInvestment,
                        value: topupSIPTotal
                    }
                });

                // Apply annual top-up
                currentMonthlyAmount *= (1 + topupRate / 100);
            }

            calculationResults = {
                yearly: yearlyData,
                monthly: monthlyBreakdown,
                summary: {
                    totalInvestment: totalInvestment,
                    regularSIPTotal: regularSIPTotal,
                    topupSIPTotal: topupSIPTotal,
                    regularInvestment: regularInvestment,
                    topupInvestment: topupInvestment
                }
            };

            updateUI();
        }

        function updateUI() {
            const regularWealthGenerated = calculationResults.summary.regularSIPTotal - calculationResults.summary.regularInvestment;
            const topupWealthGenerated = calculationResults.summary.topupSIPTotal - calculationResults.summary.topupInvestment;

            document.getElementById('totalInvestment').textContent = 
                `₹${calculationResults.summary.totalInvestment.toLocaleString('en-IN', {maximumFractionDigits: 0})}`;
            document.getElementById('regularSIPTotal').textContent = 
                `₹${calculationResults.summary.regularSIPTotal.toLocaleString('en-IN', {maximumFractionDigits: 0})}`;
            document.getElementById('regularWealthGenerated').textContent = 
                `Wealth Generated: ₹${regularWealthGenerated.toLocaleString('en-IN', {maximumFractionDigits: 0})}`;
            document.getElementById('topupSIPTotal').textContent = 
                `₹${calculationResults.summary.topupSIPTotal.toLocaleString('en-IN', {maximumFractionDigits: 0})}`;
            document.getElementById('topupWealthGenerated').textContent = 
                `Wealth Generated: ₹${topupWealthGenerated.toLocaleString('en-IN', {maximumFractionDigits: 0})}`;
            document.getElementById('additionalWealth').textContent = 
                `₹${(calculationResults.summary.topupSIPTotal - calculationResults.summary.regularSIPTotal).toLocaleString('en-IN', {maximumFractionDigits: 0})}`;

            showYearlyView();
        }

        function showYearlyView() {
            currentView = 'yearly';
            selectedYear = null;
            document.getElementById('backButton').style.display = 'none';
            document.getElementById('chartTitle').textContent = 'Yearly Overview';
            updateChart(calculationResults.yearly);
            updateTable(calculationResults.yearly, false);
        }

        function showMonthlyView(year) {
            currentView = 'monthly';
            selectedYear = year;
            document.getElementById('backButton').style.display = 'block';
            document.getElementById('chartTitle').textContent = `Monthly Breakdown - Year ${year}`;
            updateChart(calculationResults.monthly[year]);
            updateTable(calculationResults.monthly[year], true);
        }

        function updatePaymentStatus(year, monthIndex, checked) {
            const monthData = calculationResults.monthly[year][monthIndex];
            monthData.paymentStatus = checked;

            // Recalculate values from this point forward
            const monthlyRate = parseFloat(document.getElementById('expectedReturn').value) / 12 / 100;
            let regularSIPTotal = monthIndex > 0 ? calculationResults.monthly[year][monthIndex - 1].regularSIP.value : 0;
            let topupSIPTotal = monthIndex > 0 ? calculationResults.monthly[year][monthIndex - 1].topupSIP.value : 0;

            for (let i = monthIndex; i < calculationResults.monthly[year].length; i++) {
                const currentMonth = calculationResults.monthly[year][i];
                
                if (currentMonth.paymentStatus) {
                    regularSIPTotal = (regularSIPTotal + currentMonth.regularSIP.investment) * (1 + monthlyRate);
                    topupSIPTotal = (topupSIPTotal + currentMonth.topupSIP.investment) * (1 + monthlyRate);
                } else {
                    regularSIPTotal = regularSIPTotal * (1 + monthlyRate);
                    topupSIPTotal = topupSIPTotal * (1 + monthlyRate);
                }

                currentMonth.regularSIP.value = regularSIPTotal;
                currentMonth.topupSIP.value = topupSIPTotal;
            }

            // Update the display
            updateChart(calculationResults.monthly[year]);
            updateTable(calculationResults.monthly[year], true);
        }

        function updateChart(data) {
            const ctx = document.getElementById('sipChart').getContext('2d');

            if (sipChart) {
                sipChart.destroy();
            }

            sipChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: data.map(row => row.period),
                    datasets: [
                        {
                            label: 'Regular SIP',
                            data: data.map(row => row.regularSIP.value),
                            backgroundColor: 'blue',
                            borderColor: 'blue',
                            borderWidth: 1
                        },
                        {
                            label: 'Top-up SIP',
                            data: data.map(row => row.topupSIP.value),
                            backgroundColor: 'green',
                            borderColor: 'green',
                            borderWidth: 1
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    onClick: (event, elements) => {
                        if (currentView === 'yearly' && elements.length > 0) {
                            const index = elements[0].index;
                            showMonthlyView(index + 1);
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: {
                                callback: value => '₹' + value.toLocaleString('en-IN', {maximumFractionDigits: 0})
                            }
                        }
                    },
                    plugins: {
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    const value = context.raw;
                                    return `${context.dataset.label}: ₹${value.toLocaleString('en-IN', {maximumFractionDigits: 0})}`;
                                }
                            }
                        }
                    }
                }
            });
        }

        function updateTable(data, isMonthly) {
            let tableHTML = `
                <table class="breakdown-table">
                    <thead>
                        <tr>
                            <th>Period</th>
                            ${isMonthly ? '<th>Payment Status</th>' : ''}
                            <th>Regular SIP Investment</th>
                            <th>Regular SIP Value</th>
                            <th>Top-up SIP Investment</th>
                            <th>Top-up SIP Value</th>
                            <th>Additional Wealth</th>
                            ${isMonthly ? '<th>Potential Value</th>' : ''}
                        </tr>
                    </thead>
                    <tbody>
            `;

            data.forEach((row, index) => {
                const additionalWealth = row.topupSIP.value - row.regularSIP.value;
                const isMissed = isMonthly && !row.paymentStatus;
                const potentialDifference = isMonthly && isMissed ? 
                    `(₹${(row.potentialValue - row.topupSIP.value).toLocaleString('en-IN', {maximumFractionDigits: 0})} less)` : '';

                tableHTML += `
                    <tr class="${isMissed ? 'missed-payment' : ''}">
                        <td style="text-align: center">${row.period}</td>
                        ${isMonthly ? `
                            <td class="payment-status">
                                <input type="checkbox" 
                                    ${row.paymentStatus ? 'checked' : ''} 
                                    onchange="updatePaymentStatus(${selectedYear}, ${index}, this.checked)"
                                />
                            </td>
                        ` : ''}
                        <td>₹${row.regularSIP.investment.toLocaleString('en-IN', {maximumFractionDigits: 0})}</td>
                        <td>₹${row.regularSIP.value.toLocaleString('en-IN', {maximumFractionDigits: 0})}</td>
                        <td>₹${row.topupSIP.investment.toLocaleString('en-IN', {maximumFractionDigits: 0})}</td>
                        <td>₹${row.topupSIP.value.toLocaleString('en-IN', {maximumFractionDigits: 0})}</td>
                        <td>₹${additionalWealth.toLocaleString('en-IN', {maximumFractionDigits: 0})}</td>
                        ${isMonthly ? `
                            <td class="potential-value">
                                ₹${row.potentialValue.toLocaleString('en-IN', {maximumFractionDigits: 0})}
                                ${potentialDifference}
                            </td>
                        ` : ''}
                    </tr>
                `;
            });

            tableHTML += '</tbody></table>';
            document.getElementById('breakdownTable').innerHTML = tableHTML;
        }

        document.getElementById('backButton').addEventListener('click', showYearlyView);

        // Initial calculation
        calculateSIP();
    </script>
</body>
</html>
