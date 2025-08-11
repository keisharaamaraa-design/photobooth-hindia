<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Photobooth Hindia — Full Event (All Features)</title>

<!-- Fonts -->
<link href="https://fonts.googleapis.com/css2?family=Indie+Flower&family=Montserrat:wght@400;700&family=Poppins:wght@300;500;700&display=swap" rel="stylesheet">

<!-- JSZip & FileSaver (CDN) -->
<script src="https://cdn.jsdelivr.net/npm/jszip@3.10.1/dist/jszip.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/file-saver@2.0.5/dist/FileSaver.min.js"></script>

<style>
  :root{
    --bg: #f8f4ef;
    --accent: #5b4636;
    --frame: #d4b483;
    --muted:#6a5a4a;
  }
  *{box-sizing:border-box}
  body{
    margin:0;
    font-family: "Poppins", system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    background: var(--bg);
    color: #3e2f27;
    display:flex;
    flex-direction:column;
    align-items:center;
    gap:16px;
    padding:18px;
    min-height:100vh;
  }

  header{ text-align:center; color:var(--accent); }
  header h1{ margin:0; font-size:1.5rem }
  header p{ margin:6px 0 0; color:var(--muted); font-size:0.95rem }

  .stage{
    display:flex;
    gap:20px;
    flex-wrap:wrap;
    justify-content:center;
    width:100%;
    max-width:1200px;
  }

  /* camera */
  .camera-wrap{
    background:#222;
    border:14px solid var(--frame);
    border-radius:14px;
    overflow:hidden;
    position:relative;
    width:720px;
    max-width: calc(100% - 40px);
    box-shadow:0 8px 30px rgba(0,0,0,0.25);
  }
  video, canvas{
    width:100%;
    height:auto;
    display:block;
    background:#000;
    transform: translateZ(0);
  }

  /* overlay for countdown */
  .overlay {
    position:absolute;
    inset:0;
    display:flex;
    align-items:center;
    justify-content:center;
    pointer-events:none;
  }
  .countdown {
    font-size:4rem;
    color:rgba(255,255,255,0.95);
    text-shadow:0 2px 12px rgba(0,0,0,0.6);
    font-weight:700;
  }
  .countdown.small { font-size:2rem }

  /* controls */
  .controls{
    display:flex;
    flex-direction:column;
    gap:12px;
    min-width:320px;
    max-width:400px;
  }
  .controls .row{ display:flex; gap:8px; flex-wrap:wrap; align-items:center; }
  select, input[type=color], button, input[type=number], input[type=text]{
    font-family:inherit;
    font-size:1rem;
  }
  select, input[type=color], input[type=number], input[type=text]{
    padding:8px;
    border-radius:8px;
    border:1px solid #ccc;
    background:white;
  }
  button{
    background:var(--accent);
    color:white;
    border:none;
    padding:10px 14px;
    border-radius:8px;
    cursor:pointer;
  }
  button.secondary{
    background:transparent;
    color:var(--accent);
    border:1px solid #d0bfae;
  }
  .muted { color:var(--muted); font-size:0.9rem }

  /* gallery */
  .gallery{
    width:100%;
    max-width:1200px;
    display:grid;
    grid-template-columns: repeat(auto-fit, minmax(180px,1fr));
    gap:12px;
  }
  .thumb{
    background:white;
    border-radius:10px;
    padding:8px;
    box-shadow:0 6px 14px rgba(0,0,0,0.08);
    display:flex;
    flex-direction:column;
    gap:8px;
    align-items:center;
  }
  .thumb img{ width:100%; height:auto; border-radius:6px; display:block }
  .thumb .actions{ display:flex; gap:8px; width:100%; justify-content:center }
  .small { font-size:0.85rem }

  /* top-tools */
  .top-tools{
    width:100%;
    max-width:1200px;
    display:flex;
    gap:10px;
    justify-content:flex-end;
    align-items:center;
  }
  .top-tools button{ padding:8px 12px; font-size:0.95rem }

  /* responsive tweaks */
  @media (max-width:900px){
    .stage{ flex-direction:column; align-items:center }
    .controls{ width:100%; max-width:none; min-width:unset }
  }
</style>
</head>
<body>

