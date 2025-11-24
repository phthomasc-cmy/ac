<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AC Mains Simulator (3 Axes)</title>
    <!-- Load Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Load KaTeX for Math -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.css">
    <script src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.js"></script>
    
    <style>
        :root {
            --primary-blue: #2980b9;
            --accent-orange: #e67e22;
            --power-green: #27ae60;
            --bg-color: #f8f9fa;
            --panel-bg: #ffffff;
        }

        body {
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background-color: var(--bg-color);
            color: #333;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        header {
            text-align: center;
            margin-bottom: 25px;
            border-bottom: 3px solid var(--accent-orange);
            padding-bottom: 10px;
            width: 100%;
            max-width: 1100px;
        }

        h1 { margin: 0; color: var(--primary-blue); }
        .subtitle { color: #666; margin-top: 5px; }

        .main-layout {
            display: grid;
            grid-template-columns: 320px 1fr;
            gap: 25px;
            width: 100%;
            max-width: 1200px;
        }

        .controls-panel {
            background: var(--panel-bg);
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.05);
            border-left: 5px solid var(--primary-blue);
        }

        .control-group { margin-bottom: 20px; }
        
        label { 
            display: flex; 
            justify-content: space-between; 
            font-weight: 600; 
            margin-bottom: 8px;
            font-size: 0.95em;
        }

        input[type=range] { width: 100%; cursor: pointer; }
        select { width: 100%; padding: 8px; border-radius: 4px; border: 1px solid #ddd; }

        .math-display { font-size: 1.1em; }

        .stats-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-top: 20px;
            background: #f1f4f6;
            padding: 10px;
            border-radius: 6px;
        }
        .stat-item { text-align: center; }
        .stat-val { font-weight: bold; font-size: 1.1em; display: block; }
        .stat-desc { font-size: 0.75em; color: #666; }

        .chart-wrapper {
            background: var(--panel-bg);
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.05);
            height: 500px;
            position: relative;
        }

        .textbook-ref {
            grid-column: 1 / -1;
            background: #fff8e1;
            border: 1px solid #ffe0b2;
            border-radius: 8px;
            padding: 20px;
            margin-top: 20px;
        }
        .textbook-ref h3 { margin-top: 0; color: #d35400; display: flex; align-items: center; gap: 10px; }
        .ref-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 30px; }
        
        .formula-box {
            background: white;
            padding: 10px;
            border-left: 3px solid var(--accent-orange);
            margin: 10px 0;
            font-size: 0.9em;
        }

        @media (max-width: 900px) {
            .main-layout { grid-template-columns: 1fr; }
            .ref-grid { grid-template-columns: 1fr; }
            .chart-wrapper { height: 350px; }
        }
    </style>
