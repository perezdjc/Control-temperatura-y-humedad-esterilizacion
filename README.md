<html lang="es">
<head>
<meta charset="UTF-8">
<title>Registro Esterilización</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
<style>
body { font-family: Arial; background:#eaf2fb; }
h1 { color:#0056a6; text-align:center; }
.container { max-width:900px; margin:auto; background:white; padding:20px; border-radius:10px; }
label { display:block; margin-top:10px; }
select, input { width:100%; padding:8px; margin-top:5px; }
button { margin-top:15px; padding:10px; background:#0056a6; color:white; border:none; cursor:pointer; }
button:hover { background:#003f7f; }
table { width:100%; border-collapse: collapse; margin-top:20px; }
th, td { border:1px solid #ccc; padding:8px; text-align:center; }
th { background:#0056a6; color:white; }
</style>
</head>
<body>
<div class="container">
<h1>Registro de Temperatura y Humedad</h1>

<label>Nombre trabajador</label>
<select id="trabajador">
<option>Xisco</option>
<option>Jose Alberto</option>
<option>Bruno</option>
<option>Miguel Angel</option>
<option>Ximo</option>
<option>Enric</option>
<option>Sergio</option>
<option>Pablo</option>
</select>

<label>Zona</label>
<select id="zona">
<option>Zona Limpia</option>
<option>Zona Esteril</option>
</select>

<label>Temperatura (10-30)</label>
<select id="temperatura"></select>

<label>Humedad (20-55)</label>
<select id="humedad"></select>

<label>Fecha</label>
<input type="date" id="fecha">

<label>Hora</label>
<input type="time" id="hora">

<button onclick="guardar()">Guardar Registro</button>
<button onclick="exportarExcel()">Exportar Excel</button>

<input type="text" id="buscar" placeholder="Buscar por trabajador, fecha, temperatura o humedad" onkeyup="filtrar()">

<table id="tabla">
<thead>
<tr>
<th>Fecha</th>
<th>Hora</th>
<th>Trabajador</th>
<th>Zona</th>
<th>Temperatura</th>
<th>Humedad</th>
</tr>
</thead>
<tbody></tbody>
</table>

<canvas id="grafica"></canvas>

</div>

<script>
// Rellenar select
for(let i=10;i<=30;i++) document.getElementById('temperatura').innerHTML += `<option>${i}</option>`;
for(let i=20;i<=55;i++) document.getElementById('humedad').innerHTML += `<option>${i}</option>`;

let datos = JSON.parse(localStorage.getItem('datos')) || [];

function guardar(){
 let registro = {
  trabajador: trabajador.value,
  zona: zona.value,
  temperatura: temperatura.value,
  humedad: humedad.value,
  fecha: fecha.value,
  hora: hora.value
 };
 datos.push(registro);
 localStorage.setItem('datos', JSON.stringify(datos));
 actualizarTabla();
 actualizarGrafica();
}

function actualizarTabla(){
 let tbody = document.querySelector('#tabla tbody');
 tbody.innerHTML = '';
 datos.forEach(d=>{
  tbody.innerHTML += `<tr>
  <td>${d.fecha}</td>
  <td>${d.hora}</td>
  <td>${d.trabajador}</td>
  <td>${d.zona}</td>
  <td>${d.temperatura}</td>
  <td>${d.humedad}</td>
  </tr>`;
 });
}

function filtrar(){
 let val = buscar.value.toLowerCase();
 let tbody = document.querySelector('#tabla tbody');
 tbody.innerHTML = '';
 datos.filter(d => JSON.stringify(d).toLowerCase().includes(val)).forEach(d=>{
  tbody.innerHTML += `<tr>
  <td>${d.fecha}</td>
  <td>${d.hora}</td>
  <td>${d.trabajador}</td>
  <td>${d.zona}</td>
  <td>${d.temperatura}</td>
  <td>${d.humedad}</td>
  </tr>`;
 });
}

function exportarExcel(){
 let ws = XLSX.utils.json_to_sheet(datos);
 let wb = XLSX.utils.book_new();
 XLSX.utils.book_append_sheet(wb, ws, "Registros");
 XLSX.writeFile(wb, "registros.xlsx");
}

function actualizarGrafica(){
 let ctx = document.getElementById('grafica').getContext('2d');
 let fechas = datos.map(d=>d.fecha);
 let temp = datos.map(d=>d.temperatura);
 let hum = datos.map(d=>d.humedad);

 if(window.grafica) window.grafica.destroy();

 window.grafica = new Chart(ctx, {
  type: 'line',
  data: {
   labels: fechas,
   datasets: [
    { label:'Temperatura', data: temp },
    { label:'Humedad', data: hum }
   ]
  }
 });
}

actualizarTabla();
actualizarGrafica();
</script>

</body>
</html>
