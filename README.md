<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>📢 Sorteo Quiniela del 00 al 99</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: linear-gradient(to right, #74ebd5, #9face6);
            text-align: center;
            padding: 20px;
            color: #333;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh; /* Ocupa al menos el 100% de la altura de la vista */
            margin: 0;
        }
        .container {
            background-color: #fff;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            max-width: 500px;
            width: 90%; /* Ajustable a pantallas más pequeñas */
            margin: auto; /* Centra el contenedor */
        }
        h1 {
            color: #007bff;
            margin-bottom: 20px;
        }
        p {
            font-size: 1.1em;
            margin-bottom: 20px;
        }
        input, button {
            padding: 12px;
            margin: 10px 0; /* Margen superior e inferior */
            font-size: 18px;
            border-radius: 5px;
            border: 1px solid #ddd;
            width: 100%; /* Ocupa el ancho completo del contenedor */
            box-sizing: border-box; /* Incluye padding/border en el ancho */
        }
        input[type="number"] {
            -moz-appearance: textfield; /* Elimina flechas en Firefox */
        }
        input::-webkit-outer-spin-button,
        input::-webkit-inner-spin-button {
            -webkit-appearance: none; /* Elimina flechas en Chrome/Safari */
            margin: 0;
        }
        button {
            background-color: #28a745;
            color: white;
            cursor: pointer;
            transition: background-color 0.3s ease;
            width: auto; /* Permite que el botón se ajuste a su contenido */
            padding: 12px 25px;
            display: block; /* Para centrarlo con margin: auto */
            margin: 20px auto 10px auto;
        }
        button:hover {
            background-color: #218838;
        }
        #estado {
            margin-top: 25px;
            font-weight: bold;
            font-size: 1.2em;
            padding: 10px;
            border-radius: 5px;
        }
        .ocupado {
            color: #dc3545; /* Rojo */
            background-color: #f8d7da; /* Fondo rojo claro */
            border: 1px solid #f5c6cb;
        }
        .disponible {
            color: #28a745; /* Verde */
            background-color: #d4edda; /* Fondo verde claro */
            border: 1px solid #c3e6cb;
        }
        .mensaje {
            font-size: 1.2em;
        }
    </style>
</head>
<body>

    <div class="container">
        <h1>🎯 Sorteo Quiniela</h1>
        <p>Elegí tu número del 00 al 99 (solo uno por persona)</p>

        <input type="text" id="nombre" placeholder="Tu nombre" required />
        <input type="number" id="numero" placeholder="Número (00-99)" min="0" max="99" oninput="verificarNumero()" required />
        <br />
        <button onclick="anotarJugador()">Anotarme</button>

        <div id="estado" class="mensaje"></div>
    </div>

    <script>
        // ¡TU URL DE APPS SCRIPT!
        const API_URL = "https://script.google.com/macros/s/AKfycbzRHFKWxxI7Qy4973YJHjlfbxJVCXTlE6roThVcfXT6OQ756Qlc9vUDO-9WgNemXNxzMA/exec";
        
        let numerosOcupados = [];

        // Cargar la lista de números ya ocupados desde Google Sheet
        async function cargarJugadores() {
            try {
                const response = await fetch(API_URL);
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                const data = await response.json();
                numerosOcupados = data.map(j => parseInt(j.numero)).filter(n => !isNaN(n)); // Aseguramos que sean números válidos
                console.log("Números ocupados cargados:", numerosOcupados);
            } catch (err) {
                console.error("Error al cargar jugadores:", err);
                document.getElementById("estado").innerHTML = "Error al cargar la lista de números. Recarga la página.";
                document.getElementById("estado").className = "mensaje ocupado";
            }
        }

        // Verifica si el número ingresado ya está ocupado
        async function verificarNumero() {
            const numeroInput = document.getElementById("numero");
            const numero = parseInt(numeroInput.value);
            const estado = document.getElementById("estado");

            if (numeroInput.value.trim() === '') {
                estado.innerHTML = "Ingresá un número.";
                estado.className = "mensaje";
                return;
            }

            if (isNaN(numero) || numero < 0 || numero > 99) {
                estado.innerHTML = "Ingresá un número válido (00 al 99).";
                estado.className = "mensaje ocupado";
                return;
            }

            estado.innerHTML = "Verificando número...";
            estado.className = "mensaje";

            if (numerosOcupados.includes(numero)) {
                estado.innerHTML = `El número ${numero.toString().padStart(2, "0")} ya está ocupado ❌`;
                estado.className = "mensaje ocupado";
            } else {
                estado.innerHTML = `El número ${numero.toString().padStart(2, "0")} está disponible ✅`;
                estado.className = "mensaje disponible";
            }
        }

        // Envía los datos del jugador si el número está libre
        async function anotarJugador() {
            const nombre = document.getElementById("nombre").value.trim();
            const numero = parseInt(document.getElementById("numero").value);
            const estado = document.getElementById("estado");

            if (nombre === '' || isNaN(numero) || numero < 0 || numero > 99) {
                estado.innerHTML = "Completá correctamente tu nombre y un número válido (00-99).";
                estado.className = "mensaje ocupado";
                return;
            }

            // Volvemos a cargar jugadores antes de anotar para asegurar que la lista esté actualizada
            await cargarJugadores();

            if (numerosOcupados.includes(numero)) {
                estado.innerHTML = `El número ${numero.toString().padStart(2, "0")} ya está ocupado. Elegí otro ❌`;
                estado.className = "mensaje ocupado";
                return;
            }

            estado.innerHTML = "Registrando tu participación...";
            estado.className = "mensaje";

            try {
                const response = await fetch(API_URL, {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({ nombre, numero })
                });

                const text = await response.text();
                console.log("Respuesta del servidor:", text);

                if (response.ok && text.includes("Anotado")) {
                    estado.innerHTML = `✅ ¡Te anotaste con el número ${numero.toString().padStart(2, "0")} exitosamente!`;
                    estado.className = "mensaje disponible";
                    document.getElementById("nombre").value = '';
                    document.getElementById("numero").value = '';
                    numerosOcupados.push(numero); // Agregamos el número recién anotado a la lista local
                } else {
                    estado.innerHTML = `⚠️ No se pudo anotar. ${text || 'Error desconocido'}`;
                    estado.className = "mensaje ocupado";
                }
            } catch (err) {
                estado.innerHTML = "Error al conectar con el servidor. Intenta de nuevo.";
                estado.className = "mensaje ocupado";
                console.error("Error al enviar datos:", err);
            }
        }

        // Carga inicial de jugadores al cargar la página
        cargarJugadores();
    </script>

</body>
</html>
