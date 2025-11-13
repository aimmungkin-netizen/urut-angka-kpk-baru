<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <title>Urutkan Angka 4 Digit (Tanggal Sama: Urut dari Bawah)</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f6f8fa; margin: 20px; }
    h2 { text-align: center; }
    textarea, input { width: 100%; box-sizing: border-box; padding: 8px; margin: 6px 0; }
    textarea { height: 260px; resize: vertical; }
    button { padding: 8px 16px; margin: 5px; cursor: pointer; border: none; border-radius: 6px; }
    .process { background: #007bff; color: white; }
    .copy { background: #28a745; color: white; }
    .reset { background: #dc3545; color: white; }
    #resultBox {
      white-space: pre-wrap;
      background: #fff;
      border: 1px solid #ccc;
      padding: 10px;
      border-radius: 6px;
      margin-top: 10px;
      min-height: 120px;
    }
    #counter { margin-top: 6px; font-weight: bold; color: #333; }
    #infoUrut { color: #555; font-size: 0.9em; margin-bottom: 6px; }
  </style>
</head>
<body>

  <h2>📅 Urutkan Angka 4 Digit</h2>
  <div id="infoUrut">Urutan hasil: <b>dari tanggal paling lama ➜ terbaru</b><br>Jika tanggal sama → urut dari baris terbawah (karena input terbaru biasanya di atas).</div>

  <label><b>Tempel data (ada tanggal & 4 digit di baris):</b></label>
  <textarea id="dataInput" placeholder="Contoh:&#10;1	11 Nov 2025	CAM - 264	6523&#10;2	11 Nov 2025	CAM - 265	1451&#10;3	10 Nov 2025	CAM - 263	7242"></textarea>

  <div>
    <button class="process" onclick="prosesData()">Proses</button>
    <button class="copy" onclick="salinHasil()">Salin Hasil</button>
    <button class="reset" onclick="resetSemua()">Reset</button>
  </div>

  <div id="counter"></div>
  <div id="resultBox"></div>

  <script>
    function getMonthFromString(mStr) {
      if (!mStr) return undefined;
      const s = String(mStr).toLowerCase().replace(/\./g,'').trim();
      if (/^\d+$/.test(s)) {
        const n = parseInt(s);
        if (n >= 1 && n <= 12) return n - 1;
        return undefined;
      }
      if (s.startsWith('jan')) return 0;
      if (s.startsWith('feb')) return 1;
      if (s.startsWith('mar')) return 2;
      if (s.startsWith('apr')) return 3;
      if (s.startsWith('mei') || s.startsWith('may')) return 4;
      if (s.startsWith('jun')) return 5;
      if (s.startsWith('jul')) return 6;
      if (s.startsWith('ag') || s.startsWith('agu') || s.startsWith('agt')) return 7;
      if (s.startsWith('sep')) return 8;
      if (s.startsWith('okt') || s.startsWith('oct')) return 9;
      if (s.startsWith('nov')) return 10;
      if (s.startsWith('des') || s.startsWith('dec')) return 11;
      return undefined;
    }

    function prosesData() {
      const raw = document.getElementById('dataInput').value;
      const data = raw ? raw.trim() : '';
      const resultBox = document.getElementById('resultBox');
      const counter = document.getElementById('counter');
      resultBox.textContent = '';
      counter.textContent = '';

      if (!data) { alert('Mohon tempelkan data terlebih dahulu.'); return; }

      const lines = data.split(/\r?\n/);
      const records = [];

      // Melewatkan nomor urut awal (opsional), lalu: day, month (text/angka), year, ... 4 digit terakhir
      const regex = /^\s*(?:\d+\s+)?(\d{1,2})\s*(?:[-\/\s]+)\s*([A-Za-z]+|\d{1,2})\s*(?:[-\/\s]+)\s*(\d{2,4}).*?(\d{4})\s*$/;

      lines.forEach((line, index) => {
        const match = line.match(regex);
        if (!match) return;
        let [, dStr, mStr, yStr, num] = match;
        const d = parseInt(dStr, 10);
        let y = parseInt(yStr, 10);
        if (y < 100) y += 2000;
        const m = getMonthFromString(mStr);
        if (typeof m === 'undefined') return;

        // Validasi tanggal
        const timestamp = Date.UTC(y, m, d);
        const check = new Date(timestamp);
        if (check.getUTCFullYear() !== y || check.getUTCMonth() !== m || check.getUTCDate() !== d) return;

        records.push({
          timestamp,
          number: num,
          dateStr: `${String(d).padStart(2,'0')}-${String(m+1).padStart(2,'0')}-${y}`,
          index
        });
      });

      if (records.length === 0) { alert('Tidak ditemukan baris dengan tanggal & angka 4 digit yang valid.'); return; }

      // Sort: tanggal ascending; jika tanggal sama -> index descending (baris terbawah dulu)
      records.sort((a, b) => {
        if (a.timestamp !== b.timestamp) return a.timestamp - b.timestamp;
        return b.index - a.index; // <-- perubahan penting: baris bawah (index lebih besar) muncul dulu
      });

      const angka = records.map(r => r.number);
      const tanggalAwal = records[0].dateStr;
      const tanggalAkhir = records[records.length - 1].dateStr;

      let formatted = '';
      for (let i = 0; i < angka.length; i++) {
        formatted += angka[i] + ((i + 1) % 7 === 0 ? '\n' : ' ');
      }

      counter.textContent = `Jumlah data valid: ${angka.length} | Rentang: ${tanggalAwal} → ${tanggalAkhir}`;
      resultBox.textContent = formatted.trim();
    }

    function salinHasil() {
      const text = document.getElementById('resultBox').textContent;
      if (!text) return alert('Belum ada hasil untuk disalin.');
      navigator.clipboard.writeText(text).then(()=> alert('Hasil telah disalin ke clipboard!')).catch(()=> alert('Gagal menyalin. Coba manual.'));
    }

    function resetSemua() {
      document.getElementById('dataInput').value = '';
      document.getElementById('resultBox').textContent = '';
      document.getElementById('counter').textContent = '';
    }
  </script>

</body>
</html>
