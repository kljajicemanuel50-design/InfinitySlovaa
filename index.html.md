# Infinity Slova  
  
<!DOCTYPE html>  
<html lang="en">  
<head>  
<meta charset="UTF-8">  
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">  
<title>Infinity Slova - Neon + Sound</title>  
  
<link rel="icon" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><rect width=100 height=100 fill=beige/><circle cx=50 cy=50 r=40 fill=lightblue/></svg>">  
  
<style>  
body{margin:0;font-family:Arial,sans-serif;background:#f5e3b2;overflow-x:hidden;color:white;text-align:center;}  
#game-container{position:relative;z-index:1;}  
.background{position:fixed;width:200%;height:200%;background: radial-gradient(circle at center, #fdf1c4, #f5e3b2 70%);top:-50%;left:-50%;z-index:0;}  
.spots{position:fixed;width:150px;height:150px;border-radius:50%;filter: blur(35px);}  
#spot1{top:10%;left:25%;background:rgba(255,230,200,0.7);}  
#spot2{top:70%;left:80%;background:rgba(200,255,220,0.7);}  
#spot3{top:40%;left:50%;background:rgba(200,220,255,0.7);}  
  
#stats{font-size:18px;margin-top:20px;}  
#grid{display:grid;gap:6px;justify-content:center;margin-top:10px;touch-action:none;}  
.cell{width:48px;height:48px;background:#222;display:flex;align-items:center;justify-content:center;font-size:20px;border-radius:10px;transition:0.15s,box-shadow 0.3s;cursor:pointer;}  
.cell:hover{transform:scale(1.1);}  
#scorePopup{position:absolute;color:#0ff;font-weight:bold;font-size:24px;opacity:0;pointer-events:none;text-shadow:0 0 8px #0ff,0 0 12px #0ff;animation:none;}  
#wordFlash{position:absolute;color:#ff0;font-weight:bold;font-size:28px;opacity:0;pointer-events:none;text-shadow:0 0 12px #ff0,0 0 18px #ff0;animation:none;}  
#leaderboard{margin-top:20px;font-size:16px;max-width:300px;margin-left:auto;margin-right:auto;text-align:left;}  
button{margin-top:10px;padding:6px 12px;font-size:16px;border-radius:8px;border:none;background:#ff9f0a;color:black;}  
</style>  
</head>  
<body>  
<div class="background"></div>  
<div id="spot1" class="spots"></div>  
<div id="spot2" class="spots"></div>  
<div id="spot3" class="spots"></div>  
  
<div id="game-container">  
<h2>Infinity Slova</h2>  
<div id="stats">Score: <span id="score">0</span> | Level: <span id="level">1</span> | <span id="combo"></span></div>  
<div id="word"></div>  
<div id="grid"></div>  
<div id="scorePopup"></div>  
<div id="wordFlash"></div>  
<button onclick="saveScore()">Save Score</button>  
<div id="leaderboard"><strong>Leaderboard:</strong><br/></div>  
</div>  
  
<!-- Zvukovi -->  
<audio id="bgMusic" loop src="https://actions.google.com/sounds/v1/cartoon/slide_whistle.ogg"></audio>  
<audio id="clickSound" src="https://actions.google.com/sounds/v1/cartoon/pop.ogg"></audio>  
<audio id="comboSound" src="https://actions.google.com/sounds/v1/cartoon/cartoon_boing.ogg"></audio>  
  
<script>  
const letters="ABCDEFGHIJLMNOPRSTUVZŽŠČĆĐ";  
const neonColors=["#0ff","#f0f","#0f0","#ff0","#f60","#f00","#0f8","#8f0"];  
const secretWords=["med","kava","caj","cvijet","mobitel"];  
  
let size=5,grid=[],selected=[],word="",score=0,level=1,combo=1,startTime=0;  
let username=localStorage.getItem("username")||"";  
  
const gridDiv=document.getElementById("grid");  
const wordDiv=document.getElementById("word");  
const scoreEl=document.getElementById("score");  
const levelEl=document.getElementById("level");  
const comboEl=document.getElementById("combo");  
const boardEl=document.getElementById("leaderboard");  
const scorePopup=document.getElementById("scorePopup");  
const wordFlash=document.getElementById("wordFlash");  
  
const bgMusic=document.getElementById("bgMusic");  
const clickSound=document.getElementById("clickSound");  
const comboSound=document.getElementById("comboSound");  
  
bgMusic.volume=0.25; bgMusic.play();  
  
// Funkcija za random neon boju različitu od trenutne  
function randomNeon(current){  
  let choices=neonColors.filter(c=>c!==current);  
  return choices[Math.floor(Math.random()*choices.length)];  
}  
  
// Kreiranje grid-a sa potrebnim slovima za tajnu riječ  
function createGrid(){  
  gridDiv.style.gridTemplateColumns=`repeat(${size},48px)`;  
  gridDiv.innerHTML="";grid=[];  
  let pool=letters.split("");  
  let needed=[];  
  secretWords.forEach(word=>{needed.push(...word.split(""));});  
  for(let r=0;r<size;r++){grid[r]=[];for(let c=0;c<size;c++){  
    let letter;  
    if(needed.length>0){letter=needed.splice(Math.floor(Math.random()*needed.length),1)[0];}  
    else{letter=pool[Math.floor(Math.random()*pool.length)];}  
    let cell=document.createElement("div");  
    cell.className="cell"; cell.textContent=letter; cell.dataset.r=r; cell.dataset.c=c;  
    cell.dataset.color="";  
    cell.addEventListener("touchstart",()=>pressCell(cell));  
    grid[r][c]=cell;  
    gridDiv.appendChild(cell);  
  }}  
}  
  
// Funkcija za pritisak na slovo  
function pressCell(cell){  
  clickSound.currentTime=0; clickSound.play();  
  let color=randomNeon(cell.dataset.color);  
  cell.dataset.color=color; cell.style.boxShadow=`0 0 15px ${color},0 0 30px ${color}`;  
  showScorePopup("+50",cell);  
  score+=50; scoreEl.textContent=score;  
  animateLetter(cell);  
}  
  
// Animacija nestajanja slova i zamjena  
function animateLetter(cell){  
  let oldLetter=cell.textContent;  
  cell.style.transition="all 0.8s ease-out";  
  cell.style.transform="translateY(-50px) scale(1.5)";  
  cell.style.opacity="0";  
  setTimeout(()=>{  
    let newLetter=letters[Math.floor(Math.random()*letters.length)];  
    cell.textContent=newLetter;  
    cell.style.transform="translateY(0) scale(1)";  
    cell.style.opacity="1";  
    cell.style.boxShadow="none";  
    cell.dataset.color="";  
    checkSecretWord();  
  },800);  
}  
  
// Popup score animacija  
function showScorePopup(text,cell){  
  scorePopup.textContent=text;  
  let rect=cell.getBoundingClientRect();  
  scorePopup.style.left=rect.left+rect.width/2+"px";  
  scorePopup.style.top=rect.top-20+"px";  
  scorePopup.style.opacity=1;  
  scorePopup.style.animation="popupAnim 1s ease-out";  
  scorePopup.style.transform="translate(-50%,0)";  
  setTimeout(()=>{scorePopup.style.opacity=0; scorePopup.style.animation="none";},1000);  
}  
  
// Provjera tajne riječi u gridu  
function checkSecretWord(){  
  let lettersInGrid="";grid.flat().forEach(c=>lettersInGrid+=c.textContent.toLowerCase());  
  secretWords.forEach(word=>{  
    if(lettersInGrid.includes(word)){  
      comboSound.play();  
      score+=250; scoreEl.textContent=score;  
      showWordFlash(word);  
    }  
  });  
}  
  
// Animacija tajne riječi  
function showWordFlash(word){  
  wordFlash.textContent="You found: "+word+"! +250";  
  wordFlash.style.opacity=1;  
  wordFlash.style.animation="wordAnim 1.2s ease-out";  
  setTimeout(()=>{wordFlash.style.opacity=0; wordFlash.style.animation="none";},1200);  
}  
  
// Lokalni leaderboard  
function saveScore(){  
  if(!username){username=prompt("Enter username:")||"Player"; localStorage.setItem("username",username);}  
  let highscore=parseInt(localStorage.getItem("highscore")||"0");  
  if(score>highscore){localStorage.setItem("highscore",score);}  
  loadLeaderboard();  
}  
  
function loadLeaderboard(){  
  let highscore=parseInt(localStorage.getItem("highscore")||"0");  
  let user=localStorage.getItem("username")||"Player";  
  boardEl.innerHTML="<strong>Leaderboard:</strong><br>"+user+" — "+highscore;  
}  
  
// Automatsko update slova svake 2.5 sekunde  
setInterval(()=>{grid.flat().forEach(cell=>animateLetter(cell));},2500);  
  
loadLeaderboard();  
createGrid();  
</script>  
  
<style>  
@keyframes popupAnim{0%{transform:translate(-50%,0) scale(1);opacity:1;}100%{transform:translate(-50%,-50px) scale(1.5);opacity:0;}}  
@keyframes wordAnim{0%{opacity:1;transform:scale(1);}50%{transform:scale(1.5);}100%{opacity:0;transform:scale(1.8);}}  
</style>  
  
</body>  
</html>  
