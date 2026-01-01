<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Bloxd Schematic Uploader</title>

    <!-- Basic Reset & Layout -->
    <style>
        :root {
            --bg-gradient: radial-gradient(circle at top, #2df4ff 0, #001322 45%, #00060c 100%);
            --accent: #2df4ff;
            --accent-soft: rgba(45, 244, 255, 0.25);
            --accent-strong: rgba(45, 244, 255, 0.8);
            --danger: #ff4d6a;
            --success: #53ff9c;
            --card-bg: rgba(5, 12, 25, 0.9);
            --glass-bg: rgba(10, 20, 40, 0.7);
            --border-color: rgba(255, 255, 255, 0.12);
            --shadow-soft: 0 18px 45px rgba(0, 0, 0, 0.65);
            --radius-lg: 18px;
            --radius-md: 10px;
            --transition-fast: 0.18s ease-out;
            --transition-med: 0.22s ease-out;
            --font-main: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 24px;
            background: var(--bg-gradient);
            color: #f7fbff;
            font-family: var(--font-main);
            -webkit-font-smoothing: antialiased;
        }

        .app-shell {
            width: 100%;
            max-width: 720px;
            background: linear-gradient(135deg, rgba(8, 18, 36, 0.96), rgba(2, 8, 20, 0.98));
            border-radius: 24px;
            padding: 24px 24px 20px;
            box-shadow: var(--shadow-soft);
            border: 1px solid rgba(255, 255, 255, 0.06);
            position: relative;
            overflow: hidden;
        }

        /* subtle glowing border */
        .app-shell::before {
            content: "";
            position: absolute;
            inset: -1px;
            border-radius: inherit;
            background:
                radial-gradient(circle at 0 0, rgba(45, 244, 255, 0.18), transparent 55%),
                radial-gradient(circle at 100% 0, rgba(83, 255, 156, 0.16), transparent 55%),
                radial-gradient(circle at 0 100%, rgba(0, 170, 255, 0.12), transparent 55%);
            opacity: 0.75;
            mix-blend-mode: screen;
            pointer-events: none;
        }

        .app-header {
            display: flex;
            align-items: center;
            justify-content: space-between;
            gap: 16px;
            margin-bottom: 18px;
            position: relative;
            z-index: 1;
        }

        .app-title-block h1 {
            font-size: 1.5rem;
            line-height: 1.25;
            letter-spacing: 0.03em;
        }

        .app-title-block p {
            margin-top: 4px;
            font-size: 0.9rem;
            color: rgba(230, 240, 255, 0.78);
        }

        .tag-pill {
            padding: 5px 11px;
            border-radius: 999px;
            border: 1px solid rgba(45, 244, 255, 0.5);
            font-size: 0.75rem;
            text-transform: uppercase;
            letter-spacing: 0.11em;
            color: var(--accent);
            background: radial-gradient(circle at 0 0, rgba(45, 244, 255, 0.15), transparent 55%);
            display: inline-flex;
            align-items: center;
            gap: 6px;
            white-space: nowrap;
        }

        .tag-pill-dot {
            width: 7px;
            height: 7px;
            border-radius: 999px;
            background: var(--accent);
            box-shadow: 0 0 12px rgba(45, 244, 255, 0.85);
        }

        /* Main grid */
        .app-grid {
            display: grid;
            grid-template-columns: minmax(0, 1.4fr) minmax(0, 1fr);
            gap: 18px;
            position: relative;
            z-index: 1;
        }

        @media (max-width: 720px) {
            .app-shell {
                padding: 18px 16px 16px;
            }
            .app-grid {
                grid-template-columns: minmax(0, 1fr);
            }
        }

        /* Upload card */
        .card {
            background: var(--glass-bg);
            border-radius: var(--radius-lg);
            border: 1px solid var(--border-color);
            padding: 16px 16px 14px;
            position: relative;
            overflow: hidden;
        }

        .card::before {
            content: "";
            position: absolute;
            inset: 0;
            background: linear-gradient(135deg, rgba(255, 255, 255, 0.06), transparent 35%);
            mix-blend-mode: soft-light;
            opacity: 0.8;
            pointer-events: none;
        }

        .card-header {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom: 10px;
        }

        .card-header h2 {
            font-size: 1rem;
            letter-spacing: 0.06em;
            text-transform: uppercase;
            color: rgba(235, 244, 255, 0.9);
        }

        .card-header span {
            font-size: 0.75rem;
            color: rgba(178, 198, 230, 0.8);
        }

        /* Drop zone */
        .drop-zone {
            position: relative;
            border-radius: var(--radius-md);
            border: 1.6px dashed rgba(160, 196, 255, 0.8);
            padding: 18px 16px;
            background: radial-gradient(circle at 10% -20%, rgba(45, 244, 255, 0.09), transparent 55%), rgba(5, 16, 40, 0.85);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            gap: 8px;
            cursor: pointer;
            transition: border-color var(--transition-fast), background var(--transition-med), transform var(--transition-fast), box-shadow var(--transition-med);
        }

        .drop-zone:hover {
            border-color: var(--accent-strong);
            box-shadow: 0 0 0 1px rgba(45, 244, 255, 0.3), 0 14px 28px rgba(0, 0, 0, 0.65);
            transform: translateY(-1px);
        }

        .drop-zone.drag-over {
            background: radial-gradient(circle at 50% 0, rgba(45, 244, 255, 0.12), transparent 60%), rgba(3, 11, 30, 0.95);
            border-color: var(--accent);
            box-shadow: 0 0 0 1px rgba(45, 244, 255, 0.45), 0 0 24px rgba(45, 244, 255, 0.6);
        }

        .drop-zone-icon {
            width: 40px;
            height: 40px;
            border-radius: 12px;
            background: radial-gradient(circle at 0 0, rgba(45, 244, 255, 0.3), transparent 60%), rgba(4, 16, 38, 0.95);
            display: flex;
            align-items: center;
            justify-content: center;
            border: 1px solid rgba(45, 244, 255, 0.6);
            box-shadow: 0 0 14px rgba(45, 244, 255, 0.4);
            font-size: 20px;
        }

        .drop-zone-title {
            font-size: 0.98rem;
            font-weight: 600;
            letter-spacing: 0.03em;
        }

        .drop-zone-sub {
            font-size: 0.8rem;
            color: rgba(188, 210, 240, 0.85);
            text-align: center;
        }

        .drop-zone-hint {
            margin-top: 4px;
            font-size: 0.76rem;
            color: rgba(152, 182, 220, 0.8);
        }

        .drop-zone input[type="file"] {
            position: absolute;
            inset: 0;
            width: 100%;
            height: 100%;
            opacity: 0;
            cursor: pointer;
        }

        /* File info + actions */
        .file-meta-row {
            display: flex;
            gap: 10px;
            margin-top: 14px;
            align-items: center;
            justify-content: space-between;
        }

        .file-meta {
            flex: 1;
            min-width: 0;
        }

        .file-meta-label {
            font-size: 0.74rem;
            text-transform: uppercase;
            letter-spacing: 0.09em;
            color: rgba(160, 188, 230, 0.95);
            margin-bottom: 3px;
        }

        .file-meta-name {
            font-size: 0.86rem;
            font-weight: 500;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
        }

        .file-meta-size {
            font-size: 0.78rem;
            color: rgba(172, 198, 230, 0.88);
        }

        .pill-status {
            font-size: 0.75rem;
            padding: 4px 10px;
            border-radius: 999px;
            border: 1px solid rgba(45, 244, 255, 0.65);
            color: var(--accent);
            display: inline-flex;
            align-items: center;
            gap: 6px;
        }

        .pill-status-dot {
            width: 7px;
            height: 7px;
            border-radius: 999px;
            background: var(--success);
            box-shadow: 0 0 10px rgba(83, 255, 156, 0.9);
        }

        /* Buttons */
        .btn-row {
            display: flex;
            flex-wrap: wrap;
            gap: 8px;
            margin-top: 14px;
        }

        .btn {
            border: none;
            border-radius: 999px;
            padding: 8px 16px;
            font-size: 0.86rem;
            font-weight: 500;
            letter-spacing: 0.04em;
            text-transform: uppercase;
            cursor: pointer;
            display: inline-flex;
            align-items: center;
            gap: 6px;
            transition: background var(--transition-fast), box-shadow var(--transition-fast), transform var(--transition-fast), color var(--transition-fast), border-color var(--transition-fast);
        }

        .btn-primary {
            background: linear-gradient(135deg, #2df4ff, #53ff9c);
            color: #021018;
            box-shadow: 0 10px 26px rgba(45, 244, 255, 0.35);
        }

        .btn-primary:hover:not(:disabled) {
            box-shadow: 0 0 0 1px rgba(4, 20, 36, 0.85), 0 16px 32px rgba(45, 244, 255, 0.55);
            transform: translateY(-1px);
        }

        .btn-secondary {
            background: rgba(6, 18, 40, 0.9);
            color: rgba(225, 238, 255, 0.92);
            border: 1px solid rgba(144, 174, 220, 0.7);
        }

        .btn-secondary:hover:not(:disabled) {
            background: rgba(8, 22, 50, 0.95);
            border-color: rgba(198, 220, 255, 0.9);
        }

        .btn:disabled {
            opacity: 0.45;
            cursor: not-allowed;
            box-shadow: none;
            transform: none;
        }

        .btn-icon {
            font-size: 1rem;
            line-height: 1;
        }

        /* Progress + status text */
        .status-block {
            margin-top: 10px;
            font-size: 0.8rem;
            color: rgba(180, 204, 235, 0.9);
        }

        .status-block span {
            display: inline-flex;
            align-items: center;
            gap: 6px;
        }

        .status-icon {
            font-size: 1.05rem;
        }

        .status-error {
            color: var(--danger);
        }

        .status-success {
            color: var(--success);
        }

        .progress-track {
            width: 100%;
            height: 4px;
            border-radius: 999px;
            background: rgba(10, 26, 54, 0.95);
            overflow: hidden;
            margin-top: 8px;
        }

        .progress-bar {
            height: 100%;
            width: 0%;
            border-radius: inherit;
            background: linear-gradient(90deg, #2df4ff, #53ff9c);
            transition: width 0.15s linear;
        }

        /* Side panel */
        .side-panel {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .side-card {
            background: radial-gradient(circle at 0 0, rgba(45, 244, 255, 0.14), transparent 60%), rgba(5, 12, 28, 0.94);
            border-radius: var(--radius-lg);
            border: 1px solid rgba(139, 176, 225, 0.55);
            padding: 14px 14px 12px;
            position: relative;
            overflow: hidden;
        }

        .side-card h3 {
            font-size: 0.92rem;
            text-transform: uppercase;
            letter-spacing: 0.07em;
            margin-bottom: 8px;
            color: rgba(226, 238, 255, 0.96);
        }

        .side-card p {
            font-size: 0.8rem;
            color: rgba(185, 208, 238, 0.92);
            margin-bottom: 6px;
        }

        .side-list {
            list-style: none;
            font-size: 0.78rem;
            color: rgba(176, 205, 240, 0.92);
        }

        .side-list li + li {
            margin-top: 3px;
        }

        .side-label {
            font-size: 0.76rem;
            text-transform: uppercase;
            letter-spacing: 0.11em;
            color: rgba(149, 186, 230, 0.9);
            margin-bottom: 3px;
        }

        .note-pill {
            font-size: 0.77rem;
            padding: 4px 8px;
            border-radius: 999px;
            background: rgba(6, 22, 48, 0.95);
            border: 1px solid rgba(116, 160, 222, 0.7);
            display: inline-flex;
            align-items: center;
            gap: 6px;
        }

        .note-pill span {
            font-size: 0.85rem;
        }

        /* Footer */
        .app-footer {
            margin-top: 16px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            gap: 8px;
            font-size: 0.74rem;
            color: rgba(152, 180, 214, 0.85);
            position: relative;
            z-index: 1;
        }

        .app-footer a {
            color: var(--accent);
            text-decoration: none;
        }

        .app-footer a:hover {
            text-decoration: underline;
        }
    </style>
</head>
<body>
    <main class="app-shell" aria-label="Bloxd.io .bloxdschem uploader">
        <header class="app-header">
            <div class="app-title-block">
                <h1>Bloxd.io .bloxdschem Handler</h1>
                <p>Upload, verify and download Bloxd schematics for backups, sharing and testing.</p>
            </div>
            <div class="tag-pill">
                <span class="tag-pill-dot"></span>
                <span>RabbitPlaysYT</span>
            </div>
        </header>

        <section class="app-grid">
            <!-- LEFT: main uploader -->
            <section class="card" aria-label="Upload and download panel">
                <div class="card-header">
                    <h2>Schematic file</h2>
                    <span>.bloxdschem only</span>
                </div>

                <label class="drop-zone" id="dropZone">
                    <div class="drop-zone-icon">📦</div>
                    <div class="drop-zone-title">Drop your .bloxdschem here</div>
                    <div class="drop-zone-sub">or click to browse from your device</div>
                    <div class="drop-zone-hint">Your file never leaves your browser.</div>
                    <input type="file" id="fileInput" accept=".bloxdschem" />
                </label>

                <div class="file-meta-row" id="fileMetaRow" style="display: none;">
                    <div class="file-meta">
                        <div class="file-meta-label">Selected file</div>
                        <div class="file-meta-name" id="fileName">example.bloxdschem</div>
                        <div class="file-meta-size" id="fileSize">0 bytes</div>
                    </div>
                    <div class="pill-status" id="fileStatusPill">
                        <span class="pill-status-dot"></span>
                        <span id="fileStatusText">Ready</span>
                    </div>
                </div>

                <div class="btn-row">
                    <button class="btn btn-primary" id="downloadBtn" disabled>
                        <span class="btn-icon">💾</span>
                        <span>Download copy</span>
                    </button>

                    <button class="btn btn-secondary" id="clearBtn" disabled>
                        <span class="btn-icon">🗑️</span>
                        <span>Clear file</span>
                    </button>
                </div>

                <div class="status-block" id="statusBlock" aria-live="polite">
                    <span id="statusText">
                        <span class="status-icon">ℹ️</span>
                        Waiting for a .bloxdschem file…
                    </span>
                    <div class="progress-track" id="progressTrack" style="display: none;">
                        <div class="progress-bar" id="progressBar"></div>
                    </div>
                </div>
            </section>

            <!-- RIGHT: info / helper text -->
            <aside class="side-panel" aria-label="Helper panel">
                <section class="side-card">
                    <h3>What this does</h3>
                    <p>This page lets you quickly sanity‑check a Bloxd schematic file by re‑downloading the exact bytes you loaded in.</p>
                    <ul class="side-list">
                        <li>Perfect for manual backups.</li>
                        <li>Verify that exports are intact.</li>
                        <li>Share raw .bloxdschem files with friends.</li>
                    </ul>
                </section>

                <section class="side-card">
                    <div class="side-label">Tips</div>
                    <ul class="side-list">
                        <li>Only .bloxdschem files are accepted.</li>
                        <li>Rename files after download if you want versions.</li>
                        <li>Keep copies in cloud storage or version control.</li>
                    </ul>
                </section>

                <section class="side-card">
                    <div class="side-label">Privacy</div>
                    <div class="note-pill">
                        <span>🔒</span>
                        <span>Everything runs 100% locally in your browser.</span>
                    </div>
                </section>
            </aside>
        </section>

        <footer class="app-footer">
            <span>Made for Bloxd.io schematics.</span>
            <span>v1.0 · Static HTML/JS</span>
        </footer>
    </main>

    <script>
        const fileInput = document.getElementById("fileInput");
        const dropZone = document.getElementById("dropZone");
        const downloadBtn = document.getElementById("downloadBtn");
        const clearBtn = document.getElementById("clearBtn");
        const statusBlock = document.getElementById("statusBlock");
        const statusText = document.getElementById("statusText");
        const progressTrack = document.getElementById("progressTrack");
        const progressBar = document.getElementById("progressBar");
        const fileMetaRow = document.getElementById("fileMetaRow");
        const fileNameEl = document.getElementById("fileName");
        const fileSizeEl = document.getElementById("fileSize");
        const fileStatusText = document.getElementById("fileStatusText");
        const fileStatusPill = document.getElementById("fileStatusPill");

        let uploadedFile = null;
        let originalName = "";

        function formatBytes(bytes) {
            if (!bytes && bytes !== 0) return "";
            const units = ["bytes", "KB", "MB", "GB"];
            let i = 0;
            let value = bytes;
            while (value >= 1024 && i < units.length - 1) {
                value /= 1024;
                i++;
            }
            return value.toFixed(value >= 10 || i === 0 ? 0 : 1) + " " + units[i];
        }

        function setStatus(message, type = "info") {
            let icon = "ℹ️";
            statusBlock.classList.remove("status-error", "status-success");
            if (type === "error") {
                icon = "❌";
                statusBlock.classList.add("status-error");
            } else if (type === "success") {
                icon = "✅";
                statusBlock.classList.add("status-success");
            }
            statusText.innerHTML = '<span class="status-icon">' + icon + "</span>" + message;
        }

        function resetProgress() {
            progressTrack.style.display = "none";
            progressBar.style.width = "0%";
        }

        function showFileMeta(file) {
            if (!file) {
                fileMetaRow.style.display = "none";
                return;
            }
            fileMetaRow.style.display = "flex";
            fileNameEl.textContent = file.name;
            fileSizeEl.textContent = formatBytes(file.size);
            fileStatusText.textContent = "Ready";
        }

        function clearFile() {
            uploadedFile = null;
            originalName = "";
            fileInput.value = "";
            fileMetaRow.style.display = "none";
            downloadBtn.disabled = true;
            clearBtn.disabled = true;
            resetProgress();
            setStatus("Waiting for a .bloxdschem file…");
        }

        function handleFile(file) {
            if (!file) return;

            if (!file.name.toLowerCase().endsWith(".bloxdschem")) {
                clearFile();
                setStatus("Please select a valid .bloxdschem file.", "error");
                return;
            }

            uploadedFile = file;
            originalName = file.name;

            showFileMeta(file);
            fileStatusText.textContent = "Ready";
            fileStatusPill.style.borderColor = "rgba(45, 244, 255, 0.65)";

            downloadBtn.disabled = false;
            clearBtn.disabled = false;
            resetProgress();
            setStatus("File loaded successfully. Click “Download copy” to save it back to disk.", "success");
        }

        // drag and drop behaviour
        ["dragenter", "dragover"].forEach((eventName) => {
            dropZone.addEventListener(eventName, (e) => {
                e.preventDefault();
                e.stopPropagation();
                dropZone.classList.add("drag-over");
            });
        });

        ["dragleave", "drop"].forEach((eventName) => {
            dropZone.addEventListener(eventName, (e) => {
                e.preventDefault();
                e.stopPropagation();
                if (eventName === "drop") {
                    const dt = e.dataTransfer;
                    const file = dt && dt.files && dt.files[0];
                    handleFile(file);
                }
                dropZone.classList.remove("drag-over");
            });
        });

        dropZone.addEventListener("click", () => {
            fileInput.click();
        });

        fileInput.addEventListener("change", (e) => {
            const file = e.target.files && e.target.files[0];
            handleFile(file);
        });

        clearBtn.addEventListener("click", () => {
            clearFile();
        });

        downloadBtn.addEventListener("click", () => {
            if (!uploadedFile) return;

            setStatus("Preparing download…", "info");
            fileStatusText.textContent = "Downloading…";
            progressTrack.style.display = "block";
            progressBar.style.width = "0%";

            const reader = new FileReader();

            reader.onloadstart = () => {
                progressBar.style.width = "5%";
            };

            reader.onprogress = (event) => {
                if (event.lengthComputable) {
                    const percent = Math.max(5, (event.loaded / event.total) * 100);
                    progressBar.style.width = percent.toFixed(1) + "%";
                }
            };

            reader.onload = (e) => {
                progressBar.style.width = "100%";
                const blob = new Blob([e.target.result], { type: "application/octet-stream" });
                const url = URL.createObjectURL(blob);
                const a = document.createElement("a");
                a.href = url;
                a.download = originalName || "schematic.bloxdschem";
                document.body.appendChild(a);
                a.click();
                document.body.removeChild(a);
                URL.revokeObjectURL(url);
                setStatus("Downloaded exact copy of " + (originalName || "your schematic") + ".", "success");
                fileStatusText.textContent = "Downloaded";
            };

            reader.onerror = () => {
                resetProgress();
                setStatus("Something went wrong while reading the file.", "error");
                fileStatusText.textContent = "Error";
                fileStatusPill.style.borderColor = "rgba(255, 77, 106, 0.9)";
            };

           reader.readAsArrayBuffer(uploadedFile);
        });
</script>
</body>
</html>