</head>
<body>

    <header>
        <h1>22.4 Mains AC Simulator</h1>
        <div class="subtitle">Visualizing Sinusoidal Voltage, Current, and RMS Power</div>
    </header>

    <div class="main-layout">
        
        <aside class="controls-panel">
            <div class="control-group">
                <label>Waveform Type</label>
                <select id="wave-type">
                    <option value="sine">Sinusoidal (Mains)</option>
                    <option value="square">Square Wave (Example 22.3)</option>
                </select>
            </div>

            <div class="control-group">
                <label>
                    <span>Peak Voltage (<span class="math-display">V_0</span>)</span>
                    <span id="val-v0" style="color:var(--primary-blue)">10 V</span>
                </label>
                <input type="range" id="slider-v0" min="1" max="20" step="1" value="10">
            </div>

            <div class="control-group">
                <label>
                    <span>Load Resistance (<span class="math-display">R</span>)</span>
                    <span id="val-r" style="color:var(--accent-orange)">5 Ω</span>
                </label>
                <input type="range" id="slider-r" min="1" max="20" step="1" value="5">
                <small style="color:#666">Current: <span class="math-display">I_0 = V_0/R</span></small>
            </div>

            <div class="control-group">
                <label>
                    <span>Frequency (<span class="math-display">f</span>)</span>
                    <span id="val-f">1 Hz</span>
                </label>
                <input type="range" id="slider-f" min="0.5" max="3" step="0.5" value="1">
            </div>

            <div class="stats-grid">
                <div class="stat-item">
                    <span class="stat-val" id="calc-irms">1.41 A</span>
                    <span class="stat-desc">RMS Current (<span class="math-display">I_{rms}</span>)</span>
                </div>
                <div class="stat-item">
                    <span class="stat-val" id="calc-vrms">7.07 V</span>
                    <span class="stat-desc">RMS Voltage (<span class="math-display">V_{rms}</span>)</span>
                </div>
                <div class="stat-item" style="grid-column: span 2; margin-top: 5px; background: #e8f5e9; padding: 5px; border-radius: 4px;">
                    <span class="stat-val" style="color: var(--power-green)" id="calc-pavg">10.00 W</span>
                    <span class="stat-desc">Average Power <span class="math-display">\langle P \rangle</span></span>
                </div>
            </div>

            <div style="margin-top: 15px; font-size: 0.85em; color: #555;">
                <input type="checkbox" id="check-dc"> <label for="check-dc" style="display:inline;">Show Equivalent DC</label>
            </div>
        </aside>

        <main class="chart-wrapper">
            <canvas id="mainChart"></canvas>
        </main>

        <section class="textbook-ref">
            <h3>📖 Textbook Connections (Chapter 22.4)</h3>
            <div class="ref-grid">
                <div>
                    <h4>A. Sinusoidal Voltage & Current</h4>
                    <p>Voltage varies as <span class="math-display">V = V_0 \sin \omega t</span>.</p>
                    <p><strong>Note on Axes:</strong> 
                        <br><span style="color:var(--primary-blue)">Blue Axis (Left):</span> Voltage (Centered at 0).
                        <br><span style="color:var(--accent-orange)">Orange Axis (Right):</span> Current (Centered at 0).
                    </p>
                </div>
                <div>
                    <h4>B. Power Graph (Green)</h4>
                    <p><strong>Look at the Green Axis (Far Right):</strong> It starts from <strong>0</strong> and goes up. This shows that for a resistor, power is always positive (energy is consumed).</p>
                    <div class="formula-box">
                        <span class="math-display">P_{peak} = I_0^2 R</span>
                        <br>
                        <span class="math-display">\langle P \rangle = \frac{1}{2} P_{peak}</span>
                    </div>
                </div>
            </div>
        </section>

    </div>

    <script>
        // --- 1. RENDER MATH ---
        document.addEventListener("DOMContentLoaded", function() {
            document.querySelectorAll('.math-display').forEach(function(element) {
                katex.render(element.textContent, element, { throwOnError: false });
            });
        });

        // --- 2. CHART CONFIGURATION ---
        const ctx = document.getElementById('mainChart').getContext('2d');
        
        const chart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: [],
                datasets: [
                    {
                        label: 'Voltage (V)',
                        borderColor: '#2980b9',
                        borderWidth: 2,
                        data: [],
                        yAxisID: 'y',
                        tension: 0.4,
                        pointRadius: 0
                    },
                    {
                        label: 'Current (I)',
                        borderColor: '#e67e22',
                        borderWidth: 2,
                        borderDash: [5, 5],
                        data: [],
                        yAxisID: 'y1',
                        tension: 0.4,
                        pointRadius: 0
                    },
                    {
                        label: 'Inst. Power (P)',
                        borderColor: '#27ae60',
                        backgroundColor: 'rgba(39, 174, 96, 0.4)',
                        borderWidth: 1,
                        fill: true,
                        data: [],
                        yAxisID: 'y2', // NEW POWER AXIS
                        tension: 0.4,
                        pointRadius: 0
                    },
                    {
                        label: 'Avg Power',
                        borderColor: '#1e8449',
                        borderWidth: 3,
                        borderDash: [10, 5],
                        data: [],
                        yAxisID: 'y2', // Mapped to Power Axis
                        pointRadius: 0
                    },
                    {
                        label: 'DC Equiv',
                        borderColor: '#2980b9',
                        borderWidth: 1,
                        borderDash: [2, 2],
                        data: [],
                        yAxisID: 'y',
                        pointRadius: 0,
                        hidden: true
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                interaction: { mode: 'index', intersect: false },
                scales: {
                    x: {
                        type: 'linear',
                        title: { display: true, text: 'Time (s)' },
                        ticks: { callback: (val) => val.toFixed(2) }
                    },
                    // AXIS 1: VOLTAGE (Left)
                    y: {
                        type: 'linear',
                        display: true,
                        position: 'left',
                        title: { display: true, text: 'Voltage (V)', color: '#2980b9', font: {weight: 'bold'} }
                    },
                    // AXIS 2: CURRENT (Right - Inner)
                    y1: {
                        type: 'linear',
                        display: true,
                        position: 'right',
                        title: { display: true, text: 'Current (A)', color: '#e67e22', font: {weight: 'bold'} },
                        grid: { drawOnChartArea: false } // Prevent grid clutter
                    },
                    // AXIS 3: POWER (Right - Outer)
                    y2: {
                        type: 'linear',
                        display: true,
                        position: 'right',
                        min: 0, // FORCES START FROM 0
                        title: { display: true, text: 'Power (W)', color: '#27ae60', font: {weight: 'bold'} },
                        grid: { drawOnChartArea: false },
                        ticks: { color: '#27ae60' }
                    }
                }
            }
        });

        // --- 3. SIMULATION LOGIC ---
        const ui = {
            v0: document.getElementById('slider-v0'),
            r: document.getElementById('slider-r'),
            f: document.getElementById('slider-f'),
            type: document.getElementById('wave-type'),
            dcCheck: document.getElementById('check-dc'),
            dispV0: document.getElementById('val-v0'),
            dispR: document.getElementById('val-r'),
            dispF: document.getElementById('val-f'),
            dispIrms: document.getElementById('calc-irms'),
            dispVrms: document.getElementById('calc-vrms'),
            dispPavg: document.getElementById('calc-pavg')
        };

        function updateSimulation() {
            const V0 = parseFloat(ui.v0.value);
            const R = parseFloat(ui.r.value);
            const f = parseFloat(ui.f.value);
            const isSine = ui.type.value === 'sine';
            const showDC = ui.dcCheck.checked;

            const I0 = V0 / R;
            const omega = 2 * Math.PI * f;
            
            let Vrms, Irms;
            if (isSine) {
                Vrms = V0 / Math.sqrt(2);
                Irms = I0 / Math.sqrt(2);
            } else {
                Vrms = V0; 
                Irms = I0;
            }
            
            const Pavg = Vrms * Irms;
            const PeakPower = (V0 * V0) / R;

            ui.dispV0.innerText = V0 + " V";
            ui.dispR.innerText = R + " Ω";
            ui.dispF.innerText = f + " Hz";
            ui.dispVrms.innerText = Vrms.toFixed(2) + " V";
            ui.dispIrms.innerText = Irms.toFixed(2) + " A";
            ui.dispPavg.innerText = Pavg.toFixed(2) + " W";

            const points = 200;
            const duration = 2;
            const labels = [];
            const vData = [];
            const iData = [];
            const pData = [];
            const avgPData = [];
            const dcVData = [];

            for (let i = 0; i <= points; i++) {
                const t = (i / points) * duration;
                let v, current;

                if (isSine) {
                    v = V0 * Math.sin(omega * t);
                } else {
                    v = Math.sin(omega * t) >= 0 ? V0 : -V0;
                }

                current = v / R;
                const p = v * current;

                labels.push(t);
                vData.push(v);
                iData.push(current);
                pData.push(p);
                avgPData.push(Pavg);
                if(showDC) dcVData.push(Vrms);
            }

            chart.data.labels = labels;
            
            // Updates data
            chart.data.datasets[0].data = vData;
            chart.data.datasets[0].tension = isSine ? 0.4 : 0;
            
            chart.data.datasets[1].data = iData;
            chart.data.datasets[1].tension = isSine ? 0.4 : 0;
            
            chart.data.datasets[2].data = pData;
            chart.data.datasets[2].tension = isSine ? 0.4 : 0;
            
            chart.data.datasets[3].data = avgPData;
            
            chart.data.datasets[4].data = dcVData;
            chart.data.datasets[4].hidden = !showDC;

            // --- AXIS SCALING ---
            // Voltage (Left): Symmetric
            chart.options.scales.y.min = -21;
            chart.options.scales.y.max = 21;
            
            // Power (Right Green): Starts at 0, Dynamic Max
            chart.options.scales.y2.min = 0;
            chart.options.scales.y2.max = Math.max(PeakPower * 1.2, 1); 

            chart.update();
        }

        ui.v0.addEventListener('input', updateSimulation);
        ui.r.addEventListener('input', updateSimulation);
        ui.f.addEventListener('input', updateSimulation);
        ui.type.addEventListener('change', updateSimulation);
        ui.dcCheck.addEventListener('change', updateSimulation);

        updateSimulation();
    </script>
</body>
</html>
