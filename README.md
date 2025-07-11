<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kalkulator Turunan Numerik</title>
    <!-- STYLING: Layout grid 2 kolom, responsive, tema biru -->
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: Arial, sans-serif; background: #f5f5f5; padding: 20px; }
        .container { max-width: 1200px; margin: 0 auto; background: white; border-radius: 10px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        .header { background: #2c3e50; color: white; padding: 20px; text-align: center; border-radius: 10px 10px 0 0; }
        .content { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; padding: 20px; }
        .section { padding: 20px; border: 1px solid #ddd; border-radius: 8px; }
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; }
        input, select { width: 100%; padding: 8px; border: 1px solid #ddd; border-radius: 4px; }
        button { width: 100%; padding: 10px; background: #3498db; color: white; border: none; border-radius: 4px; cursor: pointer; }
        button:hover { background: #2980b9; }
        .error { background: #e74c3c; color: white; padding: 10px; border-radius: 4px; margin: 10px 0; display: none; }
        table { width: 100%; border-collapse: collapse; margin: 15px 0; }
        th, td { padding: 8px; text-align: center; border: 1px solid #ddd; }
        th { background: #f8f9fa; }
        .results { background: #f8f9fa; padding: 15px; border-radius: 4px; margin-top: 15px; }
        @media (max-width: 768px) { .content { grid-template-columns: 1fr; } }
    </style>
</head>
<body>
    <div class="container">
        <div class="header"><h1>Kalkulator Turunan Numerik</h1></div>
        <div class="content">
             <!-- BAGIAN TABEL: Generate tabel nilai f(x) untuk perhitungan numerik -->
            <div class="section">
                <h2>📊 Tabel Nilai Fungsi f(x)</h2>
                <div class="form-group">
                    <label>Fungsi f(x):</label>
                    <input type="text" id="functionInput" placeholder="Contoh: x^2, sin(x), exp(x)">
                </div>
                <div class="form-group">
                    <label>Nilai awal (x₀):</label>
                    <input type="number" id="x0Input" step="any">
                </div>
                <div class="form-group">
                    <label>Jarak antar titik (h):</label>
                    <input type="number" id="hInput" step="any">
                </div>
                <div class="form-group">
                    <label>Jumlah baris:</label>
                    <input type="number" id="rowsInput" min="3" max="20">
                </div>
                <button onclick="generateTable()">Tampilkan Tabel</button>
                <div id="tableError" class="error"></div>
                <div id="tableContainer" style="display: none">
                    <table id="functionTable">
                        <thead><tr><th>x</th><th>f(x)</th></tr></thead>
                        <tbody></tbody>
                    </table>
                </div>
            </div>
            <!-- BAGIAN PERHITUNGAN: 4 metode numerik untuk turunan ke-1 dan ke-2 -->
            <div class="section">
                <h2>🧮 Perhitungan Turunan</h2>
                <div class="form-group">
                    <label>Nilai x yang ingin dihitung:</label>
                    <input type="number" id="xCalc" step="any">
                </div>
                <div class="form-group">
                    <label>Jenis Turunan:</label>
                    <select id="derivativeOrder">
                        <option value="1">Turunan ke-1</option>
                        <option value="2">Turunan ke-2</option>
                    </select>
                </div>
                <div class="form-group">
                    <label>Metode:</label>
                    <select id="methodSelect">
                        <option value="forward-h">Selisih Maju O(h)</option>
                        <option value="forward-h2">Selisih Maju O(h²)</option>
                        <option value="central-h2">Selisih Pusat O(h²)</option>
                        <option value="central-h4">Selisih Pusat O(h⁴)</option>
                    </select>
                </div>
                <button onclick="calculateDerivative()">Hitung</button>
                <div id="calcError" class="error"></div>
                <div id="calculationResults"></div>
            </div>
        </div>
    </div>

    <script>
        // VARIABEL GLOBAL: Menyimpan data tabel dan parameter
        let functionData = [], currentFunction = "", currentH = 0;

        // EVALUASI FUNGSI: Konversi string input ke fungsi JavaScript
        // Mendukung: x^2, sin(x), cos(x), exp(x), log(x), sqrt(x), pi, e
        function evaluateFunction(funcStr, x) {
            try {
                const processedFunc = funcStr.toLowerCase().replace(/\^/g, "**").replace(/sin/g, "Math.sin").replace(/cos/g, "Math.cos").replace(/tan/g, "Math.tan").replace(/exp/g, "Math.exp").replace(/log/g, "Math.log").replace(/sqrt/g, "Math.sqrt").replace(/abs/g, "Math.abs").replace(/pi/g, "Math.PI").replace(/e(?![a-zA-Z])/g, "Math.E");
                const func = new Function("x", `return ${processedFunc}`);
                const result = func(x);
                if (isNaN(result) || !isFinite(result)) throw new Error("Hasil tidak valid");
                return result;
            } catch (error) { throw new Error("Fungsi tidak valid"); }
        }

        // ERROR HANDLING: Tampilkan pesan error
        function showError(elementId, message) {
            const errorElement = document.getElementById(elementId);
            errorElement.textContent = message;
            errorElement.style.display = "block";
            setTimeout(() => errorElement.style.display = "none", 5000);
        }

        // GENERATE TABEL: Buat tabel nilai f(x) dari x₀ dengan interval h
        // Validasi: fungsi tidak kosong, nilai numerik valid, baris 3-20
        function generateTable() {
            const funcInput = document.getElementById("functionInput").value.trim();
            const x0 = parseFloat(document.getElementById("x0Input").value);
            const h = parseFloat(document.getElementById("hInput").value);
            const rows = parseInt(document.getElementById("rowsInput").value);

            if (!funcInput) return showError("tableError", "Masukkan fungsi terlebih dahulu.");
            if (isNaN(x0) || isNaN(h) || isNaN(rows)) return showError("tableError", "Masukkan nilai numerik yang valid.");
            if (rows < 3 || rows > 20) return showError("tableError", "Jumlah baris harus antara 3-20.");

            try {
                functionData = [];
                const tbody = document.querySelector("#functionTable tbody");
                tbody.innerHTML = "";
                for (let i = 0; i < rows; i++) {
                    const x = x0 + i * h;
                    const fx = evaluateFunction(funcInput, x);
                    functionData.push({ x: x, fx: fx });
                    const row = tbody.insertRow();
                    row.insertCell(0).textContent = x.toFixed(1);
                    row.insertCell(1).textContent = fx.toFixed(3);
                }
                currentFunction = funcInput;
                currentH = h;
                document.getElementById("tableContainer").style.display = "block";
            } catch (error) {
                showError("tableError", "Fungsi tidak valid. Periksa kembali penulisan fungsi.");
            }
        }

        // CARI NILAI: Ambil f(x) dari tabel berdasarkan x target
        function findFunctionValue(targetX) {
            const tolerance = 1e-10;
            for (let data of functionData) {
                if (Math.abs(data.x - targetX) < tolerance) return data.fx;
            }
            return null;
        }

        // DATABASE TURUNAN: Untuk analisis error bound teoritis
        function getSymbolicDerivative(funcStr, order) {
            const derivatives = {
                1: { "x^2": "2*x", "x**2": "2*x", "sin(x)": "cos(x)", "cos(x)": "-sin(x)", "exp(x)": "exp(x)", "x^3": "3*x^2", "x**3": "3*x**2", "x^4": "4*x^3", "x**4": "4*x**3" },
                2: { "x^2": "2", "x**2": "2", "sin(x)": "-sin(x)", "cos(x)": "-cos(x)", "exp(x)": "exp(x)", "x^3": "6*x", "x**3": "6*x", "x^4": "12*x^2", "x**4": "12*x**2" },
                3: { "x^2": "0", "x**2": "0", "sin(x)": "-cos(x)", "cos(x)": "sin(x)", "exp(x)": "exp(x)", "x^3": "6", "x**3": "6", "x^4": "24*x", "x**4": "24*x" },
                4: { "x^2": "0", "x**2": "0", "sin(x)": "sin(x)", "cos(x)": "cos(x)", "exp(x)": "exp(x)", "x^3": "0", "x**3": "0", "x^4": "24", "x**4": "24" },
                5: { "x^2": "0", "x**2": "0", "sin(x)": "cos(x)", "cos(x)": "-sin(x)", "exp(x)": "exp(x)", "x^3": "0", "x**3": "0", "x^4": "0", "x**4": "0" }
            };
            const normalizedFunc = funcStr.toLowerCase().replace(/\^/g, "**");
            return derivatives[order] && derivatives[order][normalizedFunc] ? derivatives[order][normalizedFunc] : null;
        }

        // TURUNAN PERTAMA: 4 metode numerik dengan analisis error
        function calculateFirstDerivative(x, method) {
            const hVal = currentH;
            let result, steps, errorAnalysis;

            switch (method) {
                case "forward-h":
                    // Selisih Maju O(h): f'(x) = (f₁ - f₀) / h
                    const f0 = findFunctionValue(x), f1 = findFunctionValue(x + hVal);
                    if (f0 === null || f1 === null) throw new Error("Nilai x tidak ditemukan dalam tabel.");
                    result = (f1 - f0) / hVal;
                    steps = `<h4>Metode: Selisih Maju O(h) - Turunan ke-1</h4><p><strong>Rumus:</strong> f'(x) = (f₁ - f₀) / h</p><p><strong>Substitusi:</strong> f'(${x}) = [f(${x + hVal}) - f(${x})] / ${hVal}</p><p><strong>Perhitungan:</strong> f'(${x}) = (${f1.toFixed(3)} - ${f0.toFixed(3)}) / ${hVal}</p><p><strong>Hasil:</strong> f'(${x}) = ${result.toFixed(3)}</p>`;
                    const derivative2 = getSymbolicDerivative(currentFunction, 2);
                    if (derivative2 && derivative2 !== "0") {
                        try {
                            const f2_value = evaluateFunction(derivative2, x);
                            const error_bound = Math.abs((hVal * f2_value) / 2);
                            errorAnalysis = `<h4>Orde Galat:</h4><p><strong>Rumus:</strong> O(h) = (h/2) × |f''(x)|</p><p><strong>Turunan simbolik:</strong> f(x) = ${currentFunction} → f''(x) = ${derivative2}</p><p><strong>Substitusi:</strong> O(h) = (${hVal}/2) × |f''(${x})|</p><p><strong>Evaluasi f''(${x}):</strong> f''(${x}) = ${f2_value.toFixed(3)}</p><p><strong>Perhitungan:</strong> O(h) = ${(hVal / 2).toFixed(3)} × ${Math.abs(f2_value).toFixed(3)} = ${error_bound.toFixed(3)}</p>`;
                        } catch (e) {
                            errorAnalysis = "<h4>Orde Galat: O(h)</h4><p>Tidak dapat menghitung bound error untuk fungsi ini.</p>";
                        }
                    } else {
                        errorAnalysis = "<h4>Orde Galat: O(h) = 0</h4><p>Untuk fungsi polinomial derajat ≤ 1, galat = 0</p>";
                    }
                    break;
                case "forward-h2":
                    // Selisih Maju O(h²): f'(x) = (-3f₀ + 4f₁ - f₂) / (2h)
                    const f0_fh2 = findFunctionValue(x), f1_fh2 = findFunctionValue(x + hVal), f2_fh2 = findFunctionValue(x + 2 * hVal);
                    if (f0_fh2 === null || f1_fh2 === null || f2_fh2 === null) throw new Error("Nilai x tidak ditemukan dalam tabel.");
                    result = (-3 * f0_fh2 + 4 * f1_fh2 - f2_fh2) / (2 * hVal);
                    steps = `<h4>Metode: Selisih Maju O(h²) - Turunan ke-1</h4><p><strong>Rumus:</strong> f'(x) = (-3f₀ + 4f₁ - f₂) / (2h)</p><p><strong>Substitusi:</strong> f'(${x}) = [-3f(${x}) + 4f(${x + hVal}) - f(${x + 2 * hVal})] / (2×${hVal})</p><p><strong>Perhitungan:</strong> f'(${x}) = (-3(${f0_fh2.toFixed(3)}) + 4(${f1_fh2.toFixed(3)}) - ${f2_fh2.toFixed(3)}) / ${2 * hVal}</p><p><strong>Hasil:</strong> f'(${x}) = ${result.toFixed(3)}</p>`;
                    const derivative3_fh2 = getSymbolicDerivative(currentFunction, 3);
                    if (derivative3_fh2 && derivative3_fh2 !== "0") {
                        try {
                            const f3_value = evaluateFunction(derivative3_fh2, x);
                            const error_bound = Math.abs((hVal * hVal * f3_value) / 3);
                            errorAnalysis = `<h4>Orde Galat:</h4><p><strong>Rumus:</strong> O(h²) = (h²/3) × |f'''(x)|</p><p><strong>Turunan simbolik:</strong> f(x) = ${currentFunction} → f'''(x) = ${derivative3_fh2}</p><p><strong>Substitusi:</strong> O(h²) = (${hVal}²/3) × |f'''(${x})|</p><p><strong>Evaluasi f'''(${x}):</strong> f'''(${x}) = ${f3_value.toFixed(3)}</p><p><strong>Perhitungan:</strong> O(h²) = ${((hVal * hVal) / 3).toFixed(3)} × ${Math.abs(f3_value).toFixed(3)} = ${error_bound.toFixed(3)}</p>`;
                        } catch (e) {
                            errorAnalysis = "<h4>Orde Galat: O(h²)</h4><p>Tidak dapat menghitung bound error untuk fungsi ini.</p>";
                        }
                    } else {
                        errorAnalysis = "<h4>Orde Galat: O(h²) = 0</h4><p>Untuk fungsi polinomial derajat ≤ 2, galat = 0</p>";
                    }
                    break;
                case "central-h2":
                    // Selisih Pusat O(h²): f'(x) = (f₁ - f₋₁) / (2h)
                    const f1_ch2 = findFunctionValue(x + hVal), f_1_ch2 = findFunctionValue(x - hVal);
                    if (f1_ch2 === null || f_1_ch2 === null) throw new Error("Nilai x tidak ditemukan dalam tabel.");
                    result = (f1_ch2 - f_1_ch2) / (2 * hVal);
                    steps = `<h4>Metode: Selisih Pusat O(h²) - Turunan ke-1</h4><p><strong>Rumus:</strong> f'(x) = (f₁ - f₋₁) / (2h)</p><p><strong>Substitusi:</strong> f'(${x}) = [f(${x + hVal}) - f(${x - hVal})] / (2×${hVal})</p><p><strong>Perhitungan:</strong> f'(${x}) = (${f1_ch2.toFixed(3)} - ${f_1_ch2.toFixed(3)}) / ${2 * hVal}</p><p><strong>Hasil:</strong> f'(${x}) = ${result.toFixed(3)}</p>`;
                    const derivative3_ch2 = getSymbolicDerivative(currentFunction, 3);
                    if (derivative3_ch2 && derivative3_ch2 !== "0") {
                        try {
                            const f3_value = evaluateFunction(derivative3_ch2, x);
                            const error_bound = Math.abs((hVal * hVal * f3_value) / 6);
                            errorAnalysis = `<h4>Orde Galat:</h4><p><strong>Rumus:</strong> O(h²) = (h²/6) × |f'''(x)|</p><p><strong>Turunan simbolik:</strong> f(x) = ${currentFunction} → f'''(x) = ${derivative3_ch2}</p><p><strong>Substitusi:</strong> O(h²) = (${hVal}²/6) × |f'''(${x})|</p><p><strong>Evaluasi f'''(${x}):</strong> f'''(${x}) = ${f3_value.toFixed(3)}</p><p><strong>Perhitungan:</strong> O(h²) = ${((hVal * hVal) / 6).toFixed(3)} × ${Math.abs(f3_value).toFixed(3)} = ${error_bound.toFixed(3)}</p>`;
                        } catch (e) {
                            errorAnalysis = "<h4>Orde Galat: O(h²)</h4><p>Tidak dapat menghitung bound error untuk fungsi ini.</p>";
                        }
                    } else {
                        errorAnalysis = "<h4>Orde Galat: O(h²) = 0</h4><p>Untuk fungsi polinomial derajat ≤ 2, galat = 0</p>";
                    }
                    break;
                case "central-h4":
                    // Selisih Pusat O(h⁴): f'(x) = (-f₂ + 8f₁ - 8f₋₁ + f₋₂) / (12h)
                    const f2_ch4 = findFunctionValue(x + 2 * hVal), f1_ch4 = findFunctionValue(x + hVal), f_1_ch4 = findFunctionValue(x - hVal), f_2_ch4 = findFunctionValue(x - 2 * hVal);
                    if (f2_ch4 === null || f1_ch4 === null || f_1_ch4 === null || f_2_ch4 === null) throw new Error("Nilai x tidak ditemukan dalam tabel.");
                    result = (-f2_ch4 + 8 * f1_ch4 - 8 * f_1_ch4 + f_2_ch4) / (12 * hVal);
                    steps = `<h4>Metode: Selisih Pusat O(h⁴) - Turunan ke-1</h4><p><strong>Rumus:</strong> f'(x) = (-f₂ + 8f₁ - 8f₋₁ + f₋₂) / (12h)</p><p><strong>Substitusi:</strong> f'(${x}) = [-f(${x + 2 * hVal}) + 8f(${x + hVal}) - 8f(${x - hVal}) + f(${x - 2 * hVal})] / (12×${hVal})</p><p><strong>Perhitungan:</strong> f'(${x}) = (-${f2_ch4.toFixed(3)} + 8(${f1_ch4.toFixed(3)}) - 8(${f_1_ch4.toFixed(3)}) + ${f_2_ch4.toFixed(3)}) / ${12 * hVal}</p><p><strong>Hasil:</strong> f'(${x}) = ${result.toFixed(3)}</p>`;
                    const derivative5_ch4 = getSymbolicDerivative(currentFunction, 5);
                    if (derivative5_ch4 && derivative5_ch4 !== "0") {
                        try {
                            const f5_value = evaluateFunction(derivative5_ch4, x);
                            const error_bound = Math.abs((Math.pow(hVal, 4) * f5_value) / 30);
                            errorAnalysis = `<h4>Orde Galat:</h4><p><strong>Rumus:</strong> O(h⁴) = (h⁴/30) × |f⁽⁵⁾(x)|</p><p><strong>Turunan simbolik:</strong> f(x) = ${currentFunction} → f⁽⁵⁾(x) = ${derivative5_ch4}</p><p><strong>Substitusi:</strong> O(h⁴) = (${hVal}⁴/30) × |f⁽⁵⁾(${x})|</p><p><strong>Evaluasi f⁽⁵⁾(${x}):</strong> f⁽⁵⁾(${x}) = ${f5_value.toFixed(3)}</p><p><strong>Perhitungan:</strong> O(h⁴) = ${(Math.pow(hVal, 4) / 30).toFixed(3)} × ${Math.abs(f5_value).toFixed(3)} = ${error_bound.toFixed(3)}</p>`;
                        } catch (e) {
                            errorAnalysis = "<h4>Orde Galat: O(h⁴)</h4><p>Galat sangat kecil dengan orde h⁴.</p>";
                        }
                    } else {
                        errorAnalysis = "<h4>Orde Galat: O(h⁴) = 0</h4><p>Untuk fungsi polinomial derajat ≤ 4, galat = 0</p>";
                    }
                    break;
            }
            return { result, steps, errorAnalysis };
        }

        // TURUNAN KEDUA: 4 metode numerik dengan analisis error
        function calculateSecondDerivative(x, method) {
            const hVal = currentH;
            let result, steps, errorAnalysis;

            switch (method) {
                case "forward-h":
                    // Selisih Maju O(h): f''(x) = (f₂ - 2f₁ + f₀) / h²
                    const f0 = findFunctionValue(x), f1 = findFunctionValue(x + hVal), f2 = findFunctionValue(x + 2 * hVal);
                    if (f0 === null || f1 === null || f2 === null) throw new Error("Nilai x tidak ditemukan dalam tabel.");
                    result = (f2 - 2 * f1 + f0) / (hVal * hVal);
                    steps = `<h4>Metode: Selisih Maju O(h) - Turunan ke-2</h4><p><strong>Rumus:</strong> f''(x) = (f₂ - 2f₁ + f₀) / h²</p><p><strong>Substitusi:</strong> f''(${x}) = [f(${x + 2 * hVal}) - 2f(${x + hVal}) + f(${x})] / ${hVal}²</p><p><strong>Perhitungan:</strong> f''(${x}) = (${f2.toFixed(3)} - 2(${f1.toFixed(3)}) + ${f0.toFixed(3)}) / ${(hVal * hVal).toFixed(3)}</p><p><strong>Hasil:</strong> f''(${x}) = ${result.toFixed(3)}</p>`;
                    const derivative3_f2h = getSymbolicDerivative(currentFunction, 3);
                    if (derivative3_f2h && derivative3_f2h !== "0") {
                        try {
                            const f3_value = evaluateFunction(derivative3_f2h, x);
                            const error_bound = Math.abs(hVal * f3_value);
                            errorAnalysis = `<h4>Orde Galat:</h4><p><strong>Rumus:</strong> O(h) = h × |f'''(x)|</p><p><strong>Turunan simbolik:</strong> f(x) = ${currentFunction} → f'''(x) = ${derivative3_f2h}</p><p><strong>Substitusi:</strong> O(h) = ${hVal} × |f'''(${x})|</p><p><strong>Evaluasi f'''(${x}):</strong> f'''(${x}) = ${f3_value.toFixed(3)}</p><p><strong>Perhitungan:</strong> O(h) = ${hVal} × ${Math.abs(f3_value).toFixed(3)} = ${error_bound.toFixed(3)}</p>`;
                        } catch (e) {
                            errorAnalysis = "<h4>Orde Galat: O(h)</h4><p>Tidak dapat menghitung bound error untuk fungsi ini.</p>";
                        }
                    } else {
                        errorAnalysis = "<h4>Orde Galat: O(h) = 0</h4><p>Untuk fungsi polinomial derajat ≤ 2, galat = 0</p>";
                    }
                    break;
                case "forward-h2":
                    // Selisih Maju O(h²): f''(x) = (2f₀ - 5f₁ + 4f₂ - f₃) / h²
                    const f0_fh2 = findFunctionValue(x), f1_fh2 = findFunctionValue(x + hVal), f2_fh2 = findFunctionValue(x + 2 * hVal), f3_fh2 = findFunctionValue(x + 3 * hVal);
                    if (f0_fh2 === null || f1_fh2 === null || f2_fh2 === null || f3_fh2 === null) throw new Error("Nilai x tidak ditemukan dalam tabel.");
                    result = (2 * f0_fh2 - 5 * f1_fh2 + 4 * f2_fh2 - f3_fh2) / (hVal * hVal);
                    steps = `<h4>Metode: Selisih Maju O(h²) - Turunan ke-2</h4><p><strong>Rumus:</strong> f''(x) = (2f₀ - 5f₁ + 4f₂ - f₃) / h²</p><p><strong>Substitusi:</strong> f''(${x}) = [2f(${x}) - 5f(${x + hVal}) + 4f(${x + 2 * hVal}) - f(${x + 3 * hVal})] / ${hVal}²</p><p><strong>Perhitungan:</strong> f''(${x}) = (2(${f0_fh2.toFixed(3)}) - 5(${f1_fh2.toFixed(3)}) + 4(${f2_fh2.toFixed(3)}) - ${f3_fh2.toFixed(3)}) / ${(hVal * hVal).toFixed(3)}</p><p><strong>Hasil:</strong> f''(${x}) = ${result.toFixed(3)}</p>`;
                    const derivative4_fh2 = getSymbolicDerivative(currentFunction, 4);
                    if (derivative4_fh2 && derivative4_fh2 !== "0") {
                        try {
                            const f4_value = evaluateFunction(derivative4_fh2, x);
                            const error_bound = Math.abs((hVal * hVal * f4_value) / 12);
                            errorAnalysis = `<h4>Orde Galat:</h4><p><strong>Rumus:</strong> O(h²) = (h²/12) × |f⁽⁴⁾(x)|</p><p><strong>Turunan simbolik:</strong> f(x) = ${currentFunction} → f⁽⁴⁾(x) = ${derivative4_fh2}</p><p><strong>Substitusi:</strong> O(h²) = (${hVal}²/12) × |f⁽⁴⁾(${x})|</p><p><strong>Evaluasi f⁽⁴⁾(${x}):</strong> f⁽⁴⁾(${x}) = ${f4_value.toFixed(3)}</p><p><strong>Perhitungan:</strong> O(h²) = ${((hVal * hVal) / 12).toFixed(3)} × ${Math.abs(f4_value).toFixed(3)} = ${error_bound.toFixed(3)}</p>`;
                        } catch (e) {
                            errorAnalysis = "<h4>Orde Galat: O(h²)</h4><p>Tidak dapat menghitung bound error untuk fungsi ini.</p>";
                        }
                    } else {
                        errorAnalysis = "<h4>Orde Galat: O(h²) = 0</h4><p>Untuk fungsi polinomial derajat ≤ 3, galat = 0</p>";
                    }
                    break;
                case "central-h2":
                    // Selisih Pusat O(h²): f''(x) = (f₁ - 2f₀ + f₋₁) / h²
                    const f0_c2h2 = findFunctionValue(x), f1_c2h2 = findFunctionValue(x + hVal), f_1_c2h2 = findFunctionValue(x - hVal);
                    if (f0_c2h2 === null || f1_c2h2 === null || f_1_c2h2 === null) throw new Error("Nilai x tidak ditemukan dalam tabel.");
                    result = (f1_c2h2 - 2 * f0_c2h2 + f_1_c2h2) / (hVal * hVal);
                    steps = `<h4>Metode: Selisih Pusat O(h²) - Turunan ke-2</h4><p><strong>Rumus:</strong> f''(x) = (f_₁ - 2f₀ + f1) / h²</p><p><strong>Substitusi:</strong> f''(${x}) = [f(${x + hVal}) - 2f(${x}) + f(${x - hVal})] / ${hVal}²</p><p><strong>Perhitungan:</strong> f''(${x}) = (${f1_c2h2.toFixed(3)} - 2(${f0_c2h2.toFixed(3)}) + ${f_1_c2h2.toFixed(3)}) / ${(hVal * hVal).toFixed(3)}</p><p><strong>Hasil:</strong> f''(${x}) = ${result.toFixed(3)}</p>`;
                    const derivative4_c2h2 = getSymbolicDerivative(currentFunction, 4);
                    if (derivative4_c2h2 && derivative4_c2h2 !== "0") {
                        try {
                            const f4_value = evaluateFunction(derivative4_c2h2, x);
                            const error_bound = Math.abs((hVal * hVal * f4_value) / 12);
                            errorAnalysis = `<h4>Orde Galat:</h4><p><strong>Rumus:</strong> O(h²) = (h²/12) × |f⁽⁴⁾(x)|</p><p><strong>Turunan simbolik:</strong> f(x) = ${currentFunction} → f⁽⁴⁾(x) = ${derivative4_c2h2}</p><p><strong>Substitusi:</strong> O(h²) = (${hVal}²/12) × |f⁽⁴⁾(${x})|</p><p><strong>Evaluasi f⁽⁴⁾(${x}):</strong> f⁽⁴⁾(${x}) = ${f4_value.toFixed(3)}</p><p><strong>Perhitungan:</strong> O(h²) = ${((hVal * hVal) / 12).toFixed(3)} × ${Math.abs(f4_value).toFixed(3)} = ${error_bound.toFixed(3)}</p>`;
                        } catch (e) {
                            errorAnalysis = "<h4>Orde Galat: O(h²)</h4><p>Tidak dapat menghitung bound error untuk fungsi ini.</p>";
                        }
                    } else {
                        errorAnalysis = "<h4>Orde Galat: O(h²) = 0</h4><p>Untuk fungsi polinomial derajat ≤ 3, galat = 0</p>";
                    }
                    break;
                case "central-h4":
                    // Selisih Pusat O(h⁴): f''(x) = (-f₂ + 16f₁ - 30f₀ + 16f₋₁ - f₋₂) / (12h²)
                    const f0_c4h2 = findFunctionValue(x), f1_c4h2 = findFunctionValue(x + hVal), f2_c4h2 = findFunctionValue(x + 2 * hVal), f_1_c4h2 = findFunctionValue(x - hVal), f_2_c4h2 = findFunctionValue(x - 2 * hVal);
                    if (f0_c4h2 === null || f1_c4h2 === null || f2_c4h2 === null || f_1_c4h2 === null || f_2_c4h2 === null) throw new Error("Nilai x tidak ditemukan dalam tabel.");
                    result = (-f2_c4h2 + 16 * f1_c4h2 - 30 * f0_c4h2 + 16 * f_1_c4h2 - f_2_c4h2) / (12 * hVal * hVal);
                    steps = `<h4>Metode: Selisih Pusat O(h⁴) - Turunan ke-2</h4><p><strong>Rumus:</strong> f''(x) = (-f₂ + 16f₁ - 30f₀ + 16f₋₁ - f₋₂) / (12h²)</p><p><strong>Substitusi:</strong> f''(${x}) = [-f(${x + 2 * hVal}) + 16f(${x + hVal}) - 30f(${x}) + 16f(${x - hVal}) - f(${x - 2 * hVal})] / (12×${hVal}²)</p><p><strong>Perhitungan:</strong> f''(${x}) = (-${f2_c4h2.toFixed(3)} + 16(${f1_c4h2.toFixed(3)}) - 30(${f0_c4h2.toFixed(3)}) + 16(${f_1_c4h2.toFixed(3)}) - ${f_2_c4h2.toFixed(3)}) / ${(12 * hVal * hVal).toFixed(3)}</p><p><strong>Hasil:</strong> f''(${x}) = ${result.toFixed(3)}</p>`;
                    errorAnalysis = "<h4>Orde Galat: O(h⁴)</h4><p>Galat sangat kecil dengan orde h⁴ untuk turunan ke-2.</p>";
                    break;
                default:
                    throw new Error("Metode tidak didukung untuk turunan ke-2");
            }
            return { result, steps, errorAnalysis };
        }
        
        // FUNGSI UTAMA: Koordinator perhitungan turunan
        function calculateDerivative() {
            const xCalc = parseFloat(document.getElementById("xCalc").value);
            const derivativeOrder = parseInt(document.getElementById("derivativeOrder").value);
            const method = document.getElementById("methodSelect").value;

            if (functionData.length === 0) return showError("calcError", "Buat tabel nilai fungsi terlebih dahulu.");
            if (isNaN(xCalc)) return showError("calcError", "Masukkan nilai x yang valid.");

            try {
                let derivativeResult;
                if (derivativeOrder === 1) {
                    derivativeResult = calculateFirstDerivative(xCalc, method);
                } else {
                    derivativeResult = calculateSecondDerivative(xCalc, method);
                }
                document.getElementById("calculationResults").innerHTML = `<div class="results">${derivativeResult.steps}${derivativeResult.errorAnalysis}</div>`;
            } catch (error) {
                showError("calcError", error.message);
            }
        }
    </script>
</body>
</html>
