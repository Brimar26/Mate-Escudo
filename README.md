
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>⚡ MATE-ESCUDO XPRESS ⚡</title>
    <style>
        body { background: #2c3e50; color: white; font-family: 'Arial Black', sans-serif; text-align: center; margin: 0; padding: 20px; }
        .game-box { background: #34495e; max-width: 500px; margin: auto; padding: 20px; border-radius: 20px; border: 5px solid #f1c40f; }
        #escena { height: 200px; background: #2c3e50; border: 2px solid #555; position: relative; margin: 20px 0; overflow: hidden; }
        #mate { font-size: 50px; position: absolute; bottom: 10px; left: 20px; transition: transform 0.2s; }
        #obstaculo { position: absolute; bottom: 20px; right: 20px; font-size: 1.5em; background: #c0392b; padding: 10px; border-radius: 10px; }
        .vida-bar { width: 100%; height: 20px; background: #7f8c8d; margin: 10px 0; border-radius: 10px; }
        #vida-actual { width: 100%; height: 100%; background: #27ae60; transition: width 0.3s; }
        #timer { font-size: 1.5em; color: #f1c40f; }
        .btn-main { padding: 15px 30px; background: #f39c12; border: none; color: white; cursor: pointer; border-radius: 10px; font-size: 1.2em; }
        .hidden { display: none; }
         /* IMPORTANTE: Ajuste para móviles */
@media (max-width: 768px) {
    .sidebar { width: 100%; height: auto; position: relative; } /* El menú se vuelve una barra superior */
    .main { margin-left: 0; width: 100%; } /* El contenido ocupa todo el ancho */
    .grid { grid-template-columns: repeat(2, 1fr); } /* Módulos de 2 en 2 en móviles */
}
 
  </style>
</head>
<body>

<div class="game-box">
    <h1>⚡ MATE-ESCUDO XPRESS ⚡</h1>
    <div id="instrucciones">
        <p>¡Misión: 15 retos! 4s por ejercicio.</p>
        <button class="btn-main" onclick="iniciarJuego()">¡COMENZAR!</button>
    </div>

    <div id="juego" class="hidden">
        <div id="timer">4s</div>
        <div class="vida-bar"><div id="vida-actual"></div></div>
        <div id="escena"><div id="mate">🏃‍♂️</div><div id="obstaculo">?</div></div>
        <input type="number" id="respuesta" placeholder="Resultado" autofocus>
        <button class="btn-main" onclick="verificar()">DISPARAR</button>
    </div>
</div>

<script>
    let vida = 100, aciertos = 0, intentos = 0, prob = {r:0}, tiempoRestante = 4, timerInterval;
    const ctx = new (window.AudioContext || window.webkitAudioContext)();

    function beep(freq) {
        let o = ctx.createOscillator(), g = ctx.createGain();
        o.frequency.value = freq; o.connect(g); g.connect(ctx.destination);
        o.start(); g.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.1);
        o.stop(ctx.currentTime + 0.1);
    }

    function iniciarJuego() {
        document.getElementById('instrucciones').classList.add('hidden');
        document.getElementById('juego').classList.remove('hidden');
        vida = 100; aciertos = 0; intentos = 0;
        generarProblema();
    }

    function generarProblema() {
        clearInterval(timerInterval);
        if (intentos >= 15) { terminar(); return; }
        
        tiempoRestante = 4;
        document.getElementById('timer').innerText = tiempoRestante + "s";
        
        // Tipos: 0=suma, 1=resta, 2=mult, 3=div
        let tipo = Math.floor(Math.random() * 4);
        let n1, n2;

        if(tipo === 0) { // Suma < 200
            n1 = Math.floor(Math.random() * 100); n2 = Math.floor(Math.random() * 100);
            prob = {p: `${n1} + ${n2}`, r: n1 + n2};
        } else if(tipo === 1) { // Resta < 200
            n1 = Math.floor(Math.random() * 200); n2 = Math.floor(Math.random() * n1);
            prob = {p: `${n1} - ${n2}`, r: n1 - n2};
        } else if(tipo === 2) { // Mult 2-12
            n1 = Math.floor(Math.random() * 11) + 2; n2 = Math.floor(Math.random() * 11) + 2;
            prob = {p: `${n1} x ${n2}`, r: n1 * n2};
        } else { // División
            n2 = Math.floor(Math.random() * 11) + 2;
            prob.r = Math.floor(Math.random() * 11) + 2;
            n1 = prob.r * n2;
            prob.p = `${n1} ÷ ${n2}`;
        }
        
        document.getElementById('obstaculo').innerText = prob.p;
        document.getElementById('respuesta').value = '';
        document.getElementById('respuesta').focus();

        timerInterval = setInterval(() => {
            tiempoRestante--;
            document.getElementById('timer').innerText = tiempoRestante + "s";
            if (tiempoRestante <= 0) {
                aplicarPenalizacion();
                generarProblema();
            }
        }, 1000);
    }

    function aplicarPenalizacion() {
        beep(150);
        vida -= 10;
        document.getElementById('vida-actual').style.width = vida + "%";
        if (vida <= 0) terminar();
    }

    function verificar() {
        let userR = parseInt(document.getElementById('respuesta').value);
        intentos++;
        if (userR === prob.r) {
            beep(600);
            aciertos++;
        } else {
            aplicarPenalizacion();
        }
        generarProblema();
    }

    function terminar() {
        clearInterval(timerInterval);
        let pts = Math.round((aciertos / 15) * 100);
        document.getElementById('juego').innerHTML = `<h2>Misión Finalizada</h2><p>Puntaje: ${pts}%</p><button class="btn-main" onclick="location.reload()">REINICIAR</button>`;
    }
</script>
</body>
</html>