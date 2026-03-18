<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>لعبة جدول الضرب</title>

<style>
body {
  margin: 0;
  font-family: Arial;
  background: linear-gradient(135deg, #1f7a4c, #6b1e1e, #ffffff);
  text-align: center;
}

.container {
  padding: 20px;
}

h1 {
  color: white;
}

/* الفرق */
.teams input {
  padding: 12px;
  margin: 10px;
  border-radius: 10px;
  border: none;
}

.start-btn {
  padding: 12px 25px;
  background: #1f7a4c;
  color: white;
  border-radius: 10px;
  cursor: pointer;
}

/* الشبكة */
.grid {
  display: grid;
  grid-template-columns: repeat(10, 1fr);
  gap: 6px;
  margin-top: 20px;
}

.question-btn {
  padding: 12px;
  border-radius: 8px;
  background: white;
  cursor: pointer;
  font-weight: bold;
}

.question-btn:disabled {
  background: gray;
  color: white;
}

/* السؤال */
.question-box {
  display: none;
  background: white;
  margin: 20px auto;
  padding: 20px;
  border-radius: 15px;
  width: 300px;
}

.answers button {
  display: block;
  margin: 10px auto;
  width: 80%;
  padding: 10px;
  border-radius: 10px;
  cursor: pointer;
}

.correct {
  background: green !important;
  color: white;
}

.wrong {
  background: red !important;
  color: white;
}

.timer {
  font-size: 25px;
  color: white;
  margin-top: 10px;
}
</style>

</head>

<body>

<div class="container">

<h1>🎮 لعبة جدول الضرب</h1>

<div class="teams">
  <input id="team1" placeholder="اسم الفريق الأول">
  <input id="team2" placeholder="اسم الفريق الثاني">
  <br>
  <button class="start-btn" onclick="startGame()">ابدأ</button>
</div>

<div class="timer" id="timer"></div>

<div class="grid" id="grid"></div>

<div class="question-box" id="questionBox">
  <h2 id="question"></h2>
  <div class="answers" id="answers"></div>
</div>

</div>

<audio id="correctSound" src="https://www.soundjay.com/human/applause-8.mp3"></audio>
<audio id="wrongSound" src="https://www.soundjay.com/button/beep-10.mp3"></audio>

<script>

let questions = [];
let currentTeam = 1;
let timer;
let time = 120;

/* بدء اللعبة */
function startGame() {
  generateQuestions();
  createGrid();
  startTimer();
}

/* توليد الأسئلة */
function generateQuestions() {
  questions = [];
  for (let i = 1; i <= 10; i++) {
    for (let j = 1; j <= 10; j++) {

      let correct = i * j;

      let options = [correct];

      while (options.length < 4) {
        let rand = Math.floor(Math.random() * 100);
        if (!options.includes(rand)) options.push(rand);
      }

      options.sort(() => Math.random() - 0.5);

      questions.push({
        q: `${i} × ${j}`,
        correct: correct,
        options: options
      });
    }
  }
}

/* إنشاء الشبكة */
function createGrid() {
  let grid = document.getElementById("grid");
  grid.innerHTML = "";

  questions.forEach((q, index) => {
    let btn = document.createElement("button");
    btn.innerText = index + 1;
    btn.className = "question-btn";

    btn.onclick = () => showQuestion(index, btn);

    grid.appendChild(btn);
  });
}

/* عرض السؤال */
function showQuestion(index, btn) {
  btn.disabled = true;

  let q = questions[index];
  document.getElementById("question").innerText = q.q;

  let answersDiv = document.getElementById("answers");
  answersDiv.innerHTML = "";

  q.options.forEach(option => {
    let b = document.createElement("button");
    b.innerText = option;

    b.onclick = () => checkAnswer(b, option, q.correct);

    answersDiv.appendChild(b);
  });

  document.getElementById("questionBox").style.display = "block";
}

/* التحقق */
function checkAnswer(button, selected, correct) {
  if (selected === correct) {
    button.classList.add("correct");
    document.getElementById("correctSound").play();
  } else {
    button.classList.add("wrong");
    document.getElementById("wrongSound").play();
  }

  setTimeout(() => {
    document.getElementById("questionBox").style.display = "none";
  }, 1500);
}

/* المؤقت */
function startTimer() {
  time = 120;
  updateTimer();

  clearInterval(timer);
  timer = setInterval(() => {
    time--;
    updateTimer();

    if (time <= 0) {
      switchTeam();
    }
  }, 1000);
}

function updateTimer() {
  document.getElementById("timer").innerText =
    `⏱ الفريق ${currentTeam} - ${time} ثانية`;
}

/* تغيير الفريق */
function switchTeam() {
  currentTeam = currentTeam === 1 ? 2 : 1;
  time = 120;
}

</script>

</body>
</html>
