<html lang="ar">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<title>لعبة جدول الضرب - الفريقين</title>

<style>
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: linear-gradient(to bottom, #1f8fbf, #a3d9c5, #ffffff);
  text-align: center;
}

/* تصميم الحاوية */
.container {
  max-width: 1024px;
  margin: 0 auto;
  padding: 15px;
}

/* العنوان */
h1 {
  color: white;
  font-size: 2em;
}

/* الفرق والنتائج */
.teams {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
  flex-wrap: wrap;
}

.team {
  display: flex;
  align-items: center;
  padding: 10px;
  border-radius: 12px;
  color: white;
  font-weight: bold;
}

.team1 { background-color: #1f7a4c; justify-content: flex-start; }
.team2 { background-color: #bf1f3f; justify-content: flex-end; }

input.team-name {
  padding: 8px;
  width: 150px;
  border-radius: 8px;
  border: none;
  text-align: center;
  font-weight: bold;
}

/* زر البداية */
.start-btn {
  padding: 10px 25px;
  margin-top: 10px;
  border-radius: 10px;
  border: none;
  background-color: #ffd700;
  font-weight: bold;
  cursor: pointer;
}

/* زر الرجوع للبداية */
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
  grid-template-columns: repeat(auto-fit, minmax(60px, 1fr));
  gap: 6px;
  margin-top: 20px;
}

/* أزرار الأسئلة */
.question-btn {
  padding: 14px;
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
  max-width: 400px;
}

.answers button {
  display: block;
  margin: 8px auto;
  width: 80%;
  padding: 12px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
}

.correct { background-color: green !important; color: white; }
.wrong { background-color: red !important; color: white; }

.timer {
  font-size: 22px;
  margin-top: 10px;
  color: white;
}

.message {
  font-size: 18px;
  margin-top: 10px;
  font-weight: bold;
  color: #ff4500;
}

/* عرض النتائج (عدد الإجابات) */
.scores {
  margin-top: 15px;
  display: flex;
  justify-content: space-between;
  color: white;
  font-weight: bold;
  font-size: 18px;
}

.score-item {
  display: flex;
  align-items: center;
  gap: 8px;
}

.score-number {
  background: rgba(255,255,255,0.3);
  padding: 4px 8px;
  border-radius: 6px;
}
</style>
</head>

<body>
<div class="container">
  <h1>🎮 لعبة جدول الضرب</h1>

  <!-- زر العودة للبداية -->
  <button class="home-btn" onclick="restartGame()">🏠 الرجوع للبداية</button>

  <!-- الفرق -->
  <div class="teams">
    <div class="team team1">
      <input type="text" id="team1Name" class="team-name" placeholder="الفريق 1">
      <div class="score-item">عدد الإجابات: <span id="scoreTeam1" class="score-number">0</span></div>
    </div>
    <div class="team team2">
      <div class="score-item">عدد الإجابات: <span id="scoreTeam2" class="score-number">0</span></div>
      <input type="text" id="team2Name" class="team-name" placeholder="الفريق 2">
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
</div>

<audio id="correctSound" src="https://www.soundjay.com/human/applause-8.mp3"></audio>
<audio id="wrongSound" src="https://www.soundjay.com/button/beep-10.mp3"></audio>

<script>
let questions=[], currentTeam=1, timer, time=90, score1=0, score2=0;

function startGame(){
  let t1=document.getElementById("team1Name").value.trim();
  let t2=document.getElementById("team2Name").value.trim();
  if(!t1||!t2){ alert("ادخل اسم الفريقين!"); return;}
  document.getElementById("team1Name").disabled=true;
  document.getElementById("team2Name").disabled=true;
  document.getElementById("startBtn").disabled=true;
  generateQuestions();
  createGrid();
  updateScores();
}

function generateQuestions(){
  questions=[];
  for(let i=1;i<=10;i++){
    for(let j=1;j<=10;j++){
      let correct=i*j, options=[correct];
      while(options.length<4){ let rand=Math.floor(Math.random()*100); if(!options.includes(rand)) options.push(rand);}
      options.sort(()=>Math.random()-0.5);
      questions.push({q:`${i} × ${j}`, correct:correct, options:options});
    }
  }
}

function createGrid(){
  let grid=document.getElementById("grid"); grid.innerHTML="";
  questions.forEach((q,index)=>{
    let btn=document.createElement("button");
    btn.innerText=index+1;
    btn.className="question-btn";
    btn.onclick=()=>showQuestion(index,btn);
    grid.appendChild(btn);
  });
}

function showQuestion(index,btn){
  btn.disabled=true;
  let q=questions[index];
  document.getElementById("question").innerText=q.q;
  let answersDiv=document.getElementById("answers"); answersDiv.innerHTML="";
  document.getElementById("message").innerText="";
  document.getElementById("questionBox").style.display="block";
  startTimer();
  q.options.forEach(option=>{
    let b=document.createElement("button");
    b.innerText=option;
    b.onclick=()=>checkAnswer(b,option,q.correct);
    answersDiv.appendChild(b);
  });
}

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

function startTimer(){
  time=90; updateTimer();
  clearInterval(timer);
  timer=setInterval(()=>{
    time--; updateTimer();
    if(time<=0){ clearInterval(timer);
      document.getElementById("message").innerText="انتهى الوقت! السؤال للفريق الآخر ⏰";
      setTimeout(switchTeam,2000);
    }
  },1000);
}

function updateTimer(){
  document.getElementById("timer").innerText=`⏱ دور الفريق ${currentTeam} - ${time} ثانية`;
}

function switchTeam(){
  currentTeam=currentTeam===1?2:1;
  document.getElementById("questionBox").style.display="none";
  document.getElementById("timer").innerText="";
}

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

function updateScores(){
  document.getElementById("scoreTeam1").innerText=score1;
  document.getElementById("scoreTeam2").innerText=score2;
}

function restartGame(){ location.reload(); }
</script>
</body>
</html>
