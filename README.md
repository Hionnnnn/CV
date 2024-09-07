<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Convert dan Gabung File by Hion</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 50px;
        }
        input[type="file"], input[type="text"], input[type="number"], textarea {
            margin: 10px 0;
            padding: 10px;
            width: 300px;
            font-size: 16px;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
        }
        #result, #result-gabung, #result-pecah, #code-error {
            margin-top: 20px;
            color: green;
        }
        #code-error {
            color: red;
        }
        .hidden {
            display: none;
        }
        .menu {
            margin-top: 20px;
        }
        .menu button {
            margin: 5px;
        }
    </style>
</head>
<body>

    <div id="main-content">
        <div class="menu">
            <button onclick="showSection('cv-vcf')">CV VCF</button>
            <button onclick="showSection('pecah-vcf')">PECAH VCF</button>
            <button onclick="showSection('gabung-vcf')">GABUNG VCF</button>
        </div>

        <!-- CV VCF Section -->
        <div id="cv-vcf" class="hidden">
            <h2>Konversi File TXT ke VCF</h2>
            <input type="file" id="txtFile" accept=".txt"><br>
            <input type="text" id="contactName" placeholder="Nama Kontak"><br>
            <input type="number" id="contactLimit" placeholder="Jumlah Kontak per File"><br>
            <input type="text" id="fileName" placeholder="Nama File Output"><br>
            <button onclick="convertToVCF()">Konversi ke VCF</button>
            <p id="result"></p>
        </div>
        
        <!-- PECAH VCF Section -->
        <div id="pecah-vcf" class="hidden">
            <h2>Pecah File VCF</h2>
            <input type="file" id="vcfFile" accept=".vcf"><br>
            <input type="number" id="contactLimitPecah" placeholder="Jumlah Kontak per File"><br>
            <input type="text" id="fileNamePecah" placeholder="Nama File Output"><br>
            <button onclick="splitVCF()">Pecah VCF</button>
            <p id="result-pecah"></p>
        </div>

        <!-- GABUNG VCF Section -->
        <div id="gabung-vcf" class="hidden">
            <h2>Gabung Kontak VCF</h2>
            <textarea id="noAdmin" rows="5" placeholder="Masukkan nomor admin, pisahkan dengan baris baru..."></textarea><br>
            <input type="text" id="namaAdmin" placeholder="Masukkan nama admin"><br>
            <textarea id="noNavy" rows="5" placeholder="Masukkan nomor navy, pisahkan dengan baris baru..."></textarea><br>
            <input type="text" id="namaNavy" placeholder="Masukkan nama navy"><br>
            <textarea id="noKontak" rows="5" placeholder="Masukkan nomor kontak lain, pisahkan dengan baris baru..."></textarea><br>
            <input type="text" id="namaKontak" placeholder="Masukkan nama kontak lain"><br>
            <input type="text" id="fileNameGabungVCF" placeholder="Nama file output"><br>
            <button onclick="gabungVCF()">Gabungkan VCF</button>
            <p id="result-gabung-vcf"></p>
        </div>
    </div>

    <script>
        function showSection(sectionId) {
            const sections = ['cv-vcf', 'pecah-vcf', 'gabung-vcf'];
            sections.forEach(section => {
                document.getElementById(section).classList.add('hidden');
            });
            document.getElementById(sectionId).classList.remove('hidden');
        }

        function convertToVCF() {
            const fileInput = document.getElementById('txtFile');
            const file = fileInput.files[0];
            const contactName = document.getElementById('contactName').value || 'Kontak';
            const contactLimit = parseInt(document.getElementById('contactLimit').value) || 100;
            const fileName = document.getElementById('fileName').value || 'contacts';

            if (!file) {
                alert('Silakan pilih file TXT terlebih dahulu.');
                return;
            }

            const reader = new FileReader();

            reader.onload = function(event) {
                const lines = event.target.result.split('\n');
                const zip = new JSZip();
                let vcfContent = '';
                let fileCount = 1;
                let contactCount = 0;

                lines.forEach((line, index) => {
                    const phone = line.trim();
                    if (phone) {
                        const name = `${contactName} ${index + 1}`;
                        vcfContent += `BEGIN:VCARD\nVERSION:3.0\nFN:${name}\nTEL:+${phone}\nEND:VCARD\n\n`;
                        contactCount++;

                        if (contactCount >= contactLimit) {
                            zip.file(`${fileName}-${fileCount}.vcf`, vcfContent);
                            vcfContent = '';
                            contactCount = 0;
                            fileCount++;
                        }
                    }
                });

                if (vcfContent !== '') {
                    zip.file(`${fileName}-${fileCount}.vcf`, vcfContent);
                }

                zip.generateAsync({ type: 'blob' }).then(function(content) {
                    const link = document.createElement('a');
                    link.href = URL.createObjectURL(content);
                    link.download = `${fileName}.zip`;
                    link.click();
                });

                document.getElementById('result').textContent = 'Konversi berhasil! File ZIP siap diunduh.';
            };

            reader.readAsText(file);
        }

        function splitVCF() {
            const fileInput = document.getElementById('vcfFile');
            const file = fileInput.files[0];
            const contactLimit = parseInt(document.getElementById('contactLimitPecah').value) || 100;
            const fileName = document.getElementById('fileNamePecah').value || 'contacts';

            if (!file) {
                alert('Silakan pilih file VCF terlebih dahulu.');
                return;
            }

            const reader = new FileReader();

            reader.onload = function(event) {
                const vcfLines = event.target.result.split('\n');
                const zip = new JSZip();
                let vcfContent = '';
                let fileCount = 1;
                let contactCount = 0;
                let inVCard = false;

                vcfLines.forEach(line => {
                    if (line.startsWith('BEGIN:VCARD')) {
                        inVCard = true;
                    }

                    if (inVCard) {
                        vcfContent += line + '\n';
                    }

                    if (line.startsWith('END:VCARD')) {
                        contactCount++;
                        inVCard = false;

                        if (contactCount >= contactLimit) {
                            zip.file(`${fileName}-${fileCount}.vcf`, vcfContent);
                            vcfContent = '';
                            contactCount = 0;
                            fileCount++;
                        }
                    }
                });

                if (vcfContent !== '') {
                    zip.file(`${fileName}-${fileCount}.vcf`, vcfContent);
                }

                zip.generateAsync({ type: 'blob' }).then(function(content) {
                    const link = document.createElement('a');
                    link.href = URL.createObjectURL(content);
                    link.download = `${fileName}.zip`;
                    link.click();
                });

                document.getElementById('result-pecah').textContent = 'Pecah file VCF berhasil! File ZIP siap diunduh.';
            };

            reader.readAsText(file);
        }

        function gabungVCF() {
            const noAdmin = document.getElementById('noAdmin').value.trim().split('\n').filter(Boolean);
            const namaAdmin = document.getElementById('namaAdmin').value.trim();
            const noNavy = document.getElementById('noNavy').value.trim().split('\n').filter(Boolean);
            const namaNavy = document.getElementById('namaNavy').value.trim();
            const noKontak = document.getElementById('noKontak').value.trim().split('\n').filter(Boolean);
            const namaKontak = document.getElementById('namaKontak').value.trim();
            const fileName = document.getElementById('fileNameGabungVCF').value.trim() || 'combined';

            if (noAdmin.length === 0 && noNavy.length === 0 && noKontak.length === 0) {
                alert('Silakan masukkan nomor kontak.');
                return;
            }

            let vcfContent = '';

            function addContacts(numbers, namePrefix) {
                numbers.forEach((number, index) => {
                    const name = `${namePrefix} ${index + 1}`;
                    vcfContent += `BEGIN:VCARD\nVERSION:3.0\nFN:${name}\nTEL:+${number}\nEND:VCARD\n\n`;
                });
            }

            // Tambahkan kontak
            if (noAdmin.length > 0) addContacts(noAdmin, namaAdmin);
            if (noNavy.length > 0) addContacts(noNavy, namaNavy);
            if (noKontak.length > 0) addContacts(noKontak, namaKontak);

            // Buat file VCF dan tawarkan unduhan
            const blob = new Blob([vcfContent], { type: 'text/vcard' });
            const link = document.createElement('a');
            link.href = URL.createObjectURL(blob);
            link.download = `${fileName}.vcf`;
            link.click();

            document.getElementById('result-gabung-vcf').textContent = 'Gabung VCF berhasil! File siap diunduh.';
        }
    </script>
    <!-- JSZip library -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
</body>
</html>
