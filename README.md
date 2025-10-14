
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SAI Student App</title>
<style>
body{font-family:Arial,sans-serif;margin:0;padding:0;background:#f5f5f5;}
header{background:#1ABC9C;color:white;padding:15px;text-align:center;font-size:24px;}
button{cursor:pointer;}
#chatBox{display:none;position:fixed;bottom:90px;right:20px;width:350px;height:450px;background:white;border-radius:10px;box-shadow:0 0 10px rgba(0,0,0,0.2);display:flex;flex-direction:column;}
#chatContent{flex:1;padding:10px;overflow:auto;}
#chatBox input{flex:1;padding:5px;}
#chatBox button{padding:5px;}
#demoSection{padding:20px;}
</style>
</head>
<body>

<header>SAI Student App</header>

<section id="demoSection">
<h2>Book Demo Class</h2>
<input type="text" id="studentName" placeholder="Enter your name">
<button onclick="bookDemo()">Book Demo</button>
<p id="demoMsg"></p>
</section>

<button id="saiBtn" style="position:fixed; bottom:20px; right:20px; width:60px; height:60px; border-radius:50%; background:#1ABC9C; color:white; border:none;">SAI AI</button>

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
const OPENAI_API_KEY = "YOUR_OPENAI_API_KEY"; // replace with your OpenAI API key

// --- BOOK DEMO ---
function bookDemo(){
  const nameInput = document.getElementById('studentName').value.trim();
  if(!nameInput){ alert("Enter your name"); return; }

  let students = JSON.parse(localStorage.getItem('students')||'[]');
  if(!students.includes(nameInput)) students.push(nameInput);
  localStorage.setItem('students', JSON.stringify(students));

  let studentStatus = JSON.parse(localStorage.getItem('studentStatus')||'{}');
  if(!studentStatus[nameInput]) studentStatus[nameInput] = {paid:false, saiStudent:false};
  localStorage.setItem('studentStatus', JSON.stringify(studentStatus));

  document.getElementById('demoMsg').innerHTML = `Demo booked for ${nameInput}. Please wait for admin approval.`;
}

// --- SAI AI CHAT ---
document.getElementById('saiBtn').onclick = ()=>{
  const nameInput = document.getElementById('studentName').value.trim();
  if(!nameInput){ alert("Enter your name to use SAI AI"); return; }
  let studentStatus = JSON.parse(localStorage.getItem('studentStatus')||'{}');
  if(!studentStatus[nameInput] || !studentStatus[nameInput].paid || !studentStatus[nameInput].saiStudent){
    alert("You are not a SAI student or fee not paid. Cannot use SAI AI.");
    return;
  }
  document.getElementById('chatBox').style.display = 'flex';
}

async function sendQuestion(){
  const input = document.getElementById('chatInput');
  const question = input.value.trim();
  if(!question) return;
  const chatContent = document.getElementById('chatContent');
  const nameInput = document.getElementById('studentName').value.trim();

  chatContent.innerHTML += `<p><b>${nameInput}:</b> ${question}</p>`;
  input.value='';
  chatContent.scrollTop = chatContent.scrollHeight;

  // Notify admin
  let aiLogs = JSON.parse(localStorage.getItem('aiLogs')||'[]');
  aiLogs.push({name:nameInput, question:question, time:new Date().toLocaleString()});
  localStorage.setItem('aiLogs', JSON.stringify(aiLogs));

  // Call OpenAI API
  const response = await fetch("https://api.openai.com/v1/chat/completions", {
    method:"POST",
    headers:{
      "Content-Type":"application/json",
      "Authorization": `Bearer ${OPENAI_API_KEY}`
    },
    body: JSON.stringify({
      model:"gpt-3.5-turbo",
      messages:[{role:"user", content: question}],
      max_tokens:500
    })
  });
  const data = await response.json();
  const answer = data.choices[0].message.content;
  chatContent.innerHTML += `<p><b>SAI AI:</b> ${answer}</p>`;
  chatContent.scrollTop = chatContent.scrollHeight;
}
</script>
</body>
</html>
