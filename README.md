<!DOCTYPE html>
<html lang="es">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>DIVINCRI LA LIBERTAD - SISTEMA</title>

  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">

  <style>
    body {
      background: #e7f1e7;
      font-family: Arial;
    }

    .header {
      background: linear-gradient(90deg, #064b22, #0b5a2a);
      color: white;
      padding: 25px;
    }

    .box {
      border-left: 6px solid #075826;
    }

    .btn {
      background: #075826;
      color: white;
      padding: 10px 20px;
      border-radius: 8px;
      font-weight: bold;
    }

    .btn:hover {
      background: #043f1a;
    }

    .btn-red {
      background: #c53030;
      color: white;
      padding: 6px 10px;
      border-radius: 6px;
    }

    .btn-blue {
      background: #2b6cb0;
      color: white;
      padding: 6px 10px;
      border-radius: 6px;
    }

    table {
      width: 100%;
      border-collapse: collapse;
    }

    th {
      background: #075826;
      color: white;
      padding: 10px;
    }

    td {
      padding: 10px;
      border: 1px solid #ddd;
      text-align: center;
    }
  </style>
</head>

<body>

  <!-- HEADER -->
  <div class="header text-center">
    <h1 class="text-3xl font-bold">DIVINCRI - LA LIBERTAD</h1>
    <p>Sistema de Registro de Números</p>
  </div>

  <div class="p-6 max-w-6xl mx-auto">

    <!-- REGISTRO -->
    <div class="box bg-white p-5 rounded mb-6">

      <h2 class="text-2xl font-bold mb-4">Registrar Número</h2>

      <input id="numero" placeholder="Número Telefónico" class="border p-2 w-full mb-2 rounded">
      <input id="solicitante" placeholder="Solicitante" class="border p-2 w-full mb-2 rounded">
      <input id="observacion" placeholder="Observación" class="border p-2 w-full mb-2 rounded">
      <input id="vinculados" placeholder="Números Vinculados" class="border p-2 w-full mb-2 rounded">

      <button class="btn" onclick="guardar()">Guardar</button>
    </div>

    <!-- TABLA -->
    <div class="box bg-white p-5 rounded">

      <h2 class="text-2xl font-bold mb-4">Últimos Registros</h2>

      <table>
        <thead>
          <tr>
            <th>Número</th>
            <th>Solicitante</th>
            <th>Observación</th>
            <th>Vinculados</th>
            <th>Fecha</th>
            <th>Acciones</th>
          </tr>
        </thead>

        <tbody id="tabla"></tbody>
      </table>

    </div>

  </div>

  <script>
    let datos = JSON.parse(localStorage.getItem("datos")) || [];
    let editIndex = -1;

    render();

    function guardar() {
      let numero = document.getElementById("numero").value;
      let solicitante = document.getElementById("solicitante").value;
      let observacion = document.getElementById("observacion").value;
      let vinculados = document.getElementById("vinculados").value;

      if (numero === "") return alert("Ingrese número");

      let obj = {
        numero,
        solicitante,
        observacion,
        vinculados,
        fecha: new Date().toLocaleDateString()
      };

      if (editIndex === -1) {
        datos.push(obj);
      } else {
        datos[editIndex] = obj;
        editIndex = -1;
      }

      localStorage.setItem("datos", JSON.stringify(datos));
      limpiar();
      render();
    }

    function render() {
      let tabla = document.getElementById("tabla");
      tabla.innerHTML = "";

      datos.forEach((d, i) => {
        tabla.innerHTML += `
          <tr>
            <td>${d.numero}</td>
            <td>${d.solicitante}</td>
            <td>${d.observacion}</td>
            <td>${d.vinculados}</td>
            <td>${d.fecha}</td>
            <td>
              <button class="btn-blue" onclick="editar(${i})">Editar</button>
              <button class="btn-red" onclick="eliminar(${i})">Eliminar</button>
            </td>
          </tr>
        `;
      });
    }

    function eliminar(i) {
      if (confirm("¿Eliminar registro?")) {
        datos.splice(i, 1);
        localStorage.setItem("datos", JSON.stringify(datos));
        render();
      }
    }

    function editar(i) {
      let d = datos[i];

      document.getElementById("numero").value = d.numero;
      document.getElementById("solicitante").value = d.solicitante;
      document.getElementById("observacion").value = d.observacion;
      document.getElementById("vinculados").value = d.vinculados;

      editIndex = i;
    }

    function limpiar() {
      document.getElementById("numero").value = "";
      document.getElementById("solicitante").value = "";
      document.getElementById("observacion").value = "";
      document.getElementById("vinculados").value = "";
    }
  </script>

</body>
</html>
