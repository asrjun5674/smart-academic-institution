
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SAI AI Student App</title>
<style>
body {font-family:Arial,sans-serif;margin:0;padding:0;background:#f4f6f7;}
header {background:#2471A3;color:white;padding:15px;text-align:center;font-size:24px;}
button {cursor:pointer;border:none;border-radius:5px;padding:10px 15px;}
#saiBtn {position:fixed;bottom:20px;right:20px;background:#E74C3C;color:white;border-radius:50%;width:70px;height:70px;font-weight:bold;font-size:14px;}
#chatBox {display:none;position:fixed;bottom:100px;right:20px;width:350px;height:450px;background:white;border-radius:10px;box-shadow:0 0 10px rgba(0,0,0,0.2);display:flex;flex-direction:column;}
#chatContent {flex:1;padding:10px;overflow:auto;}
#chatInput {flex:1;padding:5px;}
</style>
</head>
<body>

<header>SAI - Smart Academic Institutions (Student)</header>

<section style="padding:20px;">
  <h2>Book Demo Class</h2>
  <input type="text" id="studentName" placeholder="Enter your name">
  <button onclick="bookDemo()">Book Demo</button>
  <p id="demoMsg"></p>

  <div id="loginSection" style="display:none;">
    <h3>Login with your SAI Code</h3>
    <input type="text" id="studentCode" placeholder="Enter your SAI Code">
    <button onclick="login()">Login</button>
    <p id="loginMsg"></p>
  </div>
</section>

<button id="saiBtn" onclick="openChat()">SAI AI</button>
<div id="chatBox">
  <div id="chatContent"></div>
  <div style="display:flex;">
    <input type="text" id="chatInput" placeholder="Ask SAI AI...">
    <button onclick="sendQuestion()">Send</button>
  </div>
</div>

<h2 style="padding:20px;">SAI Meet</h2>
<iframe src="https://meet.jit.si/SAI2025MEET" width="100%" height="400px" allow="camera; microphone; fullscreen"></iframe>

<script>
const OPENAI_API_KEY = "YOUR_OPENAI_API_KEY"; // replace with your key
let loggedStudent = null;

// --- BOOK DEMO ---
function bookDemo(){
  const name = document.getElementById("studentName").value.trim();
  if(!name){ alert("Enter your name"); return; }

  let students = JSON.parse(localStorage.getItem("students")||"[]");
  if(!students.find(s=>s.name===name)){
    const code = "SAI" + Math.random().toString(36).substring(2,8).toUpperCase();
    students.push({name, code, paid:false});
    localStorage.setItem("students", JSON.stringify(students));
    alert(`Demo booked for ${name}. Your unique SAI code will be given by Admin.`);
  }
  document.getElementById("demoMsg").innerText = "Demo booked successfully. Please wait for Admin approval.";
  document.getElementById("loginSection").style.display = "block";
}

// --- LOGIN ---
function login(){
  const code = document.getElementById("studentCode").value.trim();
  const students = JSON.parse(localStorage.getItem("students")||"[]");
  const student = students.find(s=>s.code===code && s.paid);
  if(student){
    loggedStudent = student;
    document.getElementById("loginMsg").innerHTML = `Welcome ${student.name}! You can now use <b>SAI AI</b>.`;
  }else{
    document.getElementById("loginMsg").innerText = "Invalid or unpaid code. Contact Admin.";
  }
}

// --- SAI AI CHAT ---
function openChat(){
  if(!loggedStudent){ alert("Please login first."); return; }
  document.getElementById("chatBox").style.display = "flex";
}

async function sendQuestion(){
  const input = document.getElementById("chatInput");
  const question = input.value.trim();
  if(!question) return;

  const chatContent = document.getElementById("chatContent");
  chatContent.innerHTML += `<p><b>${loggedStudent.name}:</b> ${question}</p>`;
  input.value='';

  // Notify Admin
  let aiLogs = JSON.parse(localStorage.getItem("aiLogs")||"[]");
  aiLogs.push({name:loggedStudent.name, question, time:new Date().toLocaleString()});
  localStorage.setItem("aiLogs", JSON.stringify(aiLogs));

  // OpenAI API call
  chatContent.innerHTML += `<p><i>SAI AI is thinking...</i></p>`;
  const response = await fetch("https://api.openai.com/v1/chat/completions", {
    method:"POST",
    headers:{
      "Content-Type":"application/json",
      "Authorization":`Bearer ${OPENAI_API_KEY}`
    },
    body:JSON.stringify({
      model:"gpt-4o-mini",
      messages:[{role:"system",content:"You are SAI AI, a helpful AI tutor by Smart Academic Institutions."},{role:"user",content:question}]
    })
  });
  const data = await response.json();
  const answer = data.choices?.[0]?.message?.content || "Sorry, I couldn’t fetch an answer.";
  chatContent.innerHTML += `<p><b>SAI AI:</b> ${answer}</p>`;
  chatContent.scrollTop = chatContent.scrollHeight;
}
</script>
</body>
</html>