<header>
  <h1>📸 Photobooth Hindia — Full Event</h1>
  <p>Semua fitur: 3-shot (jeda), live filter, kustom teks, ornamen/ frame, galeri, Download All (zip), Upload ke server.</p>
</header>

<div class="top-tools">
  <button id="downloadAllBtn" title="Download semua foto sebagai ZIP">📦 Download All (ZIP)</button>
  <button id="uploadAllBtn" title="Upload semua foto ke server">☁️ Upload All</button>
</div>

<section class="stage">

  <!-- kamera -->
  <div class="camera-wrap" aria-label="Live camera preview">
    <video id="video" autoplay playsinline></video>
    <canvas id="hiddenCanvas" style="display:none;"></canvas>

    <!-- decorative overlay (visual only for preview) -->
    <div id="decorativeOverlay" style="position:absolute;inset:0;pointer-events:none;display:flex;align-items:flex-start;justify-content:center;padding:14px;">
      <!-- Inline simple ornate SVG frame (Hindia vibe). You can replace with other SVG or image. -->
      <svg id="ornamentSVG" viewBox="0 0 800 200" preserveAspectRatio="xMidYMin meet" style="width:70%; max-width:560px; opacity:0.12; filter:drop-shadow(0 2px 6px rgba(0,0,0,0.25));">
        <defs>
          <linearGradient id="g" x1="0" x2="1"><stop offset="0" stop-color="#b5502f"/><stop offset="1" stop-color="#f2c96b"/></linearGradient>
        </defs>
        <text x="50%" y="28" text-anchor="middle" font-family="Montserrat, Poppins, sans-serif" font-size="28" fill="url(#g)">HINDIA</text>
        <g transform="translate(0,60)" fill="none" stroke="url(#g)" stroke-width="2">
          <path d="M20 10 C100 60, 200 60, 300 10" />
          <path d="M500 10 C600 60, 700 60, 780 10" />
        </g>
      </svg>
    </div>

    <!-- overlay for countdown / message -->
    <div class="overlay" id="overlay" aria-hidden="true" style="display:none;">
      <div class="countdown" id="countdownText">3</div>
    </div>
  </div>

  <!-- controls -->
  <aside class="controls" aria-label="Controls">
    <div class="row">
      <button id="startBtn">▶️ Mulai 3-Shot (jeda 3s)</button>
      <button id="retakeBtn" class="secondary" style="display:none">🔁 Ulangi</button>
    </div>

    <div class="row">
      <label class="muted small">Jumlah foto:
        <input id="shotsCount" type="number" min="1" max="6" value="3" style="width:72px; margin-left:8px;">
      </label>
      <label class="muted small">Jeda (detik):
        <input id="intervalSec" type="number" min="1" max="10" value="3" style="width:72px; margin-left:8px;">
      </label>
    </div>

    <div class="row">
      <label>Filter:
        <select id="filterSelect" aria-label="Pilih filter">
          <option value="sepia(0.38) contrast(1.05) brightness(1.03) saturate(1.15)">Retro Warm</option>
          <option value="grayscale(1) contrast(1.2)">Hitam Putih</option>
          <option value="contrast(1.5) saturate(1.4) hue-rotate(-20deg)">VHS Style</option>
          <option value="contrast(1.1) brightness(1.1) saturate(0.9) hue-rotate(180deg)">Cool Tone</option>
        </select>
      </label>
    </div>

    <div class="row">
      <label>Font:
        <select id="fontSelect">
          <option value="'Poppins', sans-serif">Poppins</option>
          <option value="'Indie Flower', cursive">Indie Flower</option>
          <option value="'Montserrat', sans-serif">Montserrat</option>
          <option value="'Courier New', monospace">Courier New</option>
          <option value="'Times New Roman', serif">Times New Roman</option>
        </select>
      </label>
    </div>

    <div class="row">
      <label>Warna Teks:
        <input id="colorPicker" type="color" value="#ffffff">
      </label>
      <label style="display:flex;align-items:center;gap:6px"> <input id="useCustomText" type="checkbox"> Gunakan teks kustom</label>
    </div>

    <div class="row">
      <input id="customTextInput" type="text" placeholder="Masukkan teks kustom (mis. judul lagu / lirik)" style="flex:1; display:none;">
    </div>

    <div class="row">
      <label style="display:flex;gap:8px;align-items:center;">
        <input id="applyOrnament" type="checkbox" checked> Terapkan Ornamen/Frame pada hasil
      </label>
    </div>

    <hr style="width:100%;border:none;border-top:1px solid #eee">

    <div class="row">
      <input id="serverUrl" type="text" placeholder="Masukkan URL server upload (ex: https://example.com/upload)" style="flex:1">
    </div>
    <div class="row">
      <button id="uploadAllBtn2" class="secondary">☁️ Upload All (pakai field URL)</button>
      <div class="muted small" style="flex:1">Server harus menerima POST multipart/form-data (file[])</div>
    </div>

    <div class="row">
      <p class="muted small">Quote acak Hindia akan digunakan jika teks kustom tidak dipilih.</p>
    </div>

  </aside>
