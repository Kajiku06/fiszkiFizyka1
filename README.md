
<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>fizyka fiszki</title>
    <script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
    <script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        @keyframes gradientBG {
            0% { background-position: 0% 50%; }
            50% { background-position: 100% 50%; }
            100% { background-position: 0% 50%; }
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(-45deg, #ffffff, #eeeeee, #e8e8e8, #f7e9e9);
            background-size: 400% 400%;
            animation: gradientBG 15s ease infinite;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            overflow: hidden;
            transition: background-color 0.3s;
        }

        .container {
            background: rgba(255, 255, 255, 0.95);
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.1);
            width: 90%;
            max-width: 600px;
            text-align: center;
            position: relative;
            z-index: 10;
        }

        .card {
            min-height: 220px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            border: 2px solid #edeff2;
            border-radius: 15px;
            margin: 20px 0;
            padding: 20px;
            background: #fff;
            transition: transform 0.2s;
        }

        .shake {
            animation: shake 0.4s;
            border-color: #e74c3c !important;
        }

        @keyframes shake {
            0% { transform: translateX(0); }
            25% { transform: translateX(-10px); }
            50% { transform: translateX(10px); }
            75% { transform: translateX(-10px); }
            100% { transform: translateX(0); }
        }

        .formula { font-size: 26px; color: #004085; display: none; margin-top: 15px; }
        .question { font-size: 20px; font-weight: 600; color: #495057; }

        button {
            padding: 12px 25px;
            font-size: 16px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            transition: all 0.2s;
            margin: 5px;
            font-weight: bold;
        }

        .btn-main { background-color: #914da2; color: white; }
        .btn-success { background-color: #28a745; color: white; display: none; }
        .btn-danger { background-color: #dc3545; color: white; display: none; }

        button:active { transform: scale(0.95); }

        .explosion-overlay {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background-color: rgba(220, 53, 69, 0.3);
            display: none;
            pointer-events: none;
            z-index: 5;
        }
    </style>
</head>
<body>

<div id="explosion" class="explosion-overlay"></div>

<div class="container">
    <div id="start-screen">
        <h1>Fizyka 1 kol. 1 wyklad</h1>
        <p>wszelkie prawa zastrzezone!</p>
        <button class="btn-main" onclick="startSession()">START</button>
    </div>

    <div id="study-screen" style="display:none;">
        <div style="font-size: 12px; color: #6c757d;">Do zaliczenia: <span id="count">0</span></div>
        <div id="flashcard" class="card">
            <div id="question" class="question">Wczytywanie..</div>
            <div id="formula-container" class="formula">
                <span id="formula-text"></span>
            </div>
        </div>
        <button id="show-btn" class="btn-main" onclick="showFormula()">Pokaż wzór</button>
        <div id="action-btns">
            <button id="know-btn" class="btn-success" onclick="handleAnswer(true)">Znam</button>
            <button id="dont-know-btn" class="btn-danger" onclick="handleAnswer(false)">Nie znam</button>
        </div>
    </div>

    <div id="end-screen" style="display:none;">
        <h1>Udalo sie opanowac wszystko</h1>
        <button class="btn-main" onclick="startSession()">Jeszcze raz</button>
    </div>
</div>

<script>
    const allData = [
        {q: "Suma wektorowa (analitycznie)", a: "\\vec{a} + \\vec{b} = (a_x + b_x, a_y + b_y, a_z + b_z)"},
        {q: "Iloczyn skalarny (analitycznie)", a: "\\vec{a} \\cdot \\vec{b} = a_x b_x + a_y b_y + a_z b_z"},
        {q: "Iloczyn skalarny (geometrycznie)", a: "\\vec{a} \\cdot \\vec{b} = a \\cdot b \\cdot \\cos \\alpha"},
        {q: "Iloczyn wektorowy (macierzowo)", a: "\\vec{c} = \\vec{a} \\times \\vec{b} = \\begin{vmatrix} \\vec{i} & \\vec{j} & \\vec{k} \\ a_x & a_y & a_z \\ b_x & b_y & b_z \\end{vmatrix}"},
        {q: "Wartość iloczynu wektorowego", a: "|\\vec{c}| = a \\cdot b \\cdot \\sin \\alpha"},
        {q: "Prędkość liniowa chwilowa", a: "\\vec{v} = \\frac{d\\vec{s}}{dt}"},
        {q: "Prędkość kątowa", a: "\\vec{\\omega} = \\frac{d\\vec{\\theta}}{dt}"},
        {q: "Zależność prędkości liniowej od kątowej", a: "\\vec{v} = \\vec{\\omega} \\times \\vec{r}"},
        {q: "Przyspieszenie liniowe", a: "\\vec{a} = \\frac{d\\vec{v}}{dt}"},
        {q: "Przyspieszenie kątowe", a: "\\vec{\\epsilon} = \\frac{d\\vec{\\omega}}{dt}"},
        {q: "Przyspieszenie w ruchu po okręgu (styczne i normalne)", a: "\\vec{a} = \\vec{\\epsilon} \\times \\vec{r} - \\omega^2 \\cdot \\vec{r} = \\vec{a}_t + \\vec{a}_n"},
        {q: "Położenie w ruchu jednostajnie przyspieszonym", a: "x = x_0 + v_0 t + \\frac{1}{2}at^2"},
        {q: "Położenie w ruchu obrotowym jednostajnie przyspieszonym", a: "\\theta = \\theta_0 + \\omega_0 t + \\frac{1}{2}\\epsilon t^2"},
        {q: "Prędkość chwilowa w ruchu jednostajnie przyspieszonym", a: "v = v_0 + at"},
        {q: "Siła dośrodkowa", a: "F_d = \\frac{mv^2}{r}"},
        {q: "Moment bezwładności (całka)", a: "I = \\int r^2 dm"},
        {q: "Twierdzenie Steinera", a: "I = I_c + md^2"},
        {q: "Pęd ciała", a: "\\vec{p} = m \\cdot \\vec{v}"},
        {q: "Moment pędu", a: "\\vec{L} = I \\cdot \\vec{\\omega} \\text{ lub } \\vec{L} = \\vec{r} \\times \\vec{p}"},
        {q: "Moment siły", a: "\\vec{M} = \\vec{r} \\times \\vec{F}"},
        {q: "II zasada dynamiki Newtona (postać uogólniona)", a: "\\vec{F}_w = \\frac{d\\vec{p}}{dt}"},
        {q: "Praca mechaniczna (całka)", a: "W = \\int \\vec{F} \\cdot d\\vec{r}"},
        {q: "Energia potencjalna ciężkości", a: "E_p = mgh"},
        {q: "Energia kinetyczna ruchu postępowego", a: "E_k = \\frac{mv^2}{2}"},
        {q: "Energia kinetyczna ruchu obrotowego", a: "E_k = \\frac{I\\omega^2}{2}"},
        {q: "Moc chwilowa", a: "P = \\frac{dW}{dt}"},
        {q: "Zasada zachowania energii mechanicznej", a: "\\sum E = E_c = const"},
        {q: "Siła powszechnego ciążenia", a: "F = G \\frac{m_1 m_2}{r^2}"},
        {q: "Dylatacja czasu", a: "t = \\frac{t_0}{\\sqrt{1-\\beta^2}} \\text{ gdzie } \\beta = \\frac{v}{c}"},
        {q: "Energia całkowita (relatywistyczna)", a: "E = mc^2"},
        {q: "Energia spoczynkowa", a: "E = m_0 c^2"},
        {q: "Energia kinetyczna relatywistyczna", a: "E_k = mc^2 - m_0 c^2"}
    ];

    let currentQueue = [];
    let currentItem = null;

    function startSession() {
        currentQueue = [...allData].sort(() => Math.random() - 0.5);
        document.getElementById('start-screen').style.display = 'none';
        document.getElementById('end-screen').style.display = 'none';
        document.getElementById('study-screen').style.display = 'block';
        nextCard();
    }

    function nextCard() {
        if (currentQueue.length === 0) {
            finishSession();
            return;
        }
        currentItem = currentQueue.shift();
        document.getElementById('count').innerText = currentQueue.length + 1;
        document.getElementById('question').innerText = currentItem.q;
        document.getElementById('formula-text').innerText = `\\( ${currentItem.a} \\)`;
        
        document.getElementById('formula-container').style.display = 'none';
        document.getElementById('show-btn').style.display = 'inline-block';
        document.getElementById('know-btn').style.display = 'none';
        document.getElementById('dont-know-btn').style.display = 'none';
        
        MathJax.typeset();
    }

    function showFormula() {
        document.getElementById('formula-container').style.display = 'block';
        document.getElementById('show-btn').style.display = 'none';
        document.getElementById('know-btn').style.display = 'inline-block';
        document.getElementById('dont-know-btn').style.display = 'inline-block';
    }

    function handleAnswer(known) {
        if (known) {
            confetti({
                particleCount: 100,
                spread: 70,
                origin: { y: 0.6 }
            });
            nextCard();
        } else {
            const explosion = document.getElementById('explosion');
            const card = document.getElementById('flashcard');
            
            explosion.style.display = 'block';
            card.classList.add('shake');
            
            setTimeout(() => {
                explosion.style.display = 'none';
                card.classList.remove('shake');
                currentQueue.push(currentItem); 
                nextCard();
            }, 400);
        }
    }

    function finishSession() {
        document.getElementById('study-screen').style.display = 'none';
        document.getElementById('end-screen').style.display = 'block';
        confetti({
            particleCount: 200,
            spread: 120,
            origin: { y: 0.5 }
        });
    }
</script>

</body>
</html>
