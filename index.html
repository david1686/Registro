<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Registro de Estudiantes</title>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-firestore-compat.js"></script>
</head>
<body>
  <h1>Registro de Estudiantes</h1>

  <form id="formulario">
    <label>Nombre: <input type="text" id="nombre" required /></label><br><br>
    <label>Edad: <input type="number" id="edad" required /></label><br><br>
    <label>Correo: <input type="email" id="correo" required /></label><br><br>
    <button type="submit">Registrar</button>
  </form>

  <h2>Lista de Estudiantes</h2>
  <ul id="lista"></ul>

  <script>
    // Configuración de Firebase (la tuya, ya corregida)
    const firebaseConfig = {
      apiKey: "AIzaSyBKluxJeTIlO17uAYkrIr5JoTjLiovtDyM",
      authDomain: "registro-a9fd3.firebaseapp.com",
      projectId: "registro-a9fd3",
      storageBucket: "registro-a9fd3.appspot.com",  // CORREGIDO (.app → .appspot.com)
      messagingSenderId: "399328760047",
      appId: "1:399328760047:web:7c5c567fbefead86becb1a",
      measurementId: "G-DLZ74RJWPX"
    };

    // Inicializar Firebase
    firebase.initializeApp(firebaseConfig);
    const db = firebase.firestore();

    // Lógica del formulario
    const form = document.getElementById("formulario");
    const lista = document.getElementById("lista");

    form.addEventListener("submit", async (e) => {
      e.preventDefault();
      const nombre = document.getElementById("nombre").value;
      const edad = parseInt(document.getElementById("edad").value);
      const correo = document.getElementById("correo").value;

      try {
        await db.collection("estudiantes").add({ nombre, edad, correo });
        form.reset();
      } catch (error) {
        alert("Error al guardar: " + error.message);
      }
    });

    // Mostrar estudiantes en tiempo real
    db.collection("estudiantes").orderBy("nombre").onSnapshot(snapshot => {
      lista.innerHTML = "";
      snapshot.forEach(doc => {
        const est = doc.data();
        const li = document.createElement("li");
        li.textContent = `${est.nombre} - ${est.edad} años - ${est.correo}`;
        lista.appendChild(li);
      });
    });
  </script>
</body>
</html>