</section>

<!-- gallery -->
<section style="width:100%; max-width:1200px;">
  <h3 style="margin:6px 0 10px; color:var(--accent)">📁 Galeri Hasil</h3>
  <div id="gallery" class="gallery" aria-live="polite"></div>
</section>

<footer style="width:100%; max-width:1200px; text-align:center; color:var(--muted); margin-top:14px;">
  © 2025 Photobooth Hindia • Nuansa retro & analog
</footer>

<script>
/* FULL Photobooth: multiple shots, live filter, overlay ornament, custom text, Download All zip, Upload to server */
(async function(){
  const video = document.getElementById('video');
  const hiddenCanvas = document.getElementById('hiddenCanvas');
  const overlay = document.getElementById('overlay');
  const countdownText = document.getElementById('countdownText');

  const startBtn = document.getElementById('startBtn');
  const retakeBtn = document.getElementById('retakeBtn');
  const shotsCountInput = document.getElementById('shotsCount');
  const intervalSecInput = document.getElementById('intervalSec');
  const filterSelect = document.getElementById('filterSelect');
  const fontSelect = document.getElementById('fontSelect');
  const colorPicker = document.getElementById('colorPicker');

  const galleryEl = document.getElementById('gallery');

  const decorativeOverlay = document.getElementById('decorativeOverlay');
  const ornamentToggle = document.getElementById('applyOrnament');
  const ornamentSVG = document.getElementById('ornamentSVG');

  const useCustomTextCheckbox = document.getElementById('useCustomText');
  const customTextInput = document.getElementById('customTextInput');

  const downloadAllBtn = document.getElementById('downloadAllBtn');
  const uploadAllBtn = document.getElementById('uploadAllBtn');
  const uploadAllBtn2 = document.getElementById('uploadAllBtn2');
  const serverUrlField = document.getElementById('serverUrl');

  // Quotes Hindia (bisa diubah/ditambah)
  const hindiaQuotes = [
    "Kita ke sana hanya untuk pulang",
    "Tak ada yang abadi, tak ada yang abadi",
    "Kita di sini sekarang, esok entah di mana",
    "Apa kabar? Aku baik-baik saja",
    "Rumah yang kita pilih sendiri",
    "Langit pelangi, hati rindu",
    "Pelan-pelan saja kita jalani"
  ];

  function getRandomQuote(){ return hindiaQuotes[Math.floor(Math.random()*hindiaQuotes.length)]; }

  // minta izin kamera
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ video: { width:1280, height:720 }, audio:false });
    video.srcObject = stream;
  } catch(err){
    alert("Gagal mengakses kamera — pastikan browser mengizinkan akses kamera. Error: " + err);
    startBtn.disabled = true;
  }

  // live preview filter
  filterSelect.addEventListener('change', () => { video.style.filter = filterSelect.value; });
  video.style.filter = filterSelect.value;

  // custom text toggle
  useCustomTextCheckbox.addEventListener('change', () => {
    customTextInput.style.display = useCustomTextCheckbox.checked ? 'block' : 'none';
  });

  // decorative overlay visibility (preview only)
  ornamentToggle.addEventListener('change', () => {
    decorativeOverlay.style.display = ornamentToggle.checked ? 'flex' : 'none';
  });

  decorativeOverlay.style.display = ornamentToggle.checked ? 'flex' : 'none';

  // helper: sleep ms
  const sleep = ms => new Promise(res => setTimeout(res, ms));

  // helper: convert dataURL to blob
  function dataURLToBlob(dataURL) {
    const parts = dataURL.split(';base64,');
    const contentType = parts[0].split(':')[1];
    const byteString = atob(parts[1]);
    const arrayBuffer = new ArrayBuffer(byteString.length);
    const uint8Array = new Uint8Array(arrayBuffer);
    for (let i = 0; i < byteString.length; i++) uint8Array[i] = byteString.charCodeAt(i);
    return new Blob([uint8Array], { type: contentType });
  }

  // wrap text helper: returns array of lines
  function wrapText(ctx, text, maxWidth) {
    const words = text.split(' ');
    const lines = [];
    let current = '';
    for (let w of words) {
      const test = current ? current + ' ' + w : w;
      if (ctx.measureText(test).width > maxWidth) {
        if (current) lines.push(current);
        current = w;
      } else {
        current = test;
      }
    }
    if (current) lines.push(current);
    return lines;
  }

  // draw ornament SVG onto canvas (returns Promise)
  async function drawOrnamentOnto(ctx, vw, vh) {
    // Use the SVG element's outerHTML to create an image
    try {
      const svg = ornamentSVG.cloneNode(true);
      // adapt SVG width to canvas width
      svg.setAttribute('width', vw * 0.7);
      svg.setAttribute('height', Math.floor(vh * 0.15));
      const svgStr = new XMLSerializer().serializeToString(svg);
      const svgBlob = new Blob([svgStr], { type: 'image/svg+xml;charset=utf-8' });
      const url = URL.createObjectURL(svgBlob);
      await new Promise((res, rej) => {
        const img = new Image();
        img.onload = () => {
          // place near top center
          const x = (vw - img.width)/2;
          const y = 14;
          ctx.drawImage(img, x, y, img.width, img.height);
          URL.revokeObjectURL(url);
          res();
        };
        img.onerror = (e)=>{ URL.revokeObjectURL(url); res(); };
        img.src = url;
      });
    } catch(e){
      // fail silently
      console.warn('ornament draw failed', e);
    }
  }

  // capture single frame -> return dataURL
  async function captureShot(filterValue, fontValue, colorValue, quoteText, applyOrn) {
    // set canvas same size as video feed
    const vw = video.videoWidth || 640;
    const vh = video.videoHeight || 480;
    hiddenCanvas.width = vw;
    hiddenCanvas.height = vh;
    const ctx = hiddenCanvas.getContext('2d');

    // apply same filter to canvas drawing
    ctx.filter = filterValue || "none";
    ctx.drawImage(video, 0, 0, vw, vh);

    // apply ornament if required
    if (applyOrn) {
      await drawOrnamentOnto(ctx, vw, vh);
    }

    // draw overlay text (quote or custom)
    const fontSize = Math.max(28, Math.floor(vw * 0.05)); // responsive size
    ctx.textAlign = "center";
    ctx.fillStyle = colorValue || "#fff";
    // set font family — ensure no quotes cause issues
    const safeFont = fontValue.replace(/['"]/g,'') || 'Poppins';
    ctx.font = `${fontSize}px ${safeFont}`;
    // shadow for readability
    ctx.shadowColor = "rgba(0,0,0,0.45)";
    ctx.shadowBlur = 10;
    ctx.shadowOffsetX = 0;
    ctx.shadowOffsetY = 3;

    // allow multi-line if long
    const maxWidth = vw * 0.86;
    const lines = wrapText(ctx, quoteText, maxWidth);
    const lineHeight = fontSize * 1.1;
    const startY = vh - (lines.length * lineHeight) - 40; // 40px from bottom
    lines.forEach((ln, i) => {
      ctx.fillText(ln, vw/2, startY + i*lineHeight);
    });

    return hiddenCanvas.toDataURL('image/png');
  }

  // add image to gallery UI
  function addToGallery(dataUrl, index){
    const card = document.createElement('div');
    card.className = 'thumb';
    card.innerHTML = `
      <img src="${dataUrl}" alt="Shot ${index+1}">
      <div style="width:100%; display:flex; justify-content:space-between; align-items:center;">
        <div class="small">Foto ${index+1}</div>
        <div class="actions">
          <button class="downloadBtn">⬇️</button>
          <button class="delBtn secondary">🗑️</button>
        </div>
      </div>
    `;

    // download single
    const dlBtn = card.querySelector('.downloadBtn');
    dlBtn.addEventListener('click', () => {
      const a = document.createElement('a');
      a.href = dataUrl;
      a.download = `photobooth-hindia-shot${index+1}.png`;
      a.click();
    });

    // delete
    const delBtn = card.querySelector('.delBtn');
    delBtn.addEventListener('click', () => {
      card.remove();
    });

    galleryEl.appendChild(card);
  }

  // main: do a sequence of shots
  let running = false;
  startBtn.addEventListener('click', async () => {
    if (running) return;
    running = true;
    startBtn.disabled = true;
    retakeBtn.style.display = 'none';

    const shotsCount = Math.max(1, Math.min(6, parseInt(shotsCountInput.value || 3)));
    const intervalSec = Math.max(1, Math.min(10, parseInt(intervalSecInput.value || 3)));

    overlay.style.display = 'flex';
    // sequence
    for (let i=0; i<shotsCount; i++){
      // countdown = intervalSec..1
      for (let t = intervalSec; t>=1; t--){
        countdownText.textContent = t;
        await sleep(1000);
      }
      // SNAP
      countdownText.textContent = "SNAP!";
      // choose quote or custom text
      const quote = useCustomTextCheckbox.checked && customTextInput.value.trim()
        ? customTextInput.value.trim()
        : getRandomQuote();

      const dataUrl = await captureShot(filterSelect.value, fontSelect.value, colorPicker.value, quote, ornamentToggle.checked);
      addToGallery(dataUrl, galleryEl.children.length);
      // small pause
      await sleep(600);
    }

    overlay.style.display = 'none';
    startBtn.disabled = false;
    running = false;
    retakeBtn.style.display = 'inline-block';
  });

  // retake: scroll to top (no deleting)
  retakeBtn.addEventListener('click', () => { window.scrollTo({ top: 0, behavior: 'smooth' }); });

  // Download All (zip)
  downloadAllBtn.addEventListener('click', async () => {
    const items = Array.from(galleryEl.querySelectorAll('.thumb img'));
    if (!items.length) { alert('Galeri kosong — ambil foto dulu.'); return; }
    const zip = new JSZip();
    const folder = zip.folder('photobooth-hindia');
    // fetch blobs and add
    for (let i=0; i<items.length; i++){
      const url = items[i].src;
      const blob = dataURLToBlob(url);
      folder.file(`shot-${i+1}.png`, blob);
    }
    const content = await zip.generateAsync({type:"blob"});
    saveAs(content, 'photobooth-hindia-gallery.zip');
  });

  // Upload All (takes gallery images and POST to serverUrl)
  async function uploadAllToServer(urlField){
    const url = urlField.trim();
    if (!url) { alert('Masukkan URL server pada field di sebelah kiri.'); return; }
    const items = Array.from(galleryEl.querySelectorAll('.thumb img'));
    if (!items.length) { alert('Galeri kosong — ambil foto dulu.'); return; }

    // build formdata
    const form = new FormData();
    for (let i=0; i<items.length; i++){
      const blob = dataURLToBlob(items[i].src);
      const filename = `photobooth-shot-${i+1}.png`;
      form.append('file[]', blob, filename);
    }

    try {
      const resp = await fetch(url, { method: 'POST', body: form });
      if (!resp.ok) {
        const txt = await resp.text().catch(()=>resp.statusText);
        alert('Upload gagal: ' + resp.status + ' — ' + txt);
        return;
      }
      const result = await resp.json().catch(()=>({status:'ok'}));
      alert('Upload berhasil! Server response: ' + JSON.stringify(result));
    } catch (e) {
      alert('Gagal meng-upload: ' + e);
    }
  }

  // upload all (top button) uses serverUrlField value
  uploadAllBtn.addEventListener('click', () => {
    const url = serverUrlField.value.trim();
    uploadAllToServer(url);
  });
  // uploadAllBtn2 (below) triggers same
  uploadAllBtn2.addEventListener('click', () => {
    const url = serverUrlField.value.trim();
    uploadAllToServer(url);
  });

  // accessibility: allow Enter to start
  startBtn.addEventListener('keyup', (e)=>{ if(e.key==='Enter') startBtn.click(); });

})();
</script>

</body>
</html>
