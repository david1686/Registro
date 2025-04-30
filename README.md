<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Registro de Estudiantes</title>

  <!-- Bootstrap 5 -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">

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
    <h1 class="mb-4 text-center">Registro de Estudiantes</h1>

    <form id="formulario" class="bg-white p-4 rounded shadow">
      <div class="row g-3">
        <div class="col-md-4">
          <label class="form-label">Primer Apellido:</label>
          <input type="text" id="apellido1" class="form-control" required>
        </div>
        <div class="col-md-4">
          <label class="form-label">Segundo Apellido:</label>
          <input type="text" id="apellido2" class="form-control" required>
        </div>
        <div class="col-md-4">
          <label class="form-label">Nombre:</label>
          <input type="text" id="nombre" class="form-control" required>
        </div>
        <div class="col-md-4">
          <label class="form-label">Sección:</label>
          <input type="text" id="seccion" class="form-control" required>
        </div>
        <div class="col-md-4">
          <label class="form-label">Materia:</label>
          <input type="text" id="materia" class="form-control" required>
        </div>
        <div class="col-md-4">
          <label class="form-label">Docente:</label>
          <input type="text" id="docente" class="form-control" required>
        </div>
        <div class="col-md-4">
          <label class="form-label">Número de ausencias:</label>
          <input type="number" id="ausencias" class="form-control" required>
        </div>
        <div class="col-md-12">
          <label class="form-label">Factores de riesgo:</label>
          <textarea id="riesgo" class="form-control" required></textarea>
        </div>
        <div class="col-md-12">
          <label class="form-label">Acciones realizadas:</label>
          <textarea id="acciones" class="form-control" required></textarea>
        </div>
      </div>
      <div class="text-end mt-4">
        <button type="submit" class="btn btn-primary">Registrar</button>
      </div>
    </form>

    <div class="mt-5">
      <div class="row align-items-center">
        <div class="col-md-6">
          <label for="search" class="form-label">Buscar por Sección:</label>
          <input type="text" id="search" class="form-control" placeholder="Ej: 10-2" oninput="filtrarEstudiantes()">
        </div>
        <div class="col-md-6 text-md-end mt-3 mt-md-0">
          <button class="btn btn-outline-success me-2" onclick="exportarPDF()">Exportar a PDF</button>
          <button class="btn btn-outline-success" onclick="exportarExcel()">Exportar a Excel</button>
        </div>
      </div>

      <ul id="lista" class="list-group mt-4"></ul>
    </div>
  </div>

  <!-- Script con Firebase y funciones -->
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

    function filtrarEstudiantes() {
      const searchTerm = searchInput.value.toLowerCase();
      db.collection("estudiantes")
        .orderBy("nombre")
        .onSnapshot(snapshot => {
          lista.innerHTML = "";
          snapshot.forEach(doc => {
            const est = doc.data();
            if (est.seccion.toLowerCase().includes(searchTerm)) {
              const li = document.createElement("li");
              li.classList.add("list-group-item");
              li.innerHTML = `
                <strong>${est.nombre} ${est.apellido1} ${est.apellido2}</strong><br>
                Sección: ${est.seccion} | Materia: ${est.materia} | Docente: ${est.docente}<br>
                Ausencias: ${est.ausencias}<br>
                <em>Factores de riesgo:</em> ${est.riesgo}<br>
                <em>Acciones realizadas:</em> ${est.acciones}<br>
                <button class="btn btn-warning btn-sm mt-2" onclick="editarEstudiante('${doc.id}')">Editar</button>
                <button class="btn btn-danger btn-sm mt-2" onclick="eliminarEstudiante('${doc.id}')">Eliminar</button>
                <hr>
              `;
              lista.appendChild(li);
            }
          });
        });
    }

    db.collection("estudiantes").orderBy("nombre").onSnapshot(snapshot => {
      lista.innerHTML = "";
      snapshot.forEach(doc => {
        const est = doc.data();
        const li = document.createElement("li");
        li.classList.add("list-group-item");
        li.innerHTML = `
          <strong>${est.nombre} ${est.apellido1} ${est.apellido2}</strong><br>
          Sección: ${est.seccion} | Materia: ${est.materia} | Docente: ${est.docente}<br>
          Ausencias: ${est.ausencias}<br>
          <em>Factores de riesgo:</em> ${est.riesgo}<br>
          <em>Acciones realizadas:</em> ${est.acciones}<br>
          <button class="btn btn-warning btn-sm mt-2" onclick="editarEstudiante('${doc.id}')">Editar</button>
          <button class="btn btn-danger btn-sm mt-2" onclick="eliminarEstudiante('${doc.id}')">Eliminar</button>
          <hr>
        `;
        lista.appendChild(li);
      });
    });

    function editarEstudiante(id) {
      db.collection("estudiantes").doc(id).get().then(doc => {
        if (doc.exists) {
          const est = doc.data();
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
      });
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

      const snapshot = await db.collection("estudiantes").orderBy("nombre").get();

      let y = 10;
      doc.setFontSize(12);

      snapshot.forEach((docSnap, i) => {
        const est = docSnap.data();
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
      const snapshot = await db.collection("estudiantes").orderBy("nombre").get();
      const data = [];

      snapshot.forEach(docSnap => {
        const est = docSnap.data();
        data.push({
          Nombre: est.nombre,
          "Primer Apellido": est.apellido1,
          "Segundo Apellido": est.apellido2,
          Sección: est.seccion,
          Materia: est.materia,
          Docente: est.docente,
          Ausencias: est.ausencias,
          "Factores de riesgo": est.riesgo,
          "Acciones realizadas": est.acciones
        });
      });

      const worksheet = XLSX.utils.json_to_sheet(data);
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, "Estudiantes");

      XLSX.writeFile(workbook, "estudiantes.xlsx");
    }
  </script>

  <!-- Bootstrap 5 JS -->
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
