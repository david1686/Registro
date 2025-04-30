<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Registro de Estudiantes</title>

  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>

  <!-- Firebase -->
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-firestore-compat.js"></script>

  <!-- jsPDF para exportar a PDF -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

  <!-- SheetJS para exportar a Excel -->
  <script src="https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js"></script>
</head>
<body class="bg-gray-100">

  <div class="container mx-auto p-6">
    <h1 class="text-3xl font-bold text-center mb-8">Registro de Estudiantes</h1>

    <form id="formulario" class="bg-white p-6 rounded-lg shadow-md">
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div>
          <label for="apellido1" class="block text-sm font-semibold">Primer Apellido:</label>
          <input type="text" id="apellido1" class="w-full p-2 border border-gray-300 rounded-md" required>
        </div>
        <div>
          <label for="apellido2" class="block text-sm font-semibold">Segundo Apellido:</label>
          <input type="text" id="apellido2" class="w-full p-2 border border-gray-300 rounded-md" required>
        </div>
        <div>
          <label for="nombre" class="block text-sm font-semibold">Nombre:</label>
          <input type="text" id="nombre" class="w-full p-2 border border-gray-300 rounded-md" required>
        </div>
        <div>
          <label for="seccion" class="block text-sm font-semibold">Sección:</label>
          <input type="text" id="seccion" class="w-full p-2 border border-gray-300 rounded-md" required>
        </div>
        <div>
          <label for="materia" class="block text-sm font-semibold">Materia:</label>
          <input type="text" id="materia" class="w-full p-2 border border-gray-300 rounded-md" required>
        </div>
        <div>
          <label for="docente" class="block text-sm font-semibold">Docente:</label>
          <input type="text" id="docente" class="w-full p-2 border border-gray-300 rounded-md" required>
        </div>
        <div>
          <label for="ausencias" class="block text-sm font-semibold">Número de Ausencias:</label>
          <input type="number" id="ausencias" class="w-full p-2 border border-gray-300 rounded-md" required>
        </div>
        <div class="col-span-2">
          <label for="riesgo" class="block text-sm font-semibold">Factores de Riesgo:</label>
          <textarea id="riesgo" class="w-full p-2 border border-gray-300 rounded-md" required></textarea>
        </div>
        <div class="col-span-2">
          <label for="acciones" class="block text-sm font-semibold">Acciones Realizadas:</label>
          <textarea id="acciones" class="w-full p-2 border border-gray-300 rounded-md" required></textarea>
        </div>
      </div>
      <div class="mt-4 flex justify-end">
        <button type="submit" class="bg-blue-500 text-white py-2 px-6 rounded-md hover:bg-blue-600">Registrar</button>
      </div>
    </form>

    <div class="mt-8">
      <div class="flex justify-between items-center mb-4">
        <div class="w-1/2">
          <label for="search" class="block text-sm font-semibold">Buscar por Sección:</label>
          <input type="text" id="search" class="w-full p-2 border border-gray-300 rounded-md" placeholder="Ej: 10-2" oninput="filtrarEstudiantes()">
        </div>
        <div class="flex space-x-2">
          <button class="bg-green-500 text-white px-4 py-2 rounded-md hover:bg-green-600" onclick="exportarPDF()">Exportar a PDF</button>
          <button class="bg-green-500 text-white px-4 py-2 rounded-md hover:bg-green-600" onclick="exportarExcel()">Exportar a Excel</button>
        </div>
      </div>

      <ul id="lista" class="space-y-4"></ul>
    </div>
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
              li.classList.add("bg-white", "p-4", "rounded-lg", "shadow-md");
              li.innerHTML = `
                <strong class="text-xl">${est.nombre} ${est.apellido1} ${est.apellido2}</strong><br>
                <span class="text-sm">Sección: ${est.seccion} | Materia: ${est.materia} | Docente: ${est.docente}</span><br>
                <span class="text-sm">Ausencias: ${est.ausencias}</span><br>
                <em class="text-sm">Factores de riesgo:</em> ${est.riesgo}<br>
                <em class="text-sm">Acciones realizadas:</em> ${est.acciones}<br>
                <div class="mt-2 space-x-2">
                  <button class="bg-yellow-500 text-white px-4 py-2 rounded-md hover:bg-yellow-600" onclick="editarEstudiante('${doc.id}')">Editar</button>
                  <button class="bg-red-500 text-white px-4 py-2 rounded-md hover:bg-red-600" onclick="eliminarEstudiante('${doc.id}')">Eliminar</button>
                </div>
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
        li.classList.add("bg-white", "p-4", "rounded-lg", "shadow-md");
        li.innerHTML = `
          <strong class="text-xl">${est.nombre} ${est.apellido1} ${est.apellido2}</strong><br>
          <span class="text-sm">Sección: ${est.seccion} | Materia: ${est.materia} | Docente: ${est.docente}</span><br>
          <span class="text-sm">Ausencias: ${est.ausencias}</span><br>
          <em class="text-sm">Factores de riesgo:</em> ${est.riesgo}<br>
          <em class="text-sm">Acciones realizadas:</em> ${est.acciones}<br>
          <div class="mt-2 space-x-2">
            <button class="bg-yellow-500 text-white px-4 py-2 rounded-md hover:bg-yellow-600" onclick="editarEstudiante('${doc.id}')">Editar</button>
            <button class="bg-red-500 text-white px-4 py-2 rounded-md hover:bg-red-600" onclick="eliminarEstudiante('${doc.id}')">Eliminar</button>
          </div>
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
</body>
</html>
