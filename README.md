<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Control Esterilización</title>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
:root {
  --azul: #0078b4;
  --azul-claro: #eaf4fb;
  --gris: #f5f7fa;
}

body {
  font-family: 'Segoe UI', Arial;
  margin: 0;
  background: var(--gris);
}

header {
  background: var(--azul);
  color: white;
  padding: 15px;
  text-align: center;
  font-size: 20px;
}

.container {
  padding: 15px;
  max-width: 1000px;
  margin: auto;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px,1fr));
  gap: 10px;
}

.card {
  background: white;
  padding: 12px;
  border-radius: 8px;
}

input, select, button {
  padding: 6px;
  border-radius: 5px;
  border: 1px solid #ccc;
  font-size: 13px;
}

button {
  background: var(--azul);
  color: white;
  border: none;
  cursor: pointer;
}

button:hover {
  opacity: 0.9;
}

h2 {
  font-size: 16px;
  margin-bottom: 10px;
  color: var(--azul);
}

ul {
  padding-left: 15px;
  font-size: 13px;
}

.top-info {
  display:flex;
  justify-content: space-between;
  margin-bottom:10px;
  font-size:14px;
}
</style>
</head>

<body>

<header>Control Esterilización</header>

<div class="container">

<div class="top-info">
<div><strong>Fecha:</strong> <span id="fechaHoy"></span></div>
</div>

<!-- FORM -->
<div class="card">
<h2>Nuevo registro</h2>
<div class="grid">
<select id="nombre"></select>
<select id="zona">
<option>Zona Limpia</option>
<option>Zona Estéril</option>
</select>
<select id="temperatura"></select>
<select id="humedad"></select>
<button onclick="guardar()">Guardar</button>
</div>
</div>

<!-- FILTRO -->
<div class="card">
<h2>Búsqueda</h2>
<div class="grid">
<input type="date" id="fFecha">
<select id="fNombre"><option value="">Trabajador</option></select>
<select id="fTemp"><option value="">Temp</option></select>
<select id="fHum"><option value="">Hum</option></select>
<button onclick="filtrar()">Buscar</button>
</div>
</div>

<!-- HISTORIAL -->
<div class="card">
<h2>Historial</h2>
<ul id="lista"></ul>
</div>

<!-- GRAFICA -->
<div class="card">
<h2>Gráfica mensual</h2>
<div class="grid">
<input type="month" id="mesGrafica">
<button onclick="generarGrafica()">Ver gráfica</button>
</div>
<canvas id="grafica" style="margin-top:10px;"></canvas>
</div>

<!-- EXPORTAR -->
<div class="card">
<h2>Exportar Excel</h2>
<div class="grid">
<input type="month" id="mesExcel">
<button onclick="exportarExcel()">Descargar</button>
</div>
</div>

</div>

<script>
const trabajadores = ["Xisco","Jose Alberto","Bruno","Miguel Angel","Ximo","Enric","Sergio","Pablo"];
let registros = JSON.parse(localStorage.getItem("registros")) || [];

const fechaHoy = new Date().toLocaleDateString();
document.getElementById("fechaHoy").textContent = fechaHoy;

// cargar selects
trabajadores.forEach(t=>{
  nombre.innerHTML += `<option>${t}</option>`;
  fNombre.innerHTML += `<option>${t}</option>`;
});

for(let i=10;i<=30;i++){
  temperatura.innerHTML += `<option>${i}</option>`;
  fTemp.innerHTML += `<option>${i}</option>`;
}

for(let i=20;i<=55;i++){
  humedad.innerHTML += `<option>${i}</option>`;
  fHum.innerHTML += `<option>${i}</option>`;
}

// guardar
function guardar(){
  const zona = zonaSelect.value || zona.value;

  if(registros.some(r=>r.fecha===fechaHoy && r.zona===zona)){
    alert("Ya registrado hoy en esta zona");
    return;
  }

  registros.push({
    fecha: fechaHoy,
    nombre: nombre.value,
    zona: zona,
    temperatura: temperatura.value,
    humedad: humedad.value
  });

  localStorage.setItem("registros", JSON.stringify(registros));
  mostrar();
}

// mostrar
function mostrar(data=registros){
  lista.innerHTML="";
  data.forEach(r=>{
    lista.innerHTML += `<li>${r.fecha} | ${r.zona} | ${r.nombre} | ${r.temperatura}°C | ${r.humedad}%</li>`;
  });
}

// filtrar
function filtrar(){
  let data = registros;

  if(fFecha.value){
    const fecha = new Date(fFecha.value).toLocaleDateString();
    data = data.filter(r=>r.fecha===fecha);
  }

  if(fNombre.value) data = data.filter(r=>r.nombre===fNombre.value);
  if(fTemp.value) data = data.filter(r=>r.temperatura===fTemp.value);
  if(fHum.value) data = data.filter(r=>r.humedad===fHum.value);

  mostrar(data);
}

// grafica
let chart;
function generarGrafica(){
  const mes = mesGrafica.value;
  if(!mes) return;

  const datos = registros.filter(r=>{
    return new Date(r.fecha).toISOString().slice(0,7)===mes;
  });

  if(chart) chart.destroy();

  chart = new Chart(grafica,{
    type:'line',
    data:{
      labels: datos.map(r=>r.fecha),
      datasets:[
        {label:"Temp", data: datos.map(r=>r.temperatura)},
        {label:"Hum", data: datos.map(r=>r.humedad)}
      ]
    }
  });
}

// excel
function exportarExcel(){
  const mes = mesExcel.value;
  let csv = "Fecha,Zona,Trabajador,Temp,Hum\n";

  registros.forEach(r=>{
    if(new Date(r.fecha).toISOString().slice(0,7)===mes){
      csv += `${r.fecha},${r.zona},${r.nombre},${r.temperatura},${r.humedad}\n`;
    }
  });

  const blob = new Blob([csv]);
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = "registro.csv";
  a.click();
}

mostrar();
</script>

</body>
</html>
