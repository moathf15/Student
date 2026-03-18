<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>لعبة جدول الضرب - الفريقين</title>

<style>
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: linear-gradient(to bottom, #1f8fbf, #a3d9c5, #ffffff);
  text-align: center;
}

.container {
  padding: 15px;
}

h1 {
  color: white;
}

/* الفرق */
.teams {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 15px;
}

.team {
  flex: 1 1 150px;
  padding: 10px;
  border-radius: 12px;
  color: white;
  font-weight: bold;
}

.team1 { background-color: #1f7a4c; text-align:left; }
.team2 { background-color: #bf1f3f; text-align:right; }

input.team-name {
  padding: 8px;
  width: 120px;
  border-radius: 8px;
  border: none;
  text-align: center;
  font-weight: bold;
}

/* زر البداية */
.start-btn {
  padding: 10px 20px;
  margin-top: 10px;
  border-radius: 10px;
  border: none;
  background-color: #ffd700;
  font-weight: bold;
  cursor: pointer;
}

/* زر العودة للبداية */
.home-btn {
  padding: 8px 15px;
  border-radius: 8px;
  border: none;
  background-color: #ffffff;
  font-weight: bold;
  cursor: pointer;
  position: fixed;
  top: 10px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 10;
}

/* شبكة الأسئلة */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(50px, 1fr));
  gap: 5px;
  margin-top: 20px;
}

/* الأزرار */
.question-btn {
  padding: 12px;
  border-radius: 8px;
  background: white;
  font-weight: bold;
  cursor: pointer;
}

.question-btn:disabled {
  background: gray;
  color: white;
}

/* صندوق السؤال */
.question-box {
  display: none;
  background: white;
  color: black;
  margin: 20px auto;
  padding: 15px;
  border-radius: 12px;
  max-width: 350px;
}

.answers button {
  display: block;
  margin: 8px auto;
  width: 80%;
  padding: 10px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
}

.correct { background-color: green !important; color: white; }
.wrong { background-color: red !important; color: white; }

.timer {
  font-size: 20px;
  margin-top: 10px;
  color: white;
}

.message {
  font-size: 18px;
  margin-top: 10px;
  font-weight: bold;
  color: #ff4500;
}

.scores {
  margin-top: 10px;
  display: flex;
  justify-content: space-between;
  color: white;
  font-weight: bold;
}
</style>
</head>

<body>
<div class="container">
  <h1>🎮 لعبة جدول الضرب</h1>

  <button class="home-btn" onclick="restartGame()">🏠 الرجوع للبداية</button>

  <!-- الفرق -->
  <div class="teams">
    <div class="team team1">
      الفريق 1: <input type="text" id="team1Name" class="team-name" placeholder="اسم الفريق 1">
    </div>
    <div class="team team2">
      الفريق 2: <input type="text" id="team2Name" class="team-name" placeholder="اسم الفريق 2">
    </div>
  </div>

  <button class="start-btn" id="startBtn" onclick="startGame()">ابدأ اللعبة</button>

  <!-- المؤقت -->
  <div class="timer" id="timer"></div>

  <!-- شبكة الأسئلة -->
  <div class="grid" id="grid"></div>

  <!-- صندوق السؤال -->
  <div class="question-box" id="questionBox">
    <h2 id="question"></h2>
    <div class="answers" id="answers"></div>
    <div class="message" id="message"></div>
  </div>

  <!-- النتائج -->
  <div class="scores" id="scores">
    <div id="scoreTeam1">0</div>
    <div id="scoreTeam2">0</div>
  </div>
</div>

<audio id="correctSound" src="https://www.soundjay.com/human/applause-8.mp3"></audio>
<audio id="wrongSound" src="https://www.soundjay.com/button/beep-10.mp3"></audio>

<script>
let questions = [];
let currentTeam = 1;
let timer;
let time = 90;
let score1 = 0, score2 = 0;

