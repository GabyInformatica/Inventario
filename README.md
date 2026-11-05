<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Inventario de Salones - Sistema Matriz</title>
    <style>
        :root {
            --bg: #ffffff;
            --surface: #f8fafc;
            --border: #cbd5e1;
            --text-main: #0f172a;
            --text-muted: #64748b;
            --primary: #991b1b;
            --primary-hover: #7f1d1d;
            --danger: #dc2626;
            --success: #16a34a;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            font-family: system-ui, -apple-system, sans-serif;
            background-color: var(--bg);
            color: var(--text-main);
            padding: 16px;
        }

        .container {
            max-width: 800px;
            margin: 0 auto;
        }

        .page-view {
            display: none;
        }

        .page-view.active {
            display: block;
        }

        .top-bar {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 16px;
            padding-bottom: 12px;
            border-bottom: 2px solid var(--border);
        }

        .top-bar h1 {
            font-size: 1.4rem;
            color: var(--primary);
        }

        .card-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
            gap: 16px;
            margin-top: 16px;
        }

        .salon-card {
            background: var(--surface);
            border: 1px solid var(--border);
            border-radius: 8px;
            padding: 16px;
            position: relative;
            box-shadow: 0 1px 3px rgba(0,0,0,0.05);
        }

        .qr-container-top {
            position: absolute;
            top: 12px;
            right: 12px;
            width: 70px;
            height: 70px;
            background: #fff;
            padding: 4px;
            border: 1px solid var(--border);
            border-radius: 4px;
        }

        .qr-container-top svg {
            width: 100%;
            height: 100%;
        }

        .card-title {
            font-size: 1.1rem;
            font-weight: bold;
            color: var(--primary);
            margin-bottom: 6px;
            padding-right: 80px;
        }

        .card-sub {
            font-size: 0.85rem;
            color: var(--text-muted);
            margin-bottom: 14px;
        }

        button, .btn {
            padding: 8px 12px;
            font-size: 0.9rem;
            font-weight: 600;
            border-radius: 6px;
            border: 1px solid var(--border);
            background: #ffffff;
            color: var(--text-main);
            cursor: pointer;
            display: inline-flex;
            align-items: center;
            justify-content: center;
            gap: 4px;
        }

        button:disabled {
            opacity: 0.4;
            cursor: not-allowed;
        }

        .btn-primary {
            background: var(--primary);
            color: #ffffff;
            border: none;
        }

        .btn-danger {
            background: var(--danger);
            color: white;
            border: none;
        }

        .btn-icon-square {
            width: 32px;
            height: 32px;
            padding: 0;
            font-weight: bold;
            font-size: 1.1rem;
        }

        .toolbar {
            background: var(--surface);
            padding: 12px;
            border-radius: 8px;
            border: 1px solid var(--border);
            margin-bottom: 16px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 8px;
        }

        .matrix-controls {
            display: flex;
            align-items: center;
            gap: 6px;
        }

        .table-container {
            overflow-x: auto;
            border: 1px solid var(--border);
            border-radius: 8px;
            background: #fff;
            margin-bottom: 16px;
        }

        /* TABLA BLANCA CON LETRAS 100% NEGRAS */
        table {
            width: 100%;
            border-collapse: collapse;
            text-align: center;
            font-size: 0.9rem;
            background-color: #ffffff;
        }

        th, td {
            padding: 10px;
            border: 1px solid var(--border);
            min-width: 90px;
            background-color: #ffffff;
            color: #0f172a !important;
        }

        th {
            font-weight: bold;
            background-color: #ffffff;
            color: #0f172a !important;
        }

        .header-cell-main {
            background-color: #ffffff;
            font-weight: bold;
            color: #0f172a !important;
        }

        td[contenteditable="true"]:focus, th[contenteditable="true"]:focus {
            background-color: #fffde7;
            color: #000000 !important;
            outline: 2px solid var(--primary);
        }
    </style>
