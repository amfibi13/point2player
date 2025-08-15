<!DOCTYPE html>
<html>
<head>
  <title>Contador interactivo</title>
  <meta charset="UTF-8">
  <style>
    body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
    h1 { font-size: 2.5em; }
    button { font-size: 1.2em; margin: 10px; padding: 10px 20px; }
    .botonera {
      display: flex;
      justify-content: center;
      gap: 15px;
      margin-bottom: 10px;
    }
  </style>
</head>
<body>
  <div id="userIdContainer" style="font-size:1em; margin-bottom:10px; color:#555;"></div>
  <div id="pantallaInicio">
    <button id="btn1Jugador" onclick="iniciarJuego(1)">1 Jugador</button>
    <button id="btn2Jugadores" onclick="iniciarJuego(2)">2 Jugadores</button>
  </div>
  <div id="pantallaJuego" style="display:none; position:relative; min-height:400px;">
    <div id="juego1" class="juego" style="display:inline-block; margin: 0 40px;">
      <h1>Puntos: <span id="puntos1">0</span></h1>
      <div style="font-size:1.3em; margin-bottom:10px;">Nivel Jugador 1: <span id="nivel1">0</span></div>
      <div class="botonera">
        <button id="btnSumar1" onmousedown="sumarPuntoBloqueo(1, this)" onmouseup="habilitarBoton(this)" onmouseleave="habilitarBoton(this)">Sumar punto</button>
        <button id="btnRestar1" onmousedown="restarPuntoBloqueo(1, this)" onmouseup="habilitarBoton(this)" onmouseleave="habilitarBoton(this)">- un punto</button>
        <button onclick="resetearPuntos(1)">Resetear</button>
      </div>
      <p>Pulsa la tecla <b>1</b> para sumar y <b>r</b> para resetear.</p>
    </div>
    <div id="juego2" class="juego" style="display:none; margin: 0 40px;">
      <h1>Puntos: <span id="puntos2">0</span></h1>
      <div style="font-size:1.3em; margin-bottom:10px;">Nivel Jugador 2: <span id="nivel2">0</span></div>
      <div class="botonera">
        <button id="btnSumar2" onmousedown="sumarPuntoBloqueo(2, this)" onmouseup="habilitarBoton(this)" onmouseleave="habilitarBoton(this)">Sumar punto</button>
        <button id="btnRestar2" onmousedown="restarPuntoBloqueo(2, this)" onmouseup="habilitarBoton(this)" onmouseleave="habilitarBoton(this)">- un punto</button>
        <button onclick="resetearPuntos(2)">Resetear</button>
      </div>
      <p>Pulsa la tecla <b>2</b> para sumar y <b>t</b> para resetear.</p>
    </div>
    <div id="competicionContainer" style="display:none; position:absolute; left:0; right:0; bottom:30px; text-align:center;">
  <button id="btnCompeticion" style="font-size:1.3em; padding:12px 40px;" onclick="iniciarCompeticion()">Competición</button>
  <div id="contadorCompeticion" style="font-size:2em; margin-top:10px;"></div>
  <button id="btnCerrarCompeticion" style="display:none; margin-top:10px;" onclick="cancelarCompeticion()">Cerrar</button>
  <div id="ganadorOverlay" style="display:none; position:fixed; top:0; left:0; right:0; bottom:0; background:rgba(0,0,0,0.7); color:#fff; z-index:1000; justify-content:center; align-items:center; flex-direction:column; font-size:2em;">
    <div id="mensajeGanador"></div>
    <button id="btnCerrarGanador" onclick="cerrarGanador()">Cerrar</button>
  </div>
    </div>
    <div id="nivelOverlay" style="display:none; position:fixed; top:0; left:0; right:0; bottom:0; background:rgba(0,0,0,0.7); color:#fff; z-index:2000; justify-content:center; align-items:center; flex-direction:column; font-size:2.5em;">
    <div id="mensajeNivel"></div>
    <button onclick="cerrarNivelOverlay()">Cerrar</button>
  </div>
    <p><b>q</b> para salir (cerrar pestaña).</p>
  </div>
  <!-- Overlay para sumar puntos personalizados -->
<div id="sumarOverlay" style="display:none; position:fixed; top:0; left:0; right:0; bottom:0; background:rgba(0,0,0,0.7); color:#fff; z-index:3000; justify-content:center; align-items:center; flex-direction:column; font-size:2em;">
  <div>¿Cuántos puntos quieres sumar? (máx 10000)</div>
  <input id="inputSumar" type="number" min="1" max="10000" value="1" style="font-size:1.2em; margin:10px; width:120px; text-align:center;">
  <button onclick="confirmarSumar()">Sumar</button>
  <button onclick="cerrarSumarOverlay()">Cancelar</button>
