<!doctype html>
<html lang="es">
<head>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Inventario de Computadoras </title>

  <style>
    :root{
      --accent:#2b6cb0;
      --bg:#f6f9fc;
      --card:#ffffff;
      --muted:#6b7280;
      --danger:#e53e3e;
      font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    }
    body{background:var(--bg); margin:0; padding:18px; color:#0f1724;}
    main{max-width:1100px;margin:0 auto;}
    .center{display:flex;align-items:center;justify-content:center;}
    .card{background:var(--card);border-radius:12px;padding:14px;box-shadow:0 6px 18px rgba(15,23,36,0.06);margin-bottom:14px;}
    header{display:flex;justify-content:space-between;align-items:center;margin-bottom:18px;}
    h1{margin:0;font-size:20px;}
    input, select, textarea, button{font-size:14px;padding:8px;border-radius:8px;border:1px solid #e6edf3;}
    input[type="date"]{padding:6px 8px;}
    .grid{display:grid;gap:10px;}
    .two{grid-template-columns:1fr 1fr;}
    .three{grid-template-columns:repeat(3,1fr);}
    .controls{display:flex;gap:8px;flex-wrap:wrap;}
    .btn{background:var(--accent);color:white;border:none;cursor:pointer;padding:8px 12px;border-radius:8px;}
    .btn.secondary{background:white;color:var(--accent);border:1px solid var(--accent);}
    .btn.danger{background:var(--danger);}
    table{width:100%;border-collapse:collapse;margin-top:8px;}
    th,td{padding:8px;border-bottom:1px solid #eef3f7;text-align:left;font-size:13px;}
    th{color:var(--muted);font-weight:600;}
    .muted{color:var(--muted);font-size:13px;}
    .small{font-size:12px;padding:6px 8px;border-radius:6px;}
    #videoPreview{width:100%;max-height:360px;border-radius:8px;background:#000;}
    .hidden{display:none !important;}
    footer{margin-top:18px;color:var(--muted);font-size:13px;}
    @media (max-width:760px){
      .three{grid-template-columns:1fr;}
      .two{grid-template-columns:1fr;}
      header{flex-direction:column;align-items:flex-start;gap:8px;}
    }

    /* LOGIN FULLSCREEN */
    #loginOverlay{
      position:fixed;inset:0;background:linear-gradient(180deg, rgba(15,23,36,0.06), rgba(15,23,36,0.03));
      display:flex;align-items:center;justify-content:center;z-index:9999;
    }
    #loginBox{width:100%;max-width:520px;}
    .login-title{font-size:18px;margin-bottom:6px;}
    .role-badge{background:var(--accent);color:white;padding:6px 10px;border-radius:999px;font-weight:600;}
    .muted-block{color:var(--muted);font-size:13px;margin-top:6px;}
  </style>

  <!-- ZXing para escaneo -->
  <script src="https://unpkg.com/@zxing/library@0.19.1/umd/index.min.js"></script>
  <!-- JsBarcode para generar imagen de código -->
  <script src="https://cdn.jsdelivr.net/npm/jsbarcode@3.11.5/dist/JsBarcode.all.min.js"></script>
</head>
<body>
  <main>
    <header class="card">
      <div>
        <h1>Inventario de Computadoras </h1>
        <div class="muted">Administra el inventario de equipos</div>
      </div>
      <div class="controls">
        <div id="roleLabel" class="muted">Sin sesión</div>
        <button class="btn secondary hidden" id="btnLogout">Cerrar sesión</button>
      </div>
    </header>

    <!-- Área principal: inicialmente oculta hasta login -->
    <div id="appArea" class="hidden">
      <!-- Controles superiores -->
      <div class="card" style="display:flex;justify-content:space-between;align-items:center;">
        <div style="display:flex;gap:8px;align-items:center;flex-wrap:wrap;">
          <button class="btn" id="btnShowForm">Nueva computadora</button>
          <button class="btn secondary" id="btnExport">Exportar JSON</button>
          <button class="btn secondary" id="btnExportExcel">Exportar Excel</button>
          <label class="btn secondary" style="cursor:pointer;">
            Importar JSON <input type="file" id="fileImport" accept=".json" style="display:none">
          </label>
        </div>
        <div style="display:flex;gap:8px;align-items:center;">
          <input id="searchInput" placeholder="Buscar (marca, modelo, serial, inventario o barcode)" style="min-width:260px;">
          <select id="filterEstado" class="small">
            <option value="">Todos</option>
            <option value="funcional">Funcional</option>
            <option value="no funcional">No funcional</option>
            <option value="mantenimiento">En mantenimiento</option>
          </select>
          <button class="btn secondary" id="btnClearSearch">Limpiar</button>
        </div>
      </div>

      <!-- Formulario -->
      <div id="formCard" class="card hidden">
        <div style="display:flex;justify-content:space-between;align-items:center;">
          <strong id="formTitle">Registrar nueva computadora</strong>
          <div class="muted">Usuario: <span id="currentUser">--</span></div>
        </div>

        <form id="pcForm" onsubmit="return false;" style="margin-top:10px">
          <div class="grid three">
            <input id="marca" placeholder="Marca (ej. HP)" />
            <input id="modelo" placeholder="Modelo (ej. ProBook 450)" />
            <input id="serial" placeholder="Número de serie" />
            <input id="fechaCompra" type="date" placeholder="Fecha de compra" />
            <input id="codigoInventario" placeholder="Código de inventario" />
            <select id="estado">
              <option value="funcional">Funcional</option>
              <option value="no funcional">No funcional</option>
              <option value="mantenimiento">En mantenimiento</option>
            </select>
            <input id="barcode" placeholder="Código de barras (scan o escribe)" />
            <input id="ubicacion" placeholder="Ubicación (aula, laboratorio...)" />
            <input id="otros" placeholder="Otros (ej. RAM, HDD/SSD, observaciones)" />
          </div>

          <div style="margin-top:8px;display:flex;gap:8px;align-items:center;">
            <button class="btn" id="btnSave">Guardar (Añadir)</button>
            <button class="btn secondary hidden" id="btnCancelEdit">Cancelar edición</button>
            <button class="btn secondary" type="button" id="btnScan">Escanear código</button>
            <button class="btn secondary" type="button" id="btnGenBarcode">Generar imagen del código</button>
            <canvas id="barcodeCanvas" class="hidden" style="height:48px;"></canvas>
          </div>
        </form>
        <!-- video preview para escaneo -->
        <div id="scannerBox" class="hidden" style="margin-top:12px;">
          <div class="muted">Vista de la cámara — apunta al código de barras</div>
          <video id="videoPreview" playsinline></video>
          <div style="margin-top:8px" class="controls">
            <button class="btn danger" id="stopScan">Detener escaneo</button>
          </div>
        </div>
      </div>
      <!-- Tabla de resultados -->
      <div class="card">
        <strong>Lista de computadoras</strong>
        <div id="tableWrap" style="overflow:auto;margin-top:8px;">
          <table id="pcTable">
            <thead>
              <tr>
                <th>Código inv.</th>
                <th>Marca / Modelo</th>
                <th>Serial</th>
                <th>Compra</th>
                <th>Estado</th>
                <th>Barcode</th>
                <th>Ubicación</th>
                <th>Acciones</th>
              </tr>
            </thead>
            <tbody id="pcBody"></tbody>
          </table>
        </div>
      </div>

      <footer class="muted card">
        <div>Prototipo local — datos guardados en tu navegador (localStorage).</div>
      </footer>
    </div>
  </main>

  <!-- LOGIN OVERLAY: aparece primero -->
  <div id="loginOverlay">
    <div id="loginBox" class="card">
      <div style="display:flex;justify-content:space-between;align-items:center;">
        <div>
          <div class="login-title">Iniciar sesión</div>
          <div class="muted-block">Elige cómo quieres entrar:</div>
        </div>
        <div class="role-badge">Acceso</div>
      </div>

      <div style="margin-top:12px" class="grid two">
        <button id="btnGuest" class="btn">Entrar como invitado</button>
        <button id="btnOpenAdmin" class="btn secondary">Entrar como administrador</button>
      </div>

      <div id="adminLogin" class="hidden" style="margin-top:12px;">
        <div class="muted-block">Introduce credenciales de administrador.</div>
        <div class="grid two" style="margin-top:8px;">
          <input id="loginUser" placeholder="Usuario" />
          <input id="loginPass" placeholder="Contraseña" type="password" />
        </div>
        <div style="margin-top:8px;display:flex;gap:8px;">
          <button id="btnLogin" class="btn">Entrar</button>
          <button id="btnCancelLogin" class="btn secondary">Cancelar</button>
        </div>
        <div id="loginError" class="muted-block" style="color:var(--danger);display:none;margin-top:8px;"></div>
      </div>
    </div>
  </div>

<script>
  /* ===========================
   EXPORTAR A EXCEL
   =========================== */
document.getElementById("btnExportExcel").addEventListener("click", ()=>{
  if(items.length === 0){
    alert("No hay datos para exportar.");
    return;
  }

  // Crear arreglo con formato de tabla
  const data = items.map(it => ({
    "Código Inventario": it.codigoInventario || "",
    "Marca": it.marca || "",
    "Modelo": it.modelo || "",
    "Serial": it.serial || "",
    "Fecha Compra": formatDate(it.fechaCompra) || "",
    "Estado": it.estado || "",
    "Código de Barras": it.barcode || "",
    "Ubicación": it.ubicacion || "",
    "Otros": it.otros || ""
  }));

  // Crear hoja
  const worksheet = XLSX.utils.json_to_sheet(data);
  const workbook = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(workbook, worksheet, "Inventario");

  // Descargar archivo
  XLSX.writeFile(workbook, "inventario_pc.xlsx");
});

/* ===========================
   CONFIG y ESTADO
   =========================== */
const STORAGE_KEY = "inventario_pc_v1";
let items = [];
let currentRole = null; // "admin" or "guest"
let currentUserName = null;
let editingId = null;

/* Credenciales internas (no mostradas en UI) */
const ADMIN_USERNAME = "admi";
const ADMIN_PASSWORD = "123";

/* ZXing reader */
let codeReader = null;
let selectedDeviceId = null;

/* ===========================
   CARGAR / GUARDAR
   =========================== */
function loadStore(){
  const raw = localStorage.getItem(STORAGE_KEY);
  if(raw){
    try { items = JSON.parse(raw) || []; } catch(e){ items = []; }
  } else items = [];
}
function saveStore(){
  localStorage.setItem(STORAGE_KEY, JSON.stringify(items));
}

/* ===========================
   UTIL
   =========================== */
function uid(){ return 'pc_' + Math.random().toString(36).slice(2,11); }
function formatDate(d){
  if(!d) return "";
  const dt = new Date(d);
  if(isNaN(dt)) return d;
  return dt.toLocaleDateString();
}

/* ===========================
   RENDER TABLA
   =========================== */
function renderTable(){
  const tbody = document.getElementById("pcBody");
  tbody.innerHTML = "";

  const search = (document.getElementById("searchInput").value || "").toLowerCase().trim();
  const estadoFilter = document.getElementById("filterEstado").value;

  const filtered = items.filter(it => {
    if(estadoFilter && it.estado !== estadoFilter) return false;
    if(!search) return true;
    const hay = [
      it.codigoInventario, it.marca, it.modelo, it.serial,
      (it.barcode||""), (it.ubicacion||""), (it.otros||"")
    ].join(" ").toLowerCase();
    return hay.includes(search);
  });

  if(filtered.length === 0){
    tbody.innerHTML = `<tr><td colspan="8" class="muted">No hay registros.</td></tr>`;
    return;
  }

  for(const it of filtered){
    const tr = document.createElement("tr");
    tr.innerHTML = `
      <td>${escapeHtml(it.codigoInventario||"")}</td>
      <td><strong>${escapeHtml(it.marca||"")}</strong><div class="muted">${escapeHtml(it.modelo||"")}</div></td>
      <td>${escapeHtml(it.serial||"")}</td>
      <td>${escapeHtml(formatDate(it.fechaCompra)||"")}</td>
      <td>${escapeHtml(it.estado||"")}</td>
      <td>${escapeHtml(it.barcode||"")}</td>
      <td>${escapeHtml(it.ubicacion||"")}</td>
      <td></td>
    `;
    const actionsCell = tr.querySelector("td:last-child");

    // VIEW button always available
    const btnView = document.createElement("button");
    btnView.className = "btn secondary small";
    btnView.textContent = "Ver";
    btnView.addEventListener("click", ()=> { openView(it.id); });

    actionsCell.appendChild(btnView);

    if(currentRole === "admin"){
      const btnEdit = document.createElement("button");
      btnEdit.className = "btn small";
      btnEdit.style.marginLeft = "6px";
      btnEdit.textContent = "Editar";
      btnEdit.addEventListener("click", ()=> { startEdit(it.id); });

      const btnDel = document.createElement("button");
      btnDel.className = "btn danger small";
      btnDel.style.marginLeft = "6px";
      btnDel.textContent = "Eliminar";
      btnDel.addEventListener("click", ()=> {
        if(confirm("¿Eliminar este registro?")) {
          items = items.filter(x=>x.id !== it.id);
          saveStore();
          renderTable();
        }
      });

      actionsCell.appendChild(btnEdit);
      actionsCell.appendChild(btnDel);
    }

    tbody.appendChild(tr);
  }
}

/* ===========================
   FORMULARIO
   =========================== */
function clearForm(){
  document.getElementById("marca").value = "";
  document.getElementById("modelo").value = "";
  document.getElementById("serial").value = "";
  document.getElementById("fechaCompra").value = "";
  document.getElementById("codigoInventario").value = "";
  document.getElementById("estado").value = "funcional";
  document.getElementById("barcode").value = "";
  document.getElementById("ubicacion").value = "";
  document.getElementById("otros").value = "";
  editingId = null;
  document.getElementById("btnSave").textContent = "Guardar (Añadir)";
  document.getElementById("btnCancelEdit").classList.add("hidden");
  document.getElementById("formTitle").textContent = "Registrar nueva computadora";
}

function startEdit(id){
  const it = items.find(x=>x.id===id);
  if(!it) return alert("Registro no encontrado.");
  editingId = id;
  document.getElementById("marca").value = it.marca||"";
  document.getElementById("modelo").value = it.modelo||"";
  document.getElementById("serial").value = it.serial||"";
  document.getElementById("fechaCompra").value = it.fechaCompra||"";
  document.getElementById("codigoInventario").value = it.codigoInventario||"";
  document.getElementById("estado").value = it.estado||"funcional";
  document.getElementById("barcode").value = it.barcode||"";
  document.getElementById("ubicacion").value = it.ubicacion||"";
  document.getElementById("otros").value = it.otros||"";
  document.getElementById("btnSave").textContent = "Guardar cambios";
  document.getElementById("btnCancelEdit").classList.remove("hidden");
  document.getElementById("formTitle").textContent = "Editar computadora";
  // show form if hidden
  document.getElementById("formCard").classList.remove("hidden");
  window.scrollTo({top:0,behavior:"smooth"});
}

function openView(id){
  const it = items.find(x=>x.id===id);
  if(!it) return;
  // Mostrar en modal simple usando prompt/alert para simplicidad
  const text = `
Código inventario: ${it.codigoInventario||""}
Marca: ${it.marca||""}
Modelo: ${it.modelo||""}
Serial: ${it.serial||""}
Fecha compra: ${formatDate(it.fechaCompra)||""}
Estado: ${it.estado||""}
Barcode: ${it.barcode||""}
Ubicación: ${it.ubicacion||""}
Otros: ${it.otros||""}
  `;
  alert(text);
}

/* ===========================
   GUARDAR / ACTUALIZAR
   =========================== */
document.getElementById("btnSave").addEventListener("click", ()=>{
  if(currentRole !== "admin"){
    alert("Solo el administrador puede guardar o editar registros.");
    return;
  }
  const marca = document.getElementById("marca").value.trim();
  const modelo = document.getElementById("modelo").value.trim();
  const serial = document.getElementById("serial").value.trim();
  const fechaCompra = document.getElementById("fechaCompra").value;
  const codigoInventario = document.getElementById("codigoInventario").value.trim();
  const estado = document.getElementById("estado").value;
  const barcode = document.getElementById("barcode").value.trim();
  const ubicacion = document.getElementById("ubicacion").value.trim();
  const otros = document.getElementById("otros").value.trim();

  if(!marca && !modelo){
    return alert("Añade al menos la marca o el modelo.");
  }

  if(editingId){
    // actualizar
    const it = items.find(x=>x.id===editingId);
    if(!it) return alert("Registro no encontrado.");
    it.marca = marca; it.modelo = modelo; it.serial = serial;
    it.fechaCompra = fechaCompra; it.codigoInventario = codigoInventario;
    it.estado = estado; it.barcode = barcode; it.ubicacion = ubicacion; it.otros = otros;
    it.updatedAt = new Date().toISOString();
    saveStore();
    clearForm();
    renderTable();
    document.getElementById("formCard").classList.add("hidden");
  } else {
    // nuevo
    const newItem = {
      id: uid(),
      marca, modelo, serial, fechaCompra, codigoInventario,
      estado, barcode, ubicacion, otros,
      createdAt: new Date().toISOString()
    };
    items.unshift(newItem);
    saveStore();
    clearForm();
    renderTable();
    document.getElementById("formCard").classList.add("hidden");
  }
});

/* Cancel edit */
document.getElementById("btnCancelEdit").addEventListener("click", ()=>{
  clearForm();
});

/* Mostrar/ocultar form */
document.getElementById("btnShowForm").addEventListener("click", ()=>{
  if(currentRole !== "admin"){
    alert("Solo el administrador puede añadir registros.");
    return;
  }
  document.getElementById("formCard").classList.toggle("hidden");
});

/* ===========================
   BUSCADOR / FILTRO
   =========================== */
document.getElementById("searchInput").addEventListener("input", renderTable);
document.getElementById("filterEstado").addEventListener("change", renderTable);
document.getElementById("btnClearSearch").addEventListener("click", ()=>{
  document.getElementById("searchInput").value = "";
  document.getElementById("filterEstado").value = "";
  renderTable();
});

/* ===========================
   EXPORT / IMPORT
   =========================== */
document.getElementById("btnExport").addEventListener("click", ()=>{
  const blob = new Blob([JSON.stringify(items, null, 2)], {type:"application/json"});
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url; a.download = "inventario_pc.json"; document.body.appendChild(a);
  a.click(); a.remove(); URL.revokeObjectURL(url);
});

document.getElementById("fileImport").addEventListener("change", (ev)=>{
  const f = ev.target.files[0];
  if(!f) return;
  const reader = new FileReader();
  reader.onload = (e)=>{
    try {
      const imported = JSON.parse(e.target.result);
      if(!Array.isArray(imported)) throw new Error("Formato inválido");
      // fusionar: evitar duplicados por id
      const existingIds = new Set(items.map(x=>x.id));
      for(const it of imported){
        if(!it.id) it.id = uid();
        if(!existingIds.has(it.id)){
          items.push(it);
        }
      }
      saveStore();
      renderTable();
      alert("Importación completada.");
    } catch(err){
      alert("Error importando JSON: " + err.message);
    }
  };
  reader.readAsText(f);
});

/* ===========================
   BARCODES: GENERAR IMAGEN
   =========================== */
document.getElementById("btnGenBarcode").addEventListener("click", ()=>{
  const value = document.getElementById("barcode").value.trim();
  if(!value) return alert("Escribe o escanea un código para generar la imagen.");
  const canvas = document.getElementById("barcodeCanvas");
  canvas.classList.remove("hidden");
  try {
    JsBarcode(canvas, value, {format: "code128", displayValue: true, height:48});
    // scroll to canvas
    canvas.scrollIntoView({behavior:"smooth"});
  } catch(e){
    alert("Error generando código: " + e.message);
  }
});

/* ===========================
   ESCANEAR CÓDIGO (ZXing)
   =========================== */
document.getElementById("btnScan").addEventListener("click", async ()=>{
  // mostrar UI de scanner
  document.getElementById("scannerBox").classList.remove("hidden");
  document.getElementById("videoPreview").classList.remove("hidden");
  try {
    if(!codeReader) codeReader = new ZXing.BrowserBarcodeReader();
    // pedir lista de dispositivos para seleccionar el trasero si hay
    const devices = await codeReader.listVideoInputDevices();
    if(devices && devices.length > 0){
      // preferir cámara trasera si aparece
      const back = devices.find(d => /back|rear|environment/gi.test(d.label));
      selectedDeviceId = (back && back.deviceId) || devices[0].deviceId;
      await codeReader.decodeFromVideoDevice(selectedDeviceId, 'videoPreview', (result, err) => {
        if(result){
          const code = result.getText();
          // poner en campo barcode y detener
          document.getElementById("barcode").value = code;
          stopScan();
          // buscar coincidencia y notificar
          const found = items.find(x => (x.barcode || "") === code || (x.codigoInventario||"") === code);
          if(found){
            alert("Código detectado. Registro existente cargado en la tabla. Puedes buscar por código inventario.");
            renderTable();
          } else {
            alert("Código detectado. Puedes guardar un nuevo registro con este código.");
          }
        }
      });
    } else {
      alert("No se encontraron cámaras disponibles.");
    }
  } catch(err){
    console.error(err);
    alert("Error iniciando la cámara: " + err.message);
    document.getElementById("scannerBox").classList.add("hidden");
  }
});

document.getElementById("stopScan").addEventListener("click", stopScan);

function stopScan(){
  if(codeReader){
    try { codeReader.reset(); } catch(e){ /*ignore*/ }
  }
  document.getElementById("scannerBox").classList.add("hidden");
}

/* ===========================
   LOGIN (overlay)
   =========================== */
const loginOverlay = document.getElementById("loginOverlay");
document.getElementById("btnGuest").addEventListener("click", ()=>{
  // invitado: no password
  currentRole = "guest";
  currentUserName = "Invitado";
  enterApp();
});

document.getElementById("btnOpenAdmin").addEventListener("click", ()=>{
  document.getElementById("adminLogin").classList.remove("hidden");
  document.getElementById("loginUser").value = "";
  document.getElementById("loginPass").value = "";
  document.getElementById("loginError").style.display = "none";
});

document.getElementById("btnCancelLogin").addEventListener("click", ()=>{
  document.getElementById("adminLogin").classList.add("hidden");
});

document.getElementById("btnLogin").addEventListener("click", ()=>{
  const u = (document.getElementById("loginUser").value || "").trim();
  const p = (document.getElementById("loginPass").value || "").trim();
  // validación local, sin mostrar credenciales en UI
  if(u === ADMIN_USERNAME && p === ADMIN_PASSWORD){
    currentRole = "admin";
    currentUserName = "Administrador";
    enterApp();
  } else {
    document.getElementById("loginError").textContent = "Credenciales incorrectas.";
    document.getElementById("loginError").style.display = "block";
  }
});

/* Logout */
document.getElementById("btnLogout").addEventListener("click", ()=>{
  if(codeReader) try { codeReader.reset(); } catch(e){}
  currentRole = null; currentUserName = null;
  // ocultar app y mostrar overlay
  document.getElementById("appArea").classList.add("hidden");
  document.getElementById("roleLabel").textContent = "Sin sesión";
  document.getElementById("btnLogout").classList.add("hidden");
  loginOverlay.style.display = "flex";
});

/* Al entrar: mostrar app y ajustar permisos */
function enterApp(){
  loginOverlay.style.display = "none";
  document.getElementById("appArea").classList.remove("hidden");
  document.getElementById("roleLabel").textContent = currentUserName + (currentRole === "admin" ? " (Administrador)" : " (Invitado)");
  document.getElementById("btnLogout").classList.remove("hidden");
  document.getElementById("currentUser").textContent = currentUserName;
  // Si invitado: ocultar botones de edición/añadir
  if(currentRole === "guest"){
    document.getElementById("btnShowForm").classList.add("hidden");
    // ocultar elementos editables (se hace por cheques antes de acciones)
  } else if(currentRole === "admin"){
    document.getElementById("btnShowForm").classList.remove("hidden");
  }
  renderTable();
}

/* ===========================
   INICIALIZACIÓN
   =========================== */
function init(){
  loadStore();
  clearForm();
  renderTable();
  // evitar mostrar credenciales en interfaz (no las mostramos)
  // overlay permanece visible hasta login
}
init();

/* ===========================
   UTIL: escape HTML
   =========================== */
function escapeHtml(str){
  if(!str && str !== 0) return "";
  return String(str).replace(/[&<>"']/g, (m)=>({
    "&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"
  })[m]);
}

/* Evitar pérdida de cámara si cierras la página */
window.addEventListener("beforeunload", ()=> {
  if(codeReader) try { codeReader.reset(); } catch(e){}
});
</script>
</body>
</html>