</head>
<body>

    <div class="container">
        
        <!-- VISTA 1: DIRECTORIO DE SALONES -->
        <div id="view-directory" class="page-view active">
            <div class="top-bar">
                <h1>📦 Inventario General de Salones</h1>
                <button class="btn-primary" onclick="crearNuevoSalon()">+ Crear Salón</button>
            </div>
            <p style="color: var(--text-muted); font-size: 0.9rem;">Selecciona un salón para gestionar su tabla o consultar su ficha QR.</p>
            <div id="salonCardGrid" class="card-grid"></div>
        </div>

        <!-- VISTA 2: EDITOR MAESTRO -->
        <div id="view-editor" class="page-view">
            <div class="top-bar">
                <button onclick="volverAlDirectorio()">← Volver</button>
                <div style="display: flex; gap: 8px; align-items: center;">
                    <button id="btnToggleEdit" class="btn" onclick="toggleModoEdicion()">🔓 Activar Edición</button>
                    <button class="btn-primary" onclick="descargarImagenSelector('matrixTable')">📷 Descargar PNG</button>
                </div>
            </div>

            <h2 id="editorSalonTitle" style="color: var(--primary); margin-bottom: 12px;" contenteditable="false" onblur="guardarNombreSalon(this.innerText)">Nombre del Salón</h2>

            <!-- BARRA DE EDICIÓN CON DESHACER Y REHACER -->
            <div id="adminToolbar" class="toolbar" style="display: none;">
                <div class="matrix-controls">
                    <button id="btnUndo" class="btn" onclick="deshacer()" title="Retroceder (Deshacer)">↩️ Retroceder</button>
                    <button id="btnRedo" class="btn" onclick="rehacer()" title="Adelantar (Rehacer)">↪️ Adelantar</button>
                </div>

                <div class="matrix-controls">
                    <span style="font-weight: bold; font-size: 0.85rem; margin-right: 4px;">Filas:</span>
                    <button class="btn-icon-square" onclick="modificarFilas(1)">+</button>
                    <button class="btn-icon-square" onclick="modificarFilas(-1)">-</button>
                </div>

                <div class="matrix-controls">
                    <span style="font-weight: bold; font-size: 0.85rem; margin-right: 4px;">Columnas:</span>
                    <button class="btn-icon-square" onclick="modificarColumnas(1)">+</button>
                    <button class="btn-icon-square" onclick="modificarColumnas(-1)">-</button>
                </div>
            </div>

            <div class="table-container">
                <table id="matrixTable">
                    <thead>
                        <tr id="headerRow"></tr>
                    </thead>
                    <tbody id="tableBody"></tbody>
                </table>
            </div>
        </div>

        <!-- VISTA 3: FICHA PÚBLICA (QR) -->
        <div id="view-public" class="page-view">
            <div style="position: relative; padding-top: 10px; border-bottom: 2px solid var(--primary); margin-bottom: 16px;">
                <h1 id="publicSalonTitle" style="color: var(--primary); font-size: 1.6rem;">Salón</h1>
                <p style="color: var(--text-muted); font-size: 0.85rem; margin-bottom: 12px;">Planilla Oficial de Control de Inventario</p>
                <div id="publicQR" class="qr-container-top" style="top: 0;"></div>
            </div>

            <div class="table-container">
                <table id="publicMatrixTable">
                    <thead>
                        <tr id="publicHeaderRow"></tr>
                    </thead>
                    <tbody id="publicTableBody"></tbody>
                </table>
            </div>

            <div style="text-align: center; margin-top: 16px;">
                <button class="btn-primary" onclick="descargarImagenSelector('publicMatrixTable')">📷 Descargar Ficha como PNG</button>
            </div>
            
            <p style="text-align: center; color: var(--text-muted); font-size: 0.75rem; margin-top: 20px;">
                Documento de consulta pública sincronizado con el sistema central.
            </p>
        </div>

    </div>

    <script>
        let isEditMode = false;
        let currentSalonId = null;

        let historyStack = [];
        let historyStep = -1;

        let salones = JSON.parse(localStorage.getItem('matriz_salones_v8')) || [
            {
                id: "salon_1",
                nombre: "Aula 5° A",
                mainHeader: "Objetos",
                columnas: ["Cantidad"],
                filas: ["Sillas", "Mesas", "Pizarras", "Luminarias"],
                datos: [
                    [35],
                    [18],
                    [1],
                    [4]
                ]
            }
        ];

        function guardarBD(registrarEnHistorial = true) {
            localStorage.setItem('matriz_salones_v8', JSON.stringify(salones));
            if (registrarEnHistorial && currentSalonId) {
                guardarHistorial();
            }
        }

        function guardarHistorial() {
            const salonActual = salones.find(s => s.id === currentSalonId);
            if (!salonActual) return;
            historyStack = historyStack.slice(0, historyStep + 1);
            historyStack.push(JSON.stringify(salonActual));
            historyStep++;
            actualizarBotonesHistorial();
        }

        function deshacer() {
            if (historyStep > 0) {
                historyStep--;
                reemplazarEstadoSalon(JSON.parse(historyStack[historyStep]));
            }
        }

        function rehacer() {
            if (historyStep < historyStack.length - 1) {
                historyStep++;
                reemplazarEstadoSalon(JSON.parse(historyStack[historyStep]));
            }
        }

        function reemplazarEstadoSalon(estado) {
            const idx = salones.findIndex(s => s.id === estado.id);
            if (idx !== -1) {
                salones[idx] = estado;
                guardarBD(false);
                document.getElementById('editorSalonTitle').innerText = estado.nombre;
                renderizarTablaEditor();
                actualizarBotonesHistorial();
            }
        }

        function actualizarBotonesHistorial() {
            const btnUndo = document.getElementById('btnUndo');
            const btnRedo = document.getElementById('btnRedo');
            if (btnUndo && btnRedo) {
                btnUndo.disabled = (historyStep <= 0);
                btnRedo.disabled = (historyStep >= historyStack.length - 1);
            }
        }

        function generarQRSVG(texto) {
            const hash = Array.from(texto).reduce((acc, char) => acc + char.charCodeAt(0), 0);
            return `
                <svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                    <rect width="100" height="100" fill="white"/>
                    <rect x="5" y="5" width="30" height="30" fill="#991b1b"/>
                    <rect x="10" y="10" width="20" height="20" fill="white"/>
                    <rect x="15" y="15" width="10" height="10" fill="#991b1b"/>
                    <rect x="65" y="5" width="30" height="30" fill="#991b1b"/>
                    <rect x="70" y="10" width="20" height="20" fill="white"/>
                    <rect x="75" y="15" width="10" height="10" fill="#991b1b"/>
                    <rect x="5" y="65" width="30" height="30" fill="#991b1b"/>
                    <rect x="10" y="70" width="20" height="20" fill="white"/>
                    <rect x="15" y="75" width="10" height="10" fill="#991b1b"/>
                    <rect x="${40 + (hash % 15)}" y="40" width="15" height="15" fill="#0f172a"/>
                    <rect x="${60 - (hash % 10)}" y="65" width="12" height="12" fill="#0f172a"/>
                    <rect x="45" y="${65 + (hash % 10)}" width="10" height="20" fill="#0f172a"/>
                </svg>
            `;
        }

        function evaluarRuta() {
            const hash = window.location.hash;
            document.querySelectorAll('.page-view').forEach(v => v.classList.remove('active'));

            if (hash.startsWith('#ver=')) {
                renderizarVistaPublica(hash.replace('#ver=', ''));
            } else if (hash.startsWith('#editar=')) {
                abrirEditor(hash.replace('#editar=', ''));
            } else {
                renderizarDirectorio();
                document.getElementById('view-directory').classList.add('active');
            }
        }

        window.addEventListener('hashchange', evaluarRuta);
        window.addEventListener('load', evaluarRuta);

        function renderizarDirectorio() {
            const grid = document.getElementById('salonCardGrid');
            grid.innerHTML = '';

            salones.forEach(s => {
                const urlPublica = `${window.location.origin}${window.location.pathname}#ver=${s.id}`;
                const card = document.createElement('div');
                card.className = 'salon-card';
                card.innerHTML = `
                    <div class="qr-container-top">${generarQRSVG(urlPublica)}</div>
                    <div class="card-title">${s.nombre}</div>
                    <div class="card-sub">${s.filas.length} filas × ${s.columnas.length} columna</div>
                    <div style="display: flex; gap: 8px; margin-top: 10px;">
                        <button class="btn-primary" onclick="location.hash='#editar=${s.id}'">Gestionar Tabla</button>
                        <button onclick="eliminarSalon('${s.id}')" class="btn-danger" style="padding: 6px 8px;">🗑️</button>
                    </div>
                `;
                grid.appendChild(card);
            });
        }

        function crearNuevoSalon() {
            const nuevoId = 'salon_' + Date.now();
            salones.push({
                id: nuevoId,
                nombre: "Nuevo Salón",
                mainHeader: "Objetos",
                columnas: ["Cantidad"],
                filas: ["Fila 1", "Fila 2", "Fila 3", "Fila 4"],
                datos: [[0], [0], [0], [0]]
            });
            guardarBD(false);
            location.hash = `#editar=${nuevoId}`;
        }

        function eliminarSalon(id) {
            if (confirm("¿Deseas eliminar este salón?")) {
                salones = salones.filter(s => s.id !== id);
                guardarBD(false);
                renderizarDirectorio();
            }
        }

        function abrirEditor(id) {
            const salon = salones.find(s => s.id === id);
            if (!salon) { location.hash = ''; return; }
            currentSalonId = id;
            historyStack = [JSON.stringify(salon)];
            historyStep = 0;

            document.getElementById('view-editor').classList.add('active');
            document.getElementById('editorSalonTitle').innerText = salon.nombre;
            renderizarTablaEditor();
            actualizarBotonesHistorial();
        }

        function volverAlDirectorio() {
            isEditMode = false;
            actualizarEstadoEdicion();
            location.hash = '';
        }

        function toggleModoEdicion() {
            isEditMode = !isEditMode;
            actualizarEstadoEdicion();
        }

        function actualizarEstadoEdicion() {
            const btnToggle = document.getElementById('btnToggleEdit');
            const adminToolbar = document.getElementById('adminToolbar');
            const titleEl = document.getElementById('editorSalonTitle');

            if (isEditMode) {
                btnToggle.innerText = "🔒 Finalizar Edición";
                btnToggle.classList.add('btn-primary');
                adminToolbar.style.display = 'flex';
                titleEl.setAttribute('contenteditable', 'true');
            } else {
                btnToggle.innerText = "🔓 Activar Edición";
                btnToggle.classList.remove('btn-primary');
                adminToolbar.style.display = 'none';
                titleEl.setAttribute('contenteditable', 'false');
            }
            renderizarTablaEditor();
        }

        function guardarNombreSalon(val) {
            const salon = salones.find(s => s.id === currentSalonId);
            if (salon) { salon.nombre = val.trim() || "Salón"; guardarBD(); }
        }

        function renderizarTablaEditor() {
            const salon = salones.find(s => s.id === currentSalonId);
            if (!salon) return;

            const headerRow = document.getElementById('headerRow');
            const tableBody = document.getElementById('tableBody');

            headerRow.innerHTML = `
                <th class="header-cell-main" contenteditable="${isEditMode}" onblur="actualizarMainHeader(this.innerText)">
                    ${salon.mainHeader || 'Objetos'}
                </th>
            `;
            salon.columnas.forEach((col, cIdx) => {
                headerRow.innerHTML += `<th contenteditable="${isEditMode}" onblur="actualizarColumna(${cIdx}, this.innerText)">${col}</th>`;
            });

            tableBody.innerHTML = '';
            salon.filas.forEach((fila, fIdx) => {
                let tr = document.createElement('tr');
                tr.innerHTML = `<td style="font-weight: bold; text-align: left;" contenteditable="${isEditMode}" onblur="actualizarFila(${fIdx}, this.innerText)">${fila}</td>`;
                salon.columnas.forEach((_, cIdx) => {
                    let val = (salon.datos[fIdx] && salon.datos[fIdx][cIdx] !== undefined) ? salon.datos[fIdx][cIdx] : 0;
                    tr.innerHTML += `<td contenteditable="${isEditMode}" onblur="actualizarDato(${fIdx}, ${cIdx}, this.innerText)">${val}</td>`;
                });
                tableBody.appendChild(tr);
            });
        }

        function actualizarMainHeader(val) {
            const s = salones.find(s => s.id === currentSalonId);
            if (s) { s.mainHeader = val.trim() || 'Objetos'; guardarBD(); }
        }

        function actualizarColumna(idx, val) {
            const s = salones.find(s => s.id === currentSalonId);
            if (s) { s.columnas[idx] = val.trim() || 'Columna'; guardarBD(); }
        }

        function actualizarFila(idx, val) {
            const s = salones.find(s => s.id === currentSalonId);
            if (s) { s.filas[idx] = val.trim() || 'Fila'; guardarBD(); }
        }

        function actualizarDato(fIdx, cIdx, val) {
            const s = salones.find(s => s.id === currentSalonId);
            if (s) {
                if (!s.datos[fIdx]) s.datos[fIdx] = [];
                s.datos[fIdx][cIdx] = parseInt(val.trim()) || 0;
                guardarBD();
            }
        }

        function modificarFilas(delta) {
            const s = salones.find(s => s.id === currentSalonId);
            if (!s) return;
            if (delta > 0) {
                s.filas.push(`Fila ${s.filas.length + 1}`);
                s.datos.push(new Array(s.columnas.length).fill(0));
            } else if (delta < 0 && s.filas.length > 1) {
                s.filas.pop(); s.datos.pop();
            }
            guardarBD(); renderizarTablaEditor();
        }

        function modificarColumnas(delta) {
            const s = salones.find(s => s.id === currentSalonId);
            if (!s) return;
            if (delta > 0) {
                s.columnas.push(`Col ${s.columnas.length + 1}`);
                s.datos.forEach(row => row.push(0));
            } else if (delta < 0 && s.columnas.length > 1) {
                s.columnas.pop();
                s.datos.forEach(row => row.pop());
            }
            guardarBD(); renderizarTablaEditor();
        }

        function renderizarVistaPublica(id) {
            const salon = salones.find(s => s.id === id);
            if (!salon) { alert("Salón no encontrado"); location.hash = ''; return; }

            document.getElementById('view-public').classList.add('active');
            document.getElementById('publicSalonTitle').innerText = salon.nombre;
            document.getElementById('publicQR').innerHTML = generarQRSVG(window.location.href);

            const headerRow = document.getElementById('publicHeaderRow');
            const tableBody = document.getElementById('publicTableBody');

            headerRow.innerHTML = `<th class="header-cell-main">${salon.mainHeader || 'Objetos'}</th>`;
            salon.columnas.forEach(col => headerRow.innerHTML += `<th>${col}</th>`);

            tableBody.innerHTML = '';
            salon.filas.forEach((fila, fIdx) => {
                let tr = document.createElement('tr');
                tr.innerHTML = `<td style="font-weight: bold; text-align: left;">${fila}</td>`;
                salon.columnas.forEach((_, cIdx) => {
                    let val = (salon.datos[fIdx] && salon.datos[fIdx][cIdx] !== undefined) ? salon.datos[fIdx][cIdx] : 0;
                    tr.innerHTML += `<td>${val}</td>`;
                });
                tableBody.appendChild(tr);
            });
        }

        // EXPORTACIÓN A ARCHIVO BINARIO REAL (CON SELECTOR DE CARPETAS DEL SISTEMA)
        function descargarImagenSelector(tableId) {
            const table = document.getElementById(tableId);
            if (!table) return;

            const salon = salones.find(s => s.id === currentSalonId) || { nombre: "Inventario" };
            const nombreArchivo = `Inventario_${salon.nombre.replace(/[^a-zA-Z0-9]/g, '_')}.png`;

            const rows = Array.from(table.querySelectorAll('tr'));
            const cellHeight = 44;
            const cellWidth = 150;
            const padding = 20;

            const numCols = rows[0].children.length;
            const canvasWidth = (numCols * cellWidth) + (padding * 2);
            const canvasHeight = (rows.length * cellHeight) + (padding * 2);

            const canvas = document.createElement('canvas');
            canvas.width = canvasWidth;
            canvas.height = canvasHeight;
            const ctx = canvas.getContext('2d');

            ctx.fillStyle = '#ffffff';
            ctx.fillRect(0, 0, canvasWidth, canvasHeight);

            rows.forEach((row, rIdx) => {
                const cells = Array.from(row.children);
                cells.forEach((cell, cIdx) => {
                    const x = padding + (cIdx * cellWidth);
                    const y = padding + (rIdx * cellHeight);

                    ctx.strokeStyle = '#cbd5e1';
                    ctx.lineWidth = 1;
                    ctx.strokeRect(x, y, cellWidth, cellHeight);

                    ctx.font = (rIdx === 0 || cIdx === 0) ? 'bold 14px system-ui' : '14px system-ui';
                    ctx.fillStyle = '#0f172a';
                    ctx.textAlign = 'center';
                    ctx.textBaseline = 'middle';
                    
                    ctx.fillText(cell.innerText.trim(), x + (cellWidth / 2), y + (cellHeight / 2));
                });
            });

            // Convertir a Blob binario y ejecutar descarga nativa sin Base64
            canvas.toBlob(async (blob) => {
                if (!blob) return;

                // Intentar usar la API moderna File System Access (para elegir ubicación)
                if (window.showSaveFilePicker) {
                    try {
                        const handle = await window.showSaveFilePicker({
                            suggestedName: nombreArchivo,
                            types: [{
                                description: 'Imagen PNG',
                                accept: { 'image/png': ['.png'] }
                            }]
                        });
                        const writable = await handle.createWritable();
                        await writable.write(blob);
                        await writable.close();
                        return;
                    } catch (err) {
                        // Si el usuario cancela la ventana de guardado
                        if (err.name === 'AbortError') return;
                    }
                }

                // Fallback con ObjectURL binario para explorador de archivos móvil
                const url = URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.href = url;
                a.download = nombreArchivo;
                document.body.appendChild(a);
                a.click();
                document.body.removeChild(a);
                setTimeout(() => URL.revokeObjectURL(url), 1000);
            }, 'image/png');
        }
    </script>
</body>
</html>