</div>
  <script>
    // Variables globales
    let puntos1 = 0;
    let puntos2 = 0;
    let jugadores = 1;
    let puntos1Previo = 0;
    let puntos2Previo = 0;
    let enCompeticion = false;
    let timerCompeticion = null;
    let nivel1 = 0;
    let nivel2 = 0;

    function restarPunto(jugador) {
      if (enCompeticion && document.getElementById('pantallaJuego').style.display === '') {
        // Permitir restar solo durante competición
      }
      if (jugador === 2) {
        const btn = document.getElementById("btnRestar2");
        btn.disabled = true;
        puntos2 = Math.max(0, puntos2 - 1);
        document.getElementById("puntos2").textContent = puntos2;
        setTimeout(() => { btn.disabled = false; }, 100);
      } else {
        const btn = document.getElementById("btnRestar1");
        btn.disabled = true;
        puntos1 = Math.max(0, puntos1 - 1);
        document.getElementById("puntos1").textContent = puntos1;
        setTimeout(() => { btn.disabled = false; }, 100);
      }
    }

    function iniciarCompeticion() {
      if (enCompeticion) return;
      enCompeticion = true;
      // Guardar puntos previos
      puntos1Previo = puntos1;
      puntos2Previo = puntos2;
      // Resetear contadores
      resetearPuntos(1);
      resetearPuntos(2);
      // Desactivar botón
      document.getElementById('btnCompeticion').disabled = true;
      // Mostrar botón cerrar competición
      document.getElementById('btnCerrarCompeticion').style.display = 'inline-block';
      // Mostrar contador
      let segundos = 10;
      document.getElementById('contadorCompeticion').textContent = 'Tiempo: ' + segundos + 's';
      timerCompeticion = setInterval(() => {
        segundos--;
        document.getElementById('contadorCompeticion').textContent = 'Tiempo: ' + segundos + 's';
        if (segundos <= 0) {
          clearInterval(timerCompeticion);
          finalizarCompeticion();
        }
      }, 1000);
    }

    function cancelarCompeticion() {
  enCompeticion = false;
  clearInterval(timerCompeticion);
  document.getElementById('contadorCompeticion').textContent = '';
  document.getElementById('btnCompeticion').disabled = false;
  document.getElementById('btnCerrarCompeticion').style.display = 'none';
  // Restaurar puntos previos
  puntos1 = puntos1Previo;
  puntos2 = puntos2Previo;
  document.getElementById('puntos1').textContent = puntos1;
  document.getElementById('puntos2').textContent = puntos2;
}

    function setBotonesHabilitados(habilitado) {
  const botones = document.querySelectorAll('button');
  botones.forEach(btn => {
    if (btn.id !== 'btnCerrarGanador') {
      btn.disabled = !habilitado;
    }
  });
}

    function finalizarCompeticion() {
      enCompeticion = false;
      document.getElementById('btnCompeticion').disabled = false;
      document.getElementById('contadorCompeticion').textContent = '';
      document.getElementById('btnCerrarCompeticion').style.display = 'none';
      let ganador = '';
      if (puntos1 > puntos2) {
        ganador = '¡Jugador 1 gana!';
      } else if (puntos2 > puntos1) {
        ganador = '¡Jugador 2 gana!';
      } else {
        ganador = '¡Empate!';
      }
      document.getElementById('mensajeGanador').textContent = ganador;
      document.getElementById('ganadorOverlay').style.display = 'flex';
      setBotonesHabilitados(false);
      // Restaurar puntos después de mostrar el ganador
      setTimeout(() => {
        puntos1 = puntos1Previo;
        puntos2 = puntos2Previo;
        document.getElementById('puntos1').textContent = puntos1;
        document.getElementById('puntos2').textContent = puntos2;
      }, 100);
    }

    function cerrarGanador() {
      document.getElementById('ganadorOverlay').style.display = 'none';
      setBotonesHabilitados(true);
      document.getElementById('btnCompeticion').disabled = false;
    }
    function iniciarJuego(numJugadores) {
      jugadores = numJugadores;
      document.getElementById('pantallaInicio').style.display = 'none';
      document.getElementById('pantallaJuego').style.display = '';
      document.getElementById('juego1').style.display = 'inline-block';
      if (numJugadores === 2) {
        document.getElementById('juego2').style.display = 'inline-block';
        document.getElementById('competicionContainer').style.display = 'block';
      } else {
        document.getElementById('juego2').style.display = 'none';
        document.getElementById('competicionContainer').style.display = 'none';
      }
      resetearPuntos(1);
      resetearPuntos(2);
    }
    function actualizarNivelJugador(jugador, puntos) {
  if (jugador === 1) {
    let nuevoNivel = Math.floor(puntos / 100);
    if (nuevoNivel > nivel1) {
      nivel1 = nuevoNivel;
      document.getElementById('nivel1').textContent = nivel1;
      document.getElementById('mensajeNivel').textContent = '¡Jugador 1 ha subido de nivel!';
      document.getElementById('nivelOverlay').style.display = 'flex';
    } else {
      document.getElementById('nivel1').textContent = nivel1;
    }
  } else if (jugador === 2) {
    let nuevoNivel = Math.floor(puntos / 100);
    if (nuevoNivel > nivel2) {
      nivel2 = nuevoNivel;
      document.getElementById('nivel2').textContent = nivel2;
      document.getElementById('mensajeNivel').textContent = '¡Jugador 2 ha subido de nivel!';
      document.getElementById('nivelOverlay').style.display = 'flex';
    } else {
      document.getElementById('nivel2').textContent = nivel2;
    }
  }
}

    function cerrarNivelOverlay() {
      document.getElementById('nivelOverlay').style.display = 'none';
    }
    // Modifica sumarPunto para actualizar el nivel
    function sumarPunto(jugador) {
      if (enCompeticion && document.getElementById('pantallaJuego').style.display === '') {
        // Permitir sumar solo durante competición
      }
      if (jugador === 2) {
        const btn = document.getElementById("btnSumar2");
        btn.disabled = true;
        puntos2++;
        document.getElementById("puntos2").textContent = puntos2;
        actualizarNivelJugador(2, puntos2);
        setTimeout(() => { btn.disabled = false; }, 100);
      } else {
        const btn = document.getElementById("btnSumar1");
        btn.disabled = true;
        puntos1++;
        document.getElementById("puntos1").textContent = puntos1;
        actualizarNivelJugador(1, puntos1);
        setTimeout(() => { btn.disabled = false; }, 100);
      }
    }
    function resetearPuntos(jugador) {
      if (jugador === 2) {
        puntos2 = 0;
        document.getElementById("puntos2").textContent = puntos2;
      } else {
        puntos1 = 0;
        document.getElementById("puntos1").textContent = puntos1;
      }
    }
    function sumarPuntoBloqueo(jugador, btn) {
      sumarPunto(jugador);
      btn.disabled = true;
    }
    function restarPuntoBloqueo(jugador, btn) {
      restarPunto(jugador);
      btn.disabled = true;
    }
    function habilitarBoton(btn) {
      btn.disabled = false;
    }

