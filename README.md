<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Export Kubikasi ke Excel dengan Nilai dan Rumus</title>
    <style>
        /* Menambahkan gaya umum untuk halaman */
        body {
            font-family: 'Arial', sans-serif;
            margin: 0;
            padding: 0;
            height: 100%;
            color: white;
            background-image: url('https://example.com/path-to-your-rice-field-image.jpg'); /* Ganti dengan URL gambar sawah */
            background-size: cover;
            background-position: center;
            background-attachment: fixed;
        }

        /* Memberikan area konten dengan background transparan */
        .container {
            background-color: rgba(0, 0, 0, 0.6); /* Transparansi hitam untuk kontras */
            padding: 30px;
            max-width: 700px;
            margin: 50px auto;
            border-radius: 15px;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
        }

        h1 {
            text-align: center;
            margin-bottom: 30px;
            font-size: 2.5em;
        }

        label {
            display: block;
            margin-bottom: 8px;
            font-size: 1.1em;
        }

        input, button {
            margin: 12px 0;
            padding: 12px;
            width: 100%;
            border-radius: 8px;
            border: 1px solid #ddd;
            font-size: 1em;
        }

        button {
            background-color: #007BFF;
            color: white;
            cursor: pointer;
            font-weight: bold;
            transition: background-color 0.3s;
        }

        button:hover {
            background-color: #0056b3;
        }

        .footer-text {
            text-align: center;
            font-size: 1.2em;
            padding: 15px 0;
        }

        footer {
            text-align: center;
            position: fixed;
            bottom: 0;
            width: 100%;
            padding: 15px 0;
            background-color: rgba(0, 0, 0, 0.8);
            color: white;
            font-size: 1.1em;
        }

        footer p {
            margin: 0;
        }

        .input-group {
            margin-bottom: 20px;
        }

        .input-group input {
            width: 48%;
            margin-right: 4%;
            margin-bottom: 10px;
        }

        .input-group input:last-child {
            margin-right: 0;
        }

        @media (max-width: 768px) {
            .input-group input {
                width: 100%;
                margin-right: 0;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Kalkulator Kubikasi Saluran Irigasi</h1>
        <form id="form">
            <div class="input-group">
                <label for="length">Panjang saluran (L, meter):</label>
                <input type="number" id="length" required step="0.01">
            </div>
            
            <div class="input-group">
                <label for="topWidth">Lebar atas saluran (T, cm):</label>
                <input type="number" id="topWidth" required step="0.01">
                <label for="width">Lebar bawah saluran (B, cm):</label>
                <input type="number" id="width" required step="0.01">
            </div>

            <div class="input-group">
                <label for="height">Tinggi saluran (H, cm):</label>
                <input type="number" id="height" required step="0.01">
            </div>
           
            <div class="input-group">
                <label for="sediment">Ketebalan sedimen dari beberapa titik (cm):</label>
                <div id="sediment-inputs">
                    <input type="number" class="small-input" placeholder="Titik 1" step="0.01">
                    <input type="number" class="small-input" placeholder="Titik 2" step="0.01">
                </div>
                <div class="input-group">
                    <label for="unitPrice">Harga per m³ (IDR):</label>
                    <input type="number" id="unitPrice" step="0.01">
                </div>
                <button type="button" onclick="addSedimentInput()">Tambah Titik</button>
                <button type="button" onclick="calculateAverageSediment()">Hitung Rata-rata Sedimen</button>
            </div>

            

            <button type="button" onclick="calculateVolume()">Hitung Volume</button>
        </form>

        <h2 id="average-sediment"></h2>
        <h2 id="result"></h2>
        <h3 id="unit-volume"></h3>
        <h3 id="cost-result"></h3>
        <button type="button" onclick="exportToExcel()">Ekspor ke Excel</button>
    </div>

    <footer>
        <p>WEB INI DI BUAT OLEH <strong>WAHYU AZMI</strong></p>
    </footer>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.0/xlsx.full.min.js"></script>
    <script>
        // Fungsi yang ada sebelumnya tetap digunakan
        function addSedimentInput() {
            const container = document.getElementById('sediment-inputs');
            const input = document.createElement('input');
            input.type = 'number';
            input.className = 'small-input';
            input.placeholder = `Titik ${container.children.length + 1}`;
            input.step = '0.01';
            container.appendChild(input);
        }

        function calculateAverageSediment() {
            const inputs = document.querySelectorAll('#sediment-inputs input');
            let total = 0;
            let count = 0;

            inputs.forEach(input => {
                const value = parseFloat(input.value);
                if (!isNaN(value)) {
                    total += value;
                    count++;
                }
            });

            if (count === 0) {
                document.getElementById('average-sediment').innerText = "Masukkan setidaknya 1 titik sedimen.";
                return 0; // Mengembalikan nilai 0 jika tidak ada sedimen
            }

            const average = total / count;
            document.getElementById('average-sediment').innerText = `Rata-rata Ketebalan Sedimen: ${average.toFixed(2)} cm`;
            return average; // Mengembalikan rata-rata untuk digunakan dalam perhitungan volume
        }

        function calculateVolume() {
            const B = parseFloat(document.getElementById('width').value) / 100; // Konversi ke meter
            const T = parseFloat(document.getElementById('topWidth').value) / 100; // Konversi ke meter
            const H = parseFloat(document.getElementById('height').value) / 100; // Konversi ke meter
            const L = parseFloat(document.getElementById('length').value); // Sudah dalam meter
            const averageSediment = calculateAverageSediment() / 100; // Konversi rata-rata ke meter
            const unitPrice = parseFloat(document.getElementById('unitPrice').value); // Harga per m³

            // Hitung tinggi efektif
            const effectiveHeight = (isNaN(averageSediment) || averageSediment === 0) ? H : H - averageSediment;

            if (effectiveHeight <= 0) {
                document.getElementById('result').innerText = "Error: Tinggi saluran setelah dikurangi sedimen tidak valid!";
                return;
            }

            // Perhitungan luas penampang trapezoidal
            const area = ((B + T) / 2) * effectiveHeight; // Luas penampang dalam m²
            const volume = area * L; // Volume dalam m³

            // Tampilkan hasil
            document.getElementById('result').innerText = `Volume Saluran (dengan sedimen): ${volume.toFixed(2)} m³`;

            // Menampilkan Volume per satuan pekerjaan (Volume per meter)
            const unitVolume = volume.toFixed(2);
            document.getElementById('unit-volume').innerText = `Volume per Satuan Pekerjaan: ${unitVolume} m³`;

            // Biaya
            if (!unitPrice) {
                document.getElementById('cost-result').innerText = "Harga satuan tidak diisi, biaya tidak dapat dihitung.";
            } else {
                const totalCost = volume * unitPrice;
                document.getElementById('cost-result').innerText = `Biaya Total: IDR ${totalCost.toFixed(2)}`;
            }
        }

        function exportToExcel() {
            const B = parseFloat(document.getElementById('width').value) / 100;
            const T = parseFloat(document.getElementById('topWidth').value) / 100;
            const H = parseFloat(document.getElementById('height').value) / 100;
            const L = parseFloat(document.getElementById('length').value);
            const averageSediment = calculateAverageSediment() / 100;
            const unitPrice = parseFloat(document.getElementById('unitPrice').value);

            const effectiveHeight = (isNaN(averageSediment) || averageSediment === 0) ? H : H - averageSediment;
            const area = ((B + T) / 2) * effectiveHeight;
            const volume = area * L;
            const totalCost = volume * unitPrice;

            const wb = XLSX.utils.book_new();
            const ws_data = [
                ['No', 'Uraian', 'Satuan', 'Volume', 'Harga Satuan (Rp)', 'Jumlah Harga (Rp)', 'Penjelasan Perhitungan'],
                ['1', 'Pekerjaan Saluran', 'M3', volume.toFixed(2), unitPrice || 'Tidak diisi', unitPrice ? totalCost.toFixed(2) : 'Tidak dihitung', `Perhitungan Volume: (Luas Penampang Trapezoidal) x Panjang = ((${B.toFixed(2)} + ${T.toFixed(2)}) / 2) x ${effectiveHeight.toFixed(2)} x ${L.toFixed(2)} = Volume. Biaya: Volume x Harga Satuan.`],
                ['2', 'Total', '', '', '', unitPrice ? totalCost.toFixed(2) : 'Tidak dihitung', '']
            ];

            const ws = XLSX.utils.aoa_to_sheet(ws_data);
            XLSX.utils.book_append_sheet(wb, ws, 'Kubikasi Saluran');

            // Menambahkan rumus Excel untuk Volume dan Biaya
            ws['D2'] = { f: `(((${B.toFixed(2)} + ${T.toFixed(2)}) / 2) * ${effectiveHeight.toFixed(2)} * ${L.toFixed(2)})` }; // Rumus Volume
            ws['F2'] = { f: `D2 * ${unitPrice || 1}` }; // Rumus Harga, jika tidak ada harga satuan, gunakan 1 sebagai default

            // Mengekspor ke Excel
            XLSX.writeFile(wb, 'Kubikasi_Saluran_dengan_Rumus_dan_Nilai.xlsx');

        }
    </script>
</body>
</html>
