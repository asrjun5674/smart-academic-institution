
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SAI Student App</title>
<style>
body{font-family:Arial,sans-serif;background:#f5f5f5;margin:0;padding:0;}
header{background:#2E86C1;color:white;padding:15px;text-align:center;font-size:24px;}
section{padding:20px;}
button{cursor:pointer;border:none;padding:10px 20px;border-radius:5px;}
#saiBtn{position:fixed;bottom:20px;right:20px;width:60px;height:60px;border-radius:50%;background:#E74C3C;color:white;font-weight:bold;}
#chatBox{display:none;position:fixed;bottom:90px;right:20px;width:350px;height:450px;background:white;border-radius:10px;box-shadow:0 0 10px rgba(0,0,0,0.2);display:flex;flex-direction:column;}
#chatContent{flex:1;padding:10px;overflow:auto;}
#chatInput{flex:1;padding:5px;}
iframe{border:none;border-radius:10px;}
</style>
</head>
<body>

<header>SAI Student App</header>

<section id="demoSection">
<h2>Book Demo Class</h2>
<input type="text" id="studentName" placeholder="Enter your name">
<input type="datetime-local" id="demoDate">
<button onclick="bookDemo()">Book Demo</button>
<p id="demoMsg"></p>
</section>

<section id="loginSection" style="display:none;">
<h2>Enter Access Code</h2>
<input type="text" id="codeInput" placeholder="Enter your SAI Code">
<button onclick="login()">Login</button>
<p id="loginMsg"></p>
</section>

<section id="studentPanel" style="display:none;">
<h2>Welcome, <span id="studentDisplay"></span></h2>

<h3>SAI Meet</h3>
<iframe src="https://meet.jit.si/SAI2025MEET" width="100%" height="400px" allow="camera; microphone; fullscreen"></iframe>
</section>

<button id="saiBtn" onclick="toggleChat()">SAI AI</button>
<div id="chatBox">
  <div id="chatContent"></div>
  <div style="display:flex;">
    <input type="text" id="chatInput" placeholder="Ask SAI AI...">
    <button onclick="sendQuestion()">Send</button>
  </div>
</div>

<script>
let studentName = "";
let studentCode = "";

// --- Book Demo ---
function bookDemo(){
  const name = document.getElementById('studentName').value.trim();
  const demoDate = document.getElementById('demoDate').value;
  if(!name || !demoDate){ alert("Enter name and date!"); return; }

  let demoRequests = JSON.parse(localStorage.getItem('demoRequests')||'[]');
  demoRequests.push({name:name, date:demoDate});
  localStorage.setItem('demoRequests', JSON.stringify(demoRequests));

  document.getElementById('demoMsg').innerText = `Demo booked for ${name} on ${demoDate}. Wait for admin approval.`;
  document.getElementById('demoSection').style.display='none';
  document.getElementById('loginSection').style.display='block';
}

// --- Login ---
function login(){
  const code = document.getElementById('codeInput').value.trim();
  let studentCodes = JSON.parse(localStorage.getItem('studentCodes')||'{}');
  let found = Object.entries(studentCodes).find(([n,c]) => c===code);
  if(found){
    studentName = found[0];
    studentCode = code;
    document.getElementById('studentDisplay').innerText = studentName;
    document.getElementById('loginSection').style.display='none';
    document.getElementById('studentPanel').style.display='block';
  } else {
    document.getElementById('loginMsg').innerText = "Invalid code. Contact admin.";
  }
}

// --- SAI AI Chat ---
function toggleChat(){
  if(!studentName){ alert("Please login first!"); return; }
  let box = document.getElementById('chatBox');
  box.style.display = box.style.display==='flex'?'none':'flex';
}

async function sendQuestion(){
  const input = document.getElementById('chatInput');
  const question = input.value.trim();
  if(!question) return;
  const chatContent = document.getElementById('chatContent');
  chatContent.innerHTML += `<p><b>${studentName}:</b> ${question}</p>`;
  input.value='';

  // --- Save question to logs ---
  let aiLogs = JSON.parse(localStorage.getItem('aiLogs')||'[]');
  aiLogs.push({name:studentName, question:question, time:new Date().toLocaleString()});
  localStorage.setItem('aiLogs', JSON.stringify(aiLogs));

  // --- Call OpenAI GPT-5 API ---
  try{
    let response = await fetch('https://api.openai.com/v1/responses', {
      method:'POST',
      headers:{
        'Content-Type':'application/json',
        'Authorization':'Bearer YOUR_OPENAI_API_KEY'
      },
      body: JSON.stringify({
        model: "gpt-5-mini",
        input: question
      })
    });
    let data = await response.json();
    let answer = data.output_text || "SAI AI could not respond.";
    chatContent.innerHTML += `<p><b>SAI AI:</b> ${answer}</p>`;
  }catch(err){
    chatContent.innerHTML += `<p><b>SAI AI:</b> Error contacting AI server.</p>`;
    console.error(err);
  }
}
</script>
</body>
</html>
