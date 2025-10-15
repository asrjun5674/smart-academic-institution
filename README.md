
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

<section id="waitingSection" style="display:none;">
<h2>Waiting for Admin Approval...</h2>
<p id="waitingMsg">Your demo request has been sent. Please wait until admin approves.</p>
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
let demoCheckInterval;

function bookDemo(){
  const name = document.getElementById('studentName').value.trim();
  const demoDate = document.getElementById('demoDate').value;
  if(!name || !demoDate){ alert("Enter name and date!"); return; }

  let demoRequests = JSON.parse(localStorage.getItem('demoRequests')||'[]');
  demoRequests.push({name:name, date:demoDate, approved:false, paid:false});
  localStorage.setItem('demoRequests', JSON.stringify(demoRequests));

  document.getElementById('demoSection').style.display='none';
  document.getElementById('waitingSection').style.display='block';

  // start checking for approval
  studentName = name;
  demoCheckInterval = setInterval(checkApproval, 5000);
}

function checkApproval(){
  let demoRequests = JSON.parse(localStorage.getItem('demoRequests')||'[]');
  let found = demoRequests.find(d=>d.name===studentName);
  if(found && found.approved){
    clearInterval(demoCheckInterval);
    document.getElementById('waitingSection').style.display='none';
    document.getElementById('loginSection').style.display='block';
    alert("Admin approved you! Now you can log in with your name and code.");
  }
}

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
    document.getElementById('loginMsg').innerHTML = "Invalid code or not approved yet.";
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

  chatContent.innerHTML += `<p><b>SAI AI:</b> Thinking...</p>`;

  try {
    const response = await fetch("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer YOUR_OPENAI_API_KEY"
      },
      body: JSON.stringify({
        model: "gpt-5",
        messages: [
          {role:"system",content:"You are SAI AI, a helpful study assistant that helps with all school subjects."},
          {role:"user",content:question}
        ]
      })
    });
    const data = await response.json();
    const answer = data.choices?.[0]?.message?.content || "Sorry, I couldn’t understand that.";
    chatContent.innerHTML += `<p><b>SAI AI:</b> ${answer}</p>`;
  } catch(e){
    chatContent.innerHTML += `<p><b>SAI AI:</b> Error connecting to AI.</p>`;
  }
}
</script>
</body>
</html>