function generarUUID() {
  // Simple UUID v4 generator
  return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
    var r = Math.random() * 16 | 0, v = c === 'x' ? r : (r & 0x3 | 0x8);
    return v.toString(16);
  });
}
function obtenerIdUsuario() {
  let id = localStorage.getItem('userIdContador');
  if (!id) {
    id = generarUUID();
    localStorage.setItem('userIdContador', id);
  }
  return id;
}
document.addEventListener('DOMContentLoaded', function() {
  const id = obtenerIdUsuario();
  document.getElementById('userIdContainer').textContent = 'Tu ID de usuario: ' + id;
  if (id === 'b055c6a1-96ad-4985-8e17-06fefbf0b8e5') {
    const btn = document.createElement('button');
    btn.textContent = 'Sumar ++';
    btn.style = 'font-size:1.2em; margin: 10px; padding: 10px 20px; background: #4caf50; color: #fff;';
    btn.onclick = function() {
      document.getElementById('sumarOverlay').style.display = 'flex';
      document.getElementById('inputSumar').value = 1;
    };
    document.body.insertBefore(btn, document.getElementById('pantallaInicio'));
  }
});
function cerrarSumarOverlay() {
  document.getElementById('sumarOverlay').style.display = 'none';
}
function confirmarSumar() {
  const val = parseInt(document.getElementById('inputSumar').value, 10);
  if (isNaN(val) || val < 1) return;
  const puntos = Math.min(val, 10000);
  for (let i = 0; i < puntos; i++) {
    sumarPunto(1);
    sumarPunto(2);
  }
  cerrarSumarOverlay();
}

// Atajos de teclado solo activos en el juego
    document.addEventListener('keydown', function(e) {
      if (document.getElementById('pantallaJuego').style.display === '') {
        if (jugadores === 2) {
          if (enCompeticion) {
            if (e.key === '1') {
              sumarPunto(1);
            } else if (e.key === '2') {
              sumarPunto(2);
            }
          } else {
            if (e.key === '1') {
              sumarPunto(1);
            } else if (e.key.toLowerCase() === 'r') {
              resetearPuntos(1);
            } else if (e.key === '2') {
              sumarPunto(2);
            } else if (e.key.toLowerCase() === 't') {
              resetearPuntos(2);
            } else if (e.key.toLowerCase() === 'q') {
              window.close();
            }
          }
        } else {
          if (enCompeticion) {
            if (e.key === '1') {
              sumarPunto(1);
            }
          } else {
            if (e.key === '1') {
              sumarPunto(1);
            } else if (e.key.toLowerCase() === 'r') {
              resetearPuntos(1);
            } else if (e.key.toLowerCase() === 'q') {
              window.close();
            }
          }
        }
      }
    });
  </script>
</body>
</html>
