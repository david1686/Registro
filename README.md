<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Registro de Estudiantes</title>

  <!-- Bootstrap CSS -->
  <link
    href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
    rel="stylesheet"
  />

  <!-- Firebase -->
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-firestore-compat.js"></script>

  <!-- jsPDF para exportar a PDF -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

  <!-- SheetJS para exportar a Excel -->
  <script src="https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js"></script>
</head>
<body class="bg-light">
  <div class="container py-4">
    <h1 class="text-center mb-4">Registro de Estudiantes</h1>

    <form id="formulario" class="row g-3">
      <div class="col-md-6">
        <label class="form-label">Primer Apellido</label>
        <input type="text" id="apellido1" class="form-control" required>
      </div>
      <div class="col-md-6">
        <label class="form-label">Segundo Apellido</label>
        <input type="text" id="apellido2" class="form-control" required>
      </div>
      <div class="col-md-6">
        <label class="form-label">Nombre</label>
        <input type="text" id="nombre" class="form-control" required>
      </div>
      <div class="col-md-6">
        <label class="form-label">Sección</label>
        <input type="text" id="seccion" class="form-control" required>
      </div>
      <div class="col-md-6">
        <label class="form-label">Materia</label>
        <input type="text" id="materia" class="form-control" required>
      </div>
      <div class="col-md-6">
        <label class="form-label">Docente</label>
        <input type="text" id="docente" class="form-control" required>
      </div>
      <div class="col-md-6">
        <label class="form-label">Número de ausencias</label>
        <input type="number" id="ausencias" class="form-control" required>
      </div>
      <div class="col-md-6">
        <label class="form-label">Factores de riesgo</label>
        <textarea id="riesgo" class="form-control" required></textarea>
      </div>
      <div class="col-md-12">
        <label class="form-label">Acciones realizadas</label>
        <textarea id="acciones" class="form-control" required></textarea>
      </div>
      <div class="col-12 text-end">
        <button type="submit" class="btn btn-primary">Registrar</button>
      </div>
    </form>

    <hr class="my-4">

    <div class="d-flex justify-content-between align-items-center mb-3">
      <input type="text" id="search" class="form-control w-50" placeholder="Buscar por sección..." oninput="filtrarEstudiantes()">
      <div class="ms-3">
        <button onclick="exportarPDF()" class="btn btn-outline-danger me-2">Exportar a PDF</button>
        <button onclick="exportarExcel()" class="btn btn-outline-success">Exportar a Excel</button>
      </div>
    </div>

    <div id="lista" class="row"></div>
  </div>

  <script>
    const firebaseConfig = {
      apiKey: "AIzaSyBKluxJeTIlO17uAYkrIr5JoTjLiovtDyM",
      authDomain: "registro-a9fd3.firebaseapp.com",
      projectId: "registro-a9fd3",
      storageBucket: "registro-a9fd3.appspot.com",
      messagingSenderId: "399328760047",
      appId: "1:399328760047:web:7c5c567fbefead86becb1a",
      measurementId: "G-DLZ74RJWPX"
    };

    firebase.initializeApp(firebaseConfig);
    const db = firebase.firestore();

    const form = document.getElementById("formulario");
    const lista = document.getElementById("lista");
    const searchInput = document.getElementById("search");
    let idActual = null;
    let estudiantesFiltrados = [];

    form.addEventListener("submit", async (e) => {
      e.preventDefault();

      const data = {
        apellido1: document.getElementById("apellido1").value,
        apellido2: document.getElementById("apellido2").value,
        nombre: document.getElementById("nombre").value,
        seccion: document.getElementById("seccion").value,
        materia: document.getElementById("materia").value,
        docente: document.getElementById("docente").value,
        ausencias: parseInt(document.getElementById("ausencias").value),
        riesgo: document.getElementById("riesgo").value,
        acciones: document.getElementById("acciones").value
      };

      try {
        if (idActual) {
          await db.collection("estudiantes").doc(idActual).update(data);
          idActual = null;
        } else {
          await db.collection("estudiantes").add(data);
        }
        form.reset();
      } catch (error) {
        alert("Error al guardar: " + error.message);
      }
    });

    function renderLista(estudiantes) {
      lista.innerHTML = "";
      estudiantes.forEach(est => {
        const col = document.createElement("div");
        col.className = "col-md-6 mb-3";

        col.innerHTML = `
          <div class="card shadow-sm">
            <div class="card-body">
              <h5 class="card-title">${est.nombre} ${est.apellido1} ${est.apellido2}</h5>
              <p class="card-text">
                <strong>Sección:</strong> ${est.seccion} <br>
                <strong>Materia:</strong> ${est.materia} <br>
                <strong>Docente:</strong> ${est.docente} <br>
                <strong>Ausencias:</strong> ${est.ausencias} <br>
                <strong>Riesgo:</strong> ${est.riesgo} <br>
                <strong>Acciones:</strong> ${est.acciones}
              </p>
              <button onclick="editarEstudiante('${est.id}')" class="btn btn-sm btn-warning me-2">Editar</button>
              <button onclick="eliminarEstudiante('${est.id}')" class="btn btn-sm btn-danger">Eliminar</button>
            </div>
          </div>
        `;

        lista.appendChild(col);
      });
    }

    async function filtrarEstudiantes() {
      const seccion = searchInput.value.toLowerCase();
      const snapshot = await db.collection("estudiantes").orderBy("nombre").get();
      estudiantesFiltrados = [];

      snapshot.forEach(doc => {
        const data = doc.data();
        if (data.seccion.toLowerCase().includes(seccion)) {
          estudiantesFiltrados.push({ ...data, id: doc.id });
        }
      });

      renderLista(estudiantesFiltrados);
    }

    db.collection("estudiantes").orderBy("nombre").onSnapshot(snapshot => {
      estudiantesFiltrados = [];
      snapshot.forEach(doc => {
        estudiantesFiltrados.push({ ...doc.data(), id: doc.id });
      });
      renderLista(estudiantesFiltrados);
    });

    function editarEstudiante(id) {
      const est = estudiantesFiltrados.find(e => e.id === id);
      if (est) {
        document.getElementById("apellido1").value = est.apellido1;
        document.getElementById("apellido2").value = est.apellido2;
        document.getElementById("nombre").value = est.nombre;
        document.getElementById("seccion").value = est.seccion;
        document.getElementById("materia").value = est.materia;
        document.getElementById("docente").value = est.docente;
        document.getElementById("ausencias").value = est.ausencias;
        document.getElementById("riesgo").value = est.riesgo;
        document.getElementById("acciones").value = est.acciones;
        idActual = id;
      }
    }

    function eliminarEstudiante(id) {
      if (confirm("¿Deseás eliminar este registro?")) {
        db.collection("estudiantes").doc(id).delete().catch(err => {
          alert("Error al eliminar: " + err.message);
        });
      }
    }

    async function exportarPDF() {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF();
      let y = 10;

      estudiantesFiltrados.forEach((est, i) => {
        doc.text(`${i + 1}. ${est.nombre} ${est.apellido1} ${est.apellido2}`, 10, y);
        y += 7;
        doc.text(`   Sección: ${est.seccion} | Materia: ${est.materia}`, 10, y);
        y += 7;
        doc.text(`   Docente: ${est.docente} | Ausencias: ${est.ausencias}`, 10, y);
        y += 7;
        doc.text(`   Riesgo: ${est.riesgo}`, 10, y);
        y += 7;
        doc.text(`   Acciones: ${est.acciones}`, 10, y);
        y += 10;

        if (y > 270) {
          doc.addPage();
          y = 10;
        }
      });

      doc.save("estudiantes.pdf");
    }

    async function exportarExcel() {
      const data = estudiantesFiltrados.map(est => ({
        Nombre: est.nombre,
        "Primer Apellido": est.apellido1,
        "Segundo Apellido": est.apellido2,
        Sección: est.seccion,
        Materia: est.materia,
        Docente: est.docente,
        Ausencias: est.ausencias,
        "Factores de riesgo": est.riesgo,
        "Acciones realizadas": est.acciones
      }));

      const worksheet = XLSX.utils.json_to_sheet(data);
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, "Estudiantes");
      XLSX.writeFile(workbook, "estudiantes.xlsx");
    }
  </script>
</body>
</html>
