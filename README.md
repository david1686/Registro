<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Registro de Estudiantes</title>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-firestore-compat.js"></script>

  <!-- jsPDF para exportar a PDF -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

  <!-- SheetJS para exportar a Excel -->
  <script src="https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js"></script>
</head>
<body>
  <h1>Registro de Estudiantes</h1>

  <form id="formulario">
    <label>Primer Apellido: <input type="text" id="apellido1" required></label><br><br>
    <label>Segundo Apellido: <input type="text" id="apellido2" required></label><br><br>
    <label>Nombre: <input type="text" id="nombre" required></label><br><br>
    <label>Sección: <input type="text" id="seccion" required></label><br><br>
    <label>Materia: <input type="text" id="materia" required></label><br><br>
    <label>Docente: <input type="text" id="docente" required></label><br><br>
    <label>Número de ausencias: <input type="number" id="ausencias" required></label><br><br>
    <label>Factores de riesgo:<br><textarea id="riesgo" required></textarea></label><br><br>
    <label>Acciones realizadas:<br><textarea id="acciones" required></textarea></label><br><br>
    <button type="submit">Registrar</button>
  </form>

  <h2>Lista de Estudiantes</h2>

  <!-- Campo de búsqueda por sección -->
  <label for="search">Buscar por Sección: </label>
  <input type="text" id="search" placeholder="Ej: 10-2" oninput="filtrarEstudiantes()">
  <br><br>

  <!-- Botones de exportación -->
  <button onclick="exportarPDF()">Exportar a PDF</button>
  <button onclick="exportarExcel()">Exportar a Excel</button>

  <ul id="lista"></ul>

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
        filtrarEstudiantes(); // Refrescar la lista después de guardar
      } catch (error) {
        alert("Error al guardar: " + error.message);
      }
    });

    function filtrarEstudiantes() {
      const searchTerm = searchInput.value.toLowerCase();
      estudiantesFiltrados = [];

      db.collection("estudiantes")
        .orderBy("nombre")
        .get()
        .then(snapshot => {
          lista.innerHTML = "";
          snapshot.forEach(doc => {
            const est = doc.data();
            if (est.seccion.toLowerCase().includes(searchTerm)) {
              estudiantesFiltrados.push(est);

              const li = document.createElement("li");
              li.innerHTML = `
                <strong>${est.nombre} ${est.apellido1} ${est.apellido2}</strong><br>
                Sección: ${est.seccion} | Materia: ${est.materia} | Docente: ${est.docente}<br>
                Ausencias: ${est.ausencias}<br>
                <em>Factores de riesgo:</em> ${est.riesgo}<br>
                <em>Acciones realizadas:</em> ${est.acciones}<br>
                <button onclick="editarEstudiante('${doc.id}')">Editar</button>
                <button onclick="eliminarEstudiante('${doc.id}')">Eliminar</button>
                <hr>
              `;
              lista.appendChild(li);
            }
          });

          // Si el campo de búsqueda está vacío, mostrar todos
          if (searchTerm === "") {
            cargarEstudiantes(); // Mostrar todo
          }
        });
    }

    function cargarEstudiantes() {
      db.collection("estudiantes").orderBy("nombre").onSnapshot(snapshot => {
        lista.innerHTML = "";
        estudiantesFiltrados = [];
        snapshot.forEach(doc => {
          const est = doc.data();
          estudiantesFiltrados.push(est); // Se usa para exportar si no hay filtro

          const li = document.createElement("li");
          li.innerHTML = `
            <strong>${est.nombre} ${est.apellido1} ${est.apellido2}</strong><br>
            Sección: ${est.seccion} | Materia: ${est.materia} | Docente: ${est.docente}<br>
            Ausencias: ${est.ausencias}<br>
            <em>Factores de riesgo:</em> ${est.riesgo}<br>
            <em>Acciones realizadas:</em> ${est.acciones}<br>
            <button onclick="editarEstudiante('${doc.id}')">Editar</button>
            <button onclick="eliminarEstudiante('${doc.id}')">Eliminar</button>
            <hr>
          `;
          lista.appendChild(li);
        });
      });
    }

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
          idActual = doc.id;
        }
      });
    }

    function eliminarEstudiante(id) {
      if (confirm("¿Deseás eliminar este registro?")) {
        db.collection("estudiantes").doc(id).delete().then(() => {
          filtrarEstudiantes(); // Refresca después de eliminar
        }).catch(err => {
          alert("Error al eliminar: " + err.message);
        });
      }
    }

    async function exportarPDF() {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF();

      const datos = estudiantesFiltrados.length > 0 ? estudiantesFiltrados :
        (await db.collection("estudiantes").orderBy("nombre").get()).docs.map(d => d.data());

      let y = 10;
      doc.setFontSize(12);

      datos.forEach((est, i) => {
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
      const datos = estudiantesFiltrados.length > 0 ? estudiantesFiltrados :
        (await db.collection("estudiantes").orderBy("nombre").get()).docs.map(d => d.data());

      const data = datos.map(est => ({
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

    // Cargar todo al principio
    cargarEstudiantes();
  </script>
</body>
</html>
