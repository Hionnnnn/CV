# CV
CV VCF
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Convert File by Hion</title>
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

    <div id="welcome">
        <h1>Selamat Datang Di Convert File By Hion</h1>
        <p>Masukan Kode Untuk Lanjut</p>
        <input type="text" id="accessCode" placeholder="Masukkan Kode"><br>
        <button onclick="verifyCode()">Lanjut</button>
        <p id="code-error"></p>
        <p>Ingin Membeli Kode? Chat @SENNPOP Di Tele</p>
    </div>

    <div id="main-content" class="hidden">
        <div class="menu">
            <button onclick="showSection('cv-vcf')">CV VCF</button>
            <button onclick="showSection('cv-vcf-gabung')">CV VCF GABUNG</button>
            <button onclick="showSection('pecah-vcf')">PECAH VCF</button>
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
        
        <!-- CV VCF GABUNG Section -->
        <div id="cv-vcf-gabung" class="hidden">
            <h2>Konversi Input ke VCF</h2>
            <textarea id="contactNumbers" placeholder="MASUKAN NOMOR KONTAK, pisahkan dengan baris baru..."></textarea><br>
            <input type="text" id="contactName" placeholder="MASUKAN NAMA KONTAK"><br>
            <textarea id="adminNumbers" placeholder="MASUKAN NOMOR ADMIN, pisahkan dengan baris baru..."></textarea><br>
            <input type="text" id="adminName" placeholder="MASUKAN NAMA ADMIN"><br>
            <textarea id="navyNumbers" placeholder="MASUKAN NOMOR NAVY, pisahkan dengan baris baru..."></textarea><br>
            <input type="text" id="navyName" placeholder="MASUKAN NAMA NAVY"><br>
            <input type="text" id="fileName" placeholder="Nama File Output"><br>
            <button onclick="convertToVCFGabung()">Jadikan VCF</button>
            <p id="result-gabung"></p>
        </div>

        <!-- PECAH VCF Section -->
        <div id="pecah-vcf" class="hidden">
            <h2>Pecah File VCF</h2>
            <input type="file" id="vcfFile" accept=".vcf"><br>
            <input type="text" id="newContactName" placeholder="Nama Kontak Baru"><br>
            <input type="number" id="contactLimitPecah" placeholder="Jumlah Kontak per File"><br>
            <input type="text" id="fileNamePecah" placeholder="Nama File Output"><br>
            <button onclick="splitVCF()">Pecah VCF</button>
            <p id="result-pecah"></p>
        </div>
    </div>

    <script>
        function verifyCode() {
            const codeInput = document.getElementById('accessCode').value;
            const correctCode = '051928';
            if (codeInput === correctCode) {
                document.getElementById('welcome').classList.add('hidden');
                document.getElementById('main-content').classList.remove('hidden');
            } else {
                document.getElementById('code-error').textContent = 'KODE YANG ANDA GUNAKAN SALAH';
            }
        }

        function showSection(sectionId) {
            document.getElementById('cv-vcf').classList.add('hidden');
            document.getElementById('cv-vcf-gabung').classList.add('hidden');
            document.getElementById('pecah-vcf').classList.add('hidden');
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
                let vcfContent = '';
                let fileCount = 1;
                let contactCount = 0;

                lines.forEach((line, index) => {
                    const phone = line.trim();
                    if (phone) {
                        const name = `${contactName} ${index + 1}`;

                        vcfContent += `BEGIN:VCARD\nVERSION:3.0\nFN:${name}\nTEL:${phone}\nEND:VCARD\n\n`;
                        contactCount++;

                        if (contactCount >= contactLimit) {
                            saveVCF(vcfContent, fileName, fileCount);
                            vcfContent = '';
                            contactCount = 0;
                            fileCount++;
                        }
                    }
                });

                if (vcfContent !== '') {
                    saveVCF(vcfContent, fileName, fileCount);
                }

                document.getElementById('result').textContent = 'Konversi berhasil! File VCF siap diunduh.';
            };

            reader.readAsText(file);
        }

        function convertToVCFGabung() {
            const contactNumbers = document.getElementById('contactNumbers').value.split('\n');
            const contactName = document.getElementById('contactName').value || 'Kontak';
            
            const adminNumbers = document.getElementById('adminNumbers').value.split('\n');
            const adminName = document.getElementById('adminName').value || 'Admin';
            
            const navyNumbers = document.getElementById('navyNumbers').value.split('\n');
            const navyName = document.getElementById('navyName').value || 'Navy';

            const fileName = document.getElementById('fileName').value || 'contacts';

            let vcfContent = '';
            
            contactNumbers.forEach((phone, index) => {
                if (phone.trim()) {
                    vcfContent += `BEGIN:VCARD\nVERSION:3.0\nFN:${contactName} ${index + 1}\nTEL:${phone.trim()}\nEND:VCARD\n\n`;
                }
            });

            adminNumbers.forEach((phone, index) => {
                if (phone.trim()) {
                    vcfContent += `BEGIN:VCARD\nVERSION:3.0\nFN:${adminName} ${index + 1}\nTEL:${phone.trim()}\nEND:VCARD\n\n`;
                }
            });

            navyNumbers.forEach((phone, index) => {
                if (phone.trim()) {
                    vcfContent += `BEGIN:VCARD\nVERSION:3.0\nFN:${navyName} ${index + 1}\nTEL:${phone.trim()}\nEND:VCARD\n\n`;
                }
            });

            if (vcfContent !== '') {
                saveVCF(vcfContent, fileName);
                document.getElementById('result-gabung').textContent = 'Konversi berhasil! File VCF siap diunduh.';
            } else {
                alert('Masukkan setidaknya satu nomor telepon.');
            }
        }

        function splitVCF() {
            const fileInput = document.getElementById('vcfFile');
            const file = fileInput.files[0];
            const newContactName = document.getElementById('newContactName').value || 'Kontak';
            const contactLimitPecah = parseInt(document.getElementById('contactLimitPecah').value) || 100;
            const fileNamePecah = document.getElementById('fileNamePecah').value || 'contacts';

            if (!file) {
                alert('Silakan pilih file VCF terlebih dahulu.');
                return;
            }

            const reader = new FileReader();

            reader.onload = function(event) {
                const lines = event.target.result.split('\n');
                let vcfContent = '';
                let fileCount = 1;
                let contactCount = 0;
                let isInCard = false;

                lines.forEach(line => {
                    if (line.startsWith('BEGIN:VCARD')) {
                        isInCard = true;
                        vcfContent += line + '\n';
                    } else if (line.startsWith('END:VCARD')) {
                        isInCard = false;
                        vcfContent += line + '\n';
                        contactCount++;

                        if (contactCount >= contactLimitPecah) {
                            saveVCF(vcfContent, fileNamePecah, fileCount);
                            vcfContent = '';
                            contactCount = 0;
                            fileCount++;
                        }
                    } else if (isInCard) {
                        if (line.startsWith('FN:')) {
                            const nameLine = line.split(':');
                            vcfContent += `FN:${newContactName} ${contactCount + 1}\n`;
                        } else {
                            vcfContent += line + '\n';
                        }
                    } else {
                        vcfContent += line + '\n';
                    }
                });

                if (vcfContent !== '') {
                    saveVCF(vcfContent, fileNamePecah, fileCount);
                }

                document.getElementById('result-pecah').textContent = 'File VCF berhasil dipisah! Unduh file-file VCF yang terpisah.';
            };

            reader.readAsText(file);
        }

        function saveVCF(content, fileName, fileCount) {
            const blob = new Blob([content], { type: 'text/vcard' });
            const downloadLink = document.createElement('a');
            downloadLink.href = URL.createObjectURL(blob);
            downloadLink.download = fileCount ? `${fileName}-${fileCount}.vcf` : `${fileName}.vcf`;
            downloadLink.click();
        }
    </script>

</body>
</html>