/* بدء اللعبة */
function startGame() {
  let t1 = document.getElementById("team1Name").value.trim();
  let t2 = document.getElementById("team2Name").value.trim();
  if(!t1 || !t2) { alert("ادخل اسم الفريقين!"); return; }

  document.getElementById("team1Name").disabled = true;
  document.getElementById("team2Name").disabled = true;
  document.getElementById("startBtn").disabled = true;

  generateQuestions();
  createGrid();
  updateScores();
}

/* توليد الأسئلة */
function generateQuestions() {
  questions = [];
  for(let i=1;i<=10;i++){
    for(let j=1;j<=10;j++){
      let correct = i*j;
      let options=[correct];
      while(options.length<4){
        let rand=Math.floor(Math.random()*100);
        if(!options.includes(rand)) options.push(rand);
      }
      options.sort(()=>Math.random()-0.5);
      questions.push({q:`${i} × ${j}`, correct:correct, options:options});
    }
  }
}

/* إنشاء شبكة الأسئلة */
function createGrid(){
  let grid=document.getElementById("grid");
  grid.innerHTML="";
  questions.forEach((q,index)=>{
    let btn=document.createElement("button");
    btn.innerText=index+1;
    btn.className="question-btn";
    btn.onclick=()=> showQuestion(index,btn);
    grid.appendChild(btn);
  });
}

/* عرض السؤال */
function showQuestion(index,btn){
  btn.disabled=true;
  let q=questions[index];
  document.getElementById("question").innerText=q.q;
  let answersDiv=document.getElementById("answers");
  answersDiv.innerHTML="";
  document.getElementById("message").innerText="";
  document.getElementById("questionBox").style.display="block";
  startTimer();
  q.options.forEach(option=>{
    let b=document.createElement("button");
    b.innerText=option;
    b.onclick=()=> checkAnswer(b,option,q.correct);
    answersDiv.appendChild(b);
  });
}

/* التحقق من الإجابة */
function checkAnswer(button,selected,correct){
  clearInterval(timer);
  if(selected===correct){
    button.classList.add("correct");
    document.getElementById("correctSound").play();
    document.getElementById("message").innerText="أحسنتِ! ✅";
    if(currentTeam===1) score1++; else score2++;
    updateScores();
    setTimeout(showWinnerChoice,1500);
  } else{
    button.classList.add("wrong");
    document.getElementById("wrongSound").play();
    document.getElementById("message").innerText="خطأ ❌";
    setTimeout(switchTeam,1500);
  }
}

/* المؤقت */
function startTimer(){
  time=90;
  updateTimer();
  clearInterval(timer);
  timer=setInterval(()=>{
    time--;
    updateTimer();
    if(time<=0){
      clearInterval(timer);
      document.getElementById("message").innerText="انتهى الوقت! السؤال للفريق الآخر ⏰";
      setTimeout(switchTeam,2000);
    }
  },1000);
}

function updateTimer(){
  document.getElementById("timer").innerText=`⏱ دور الفريق ${currentTeam} - ${time} ثانية`;
}

/* تغيير الفريق */
function switchTeam(){
  currentTeam=currentTeam===1?2:1;
  document.getElementById("questionBox").style.display="none";
  document.getElementById("timer").innerText="";
}

/* اختيار الفريق الفائز */
function showWinnerChoice(){
  document.getElementById("message").innerHTML=`
    اختر الفريق الفائز بالسؤال:<br>
    <button onclick="winnerSelected(1)">الفريق 1</button>
    <button onclick="winnerSelected(2)">الفريق 2</button>
  `;
}

function winnerSelected(team){
  document.getElementById("message").innerText=`🎉 الفريق ${team} فاز بالسؤال!`;
  updateScores();
  setTimeout(switchTeam,2000);
}

/* تحديث النتائج */
function updateScores(){
  document.getElementById("scoreTeam1").innerText=document.getElementById("team1Name").value+": "+score1;
  document.getElementById("scoreTeam2").innerText=document.getElementById("team2Name").value+": "+score2;
}

/* زر الرجوع للبداية */
function restartGame(){
  location.reload();
}
</script>
</body>
</html>
