
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SAI - Smart Academic Institutions</title>
<style>
body { font-family: "Segoe UI", sans-serif; margin:0; background:#f0f4f8; color:#333; }
header { background:#243b55; color:white; text-align:center; padding:20px; font-size:1.8em; }
section { padding:20px; text-align:center; }
button { background:#28a745; color:white; padding:12px 25px; border:none; border-radius:8px; cursor:pointer; font-size:1em; transition:0.3s; }
button:hover { background:#218838; }
.hidden { display:none; }
input, select { padding:10px; margin:10px 0; border-radius:5px; border:1px solid #ccc; width:70%; }
.ai-box { margin-top:20px; background:white; padding:20px; border-radius:10px; width:80%; margin:auto; box-shadow:0 4px 8px rgba(0,0,0,0.1); }
.ai-chat { height:300px; overflow-y:auto; text-align:left; padding:10px; border:1px solid #ddd; border-radius:8px; margin-bottom:10px; background:#f9f9f9; }
.message { padding:8px 12px; border-radius:15px; margin:5px 0; max-width:70%; display:inline-block; }
.user { background:#007bff; color:white; float:right; text-align:right; }
.ai { background:#e0e0e0; color:#333; float:left; text-align:left; }
.clearfix::after { content:""; clear:both; display:table; }
.meet-box { width:80%; margin:20px auto; }
iframe { width:100%; height:400px; border:none; border-radius:10px; }
</style>
</head>
<body>

<header>SAI - Smart Academic Institutions</header>

<section id="demo-section">
  <h2>🎓 Book Your Demo Class</h2>
  <p>Choose your demo date:</p>
  <input type="date" id="demo-date" min="2025-01-01" max="2035-12-31"><br>
  <label>Time: 4:30 PM - 6:30 PM</label><br><br>
  <button onclick="bookDemo()">Book Demo Date & Time</button>
  <p id="demo-msg"></p>
</section>

<section id="login-section" class="hidden">
  <h2>Join SAI</h2>
  <p>Enter your name:</p>
  <input type="text" id="userName" placeholder="Your Name"><br><br>
  <button onclick="loginUser()">Login</button>
  <p id="login-msg"></p>
</section>

<section id="ai-section" class="hidden">
  <h2>🤖 Welcome to SAI AI</h2>
  <p id="welcomeText"></p>
  <div class="ai-box">
    <div class="ai-chat" id="chatBox">
      <div class="clearfix"><div class="message ai">Hi! I’m SAI AI. Ask me any question if your teacher isn’t available.</div></div>
    </div>
    <input type="text" id="userQuestion" placeholder="Type your question...">
    <button onclick="askAI()">Ask</button><br><br>
  </div>
  <div class="meet-box">
    <h3>📹 SAI Online Meet</h3>
    <iframe src="https://meet.jit.si/SAI2025MEET" allow="camera; microphone; fullscreen"></iframe>
    <p>Phone Join: <a href="tel:+911234567896">+91 1234567896</a></p>
  </div>
</section>

<script>
let students = JSON.parse(localStorage.getItem("sai_students")) || [];
let feePaid = false;
let studentCode = "";
let studentName = "";

function bookDemo() {
  const date = document.getElementById("demo-date").value;
  if(!date) { alert("Select a date!"); return; }
  document.getElementById("demo-msg").innerHTML = `🎉 Congrats! Demo booked on ${date} (4:30-6:30PM)`;
  localStorage.setItem("demoDate", date);
  setTimeout(()=>{
    document.getElementById("demo-section").classList.add("hidden");
    document.getElementById("login-section").classList.remove("hidden");
  },2000);
}

function loginUser() {
  const name = document.getElementById("userName").value.trim();
  if(!name){ alert("Enter your name"); return; }
  studentName = name;
  studentCode = "DA"+Math.floor(Math.random()*90000+10000);
  feePaid = false;
  students.push({name:studentName, code:studentCode, date:localStorage.getItem("demoDate"), status:"Not Paid"});
  localStorage.setItem("sai_students", JSON.stringify(students));
  document.getElementById("login-msg").innerHTML = `Welcome ${name}! Your code is <b>${studentCode}</b>`;
  setTimeout(()=>{
    document.getElementById("login-section").classList.add("hidden");
    document.getElementById("ai-section").classList.remove("hidden");
    document.getElementById("welcomeText").innerText = `Hello ${name}! Ask your doubts here.`;
  },2000);
}

function askAI() {
  const q = document.getElementById("userQuestion").value.trim();
  if(!q) return;
  const chat = document.getElementById("chatBox");

  // Student message
  const userMsg = document.createElement("div");
  userMsg.className="message user clearfix";
  userMsg.innerText=q;
  const userWrap=document.createElement("div");
  userWrap.className="clearfix";
  userWrap.appendChild(userMsg);
  chat.appendChild(userWrap);

  // AI response
  let response="";
  if(!feePaid){
    response="Fees not paid — you can’t use SAI AI now.";
  } else {
    const qLower = q.toLowerCase();
    if(qLower.includes("math")) response="For math, remember: Always break problems into steps and practice regularly.";
    else if(qLower.includes("science")) response="Science tip: Observe carefully, understand concepts, and do experiments whenever possible.";
    else if(qLower.includes("english")) response="English tip: Read stories, practice grammar, and write daily to improve skills.";
    else response="Let me explain: Think step by step and focus on understanding the concept.";
  }

  const aiMsg = document.createElement("div");
  aiMsg.className="message ai clearfix";
  aiMsg.innerText=response;
  const aiWrap=document.createElement("div");
  aiWrap.className="clearfix";
  aiWrap.appendChild(aiMsg);
  chat.appendChild(aiWrap);

  document.getElementById("userQuestion").value="";
  chat.scrollTop = chat.scrollHeight;
}
</script>

</body>
</html>
