<!-- Keep all existing HTML the same above -->

<script>
// --- Improved SAI AI ---
let conversation = [];

function toggleAI(){
  if(!studentName){ alert("Please login with your name first!"); return; }
  const popup = document.getElementById('ai-popup');
  popup.style.display = popup.style.display==='block'?'none':'block';
}

async function sendAI(){
  const inputEl = document.getElementById('ai-input');
  const input = inputEl.value.trim();
  if(!input) return;
  inputEl.value = '';

  // Add student message
  const messagesDiv = document.getElementById('ai-messages');
  const studentMsg = document.createElement('div');
  studentMsg.innerHTML = `<b>${studentName}:</b> ${input}`;
  studentMsg.style.background = "#DCF8C6";
  studentMsg.style.padding = "5px";
  studentMsg.style.margin = "5px";
  studentMsg.style.borderRadius = "5px";
  messagesDiv.appendChild(studentMsg);

  conversation.push({role:'user', content: input});

  // Notify admin (simulate via localStorage)
  const aiLogs = JSON.parse(localStorage.getItem('aiLogs')||'[]');
  aiLogs.push({name: studentName, question: input, time: new Date().toLocaleString()});
  localStorage.setItem('aiLogs', JSON.stringify(aiLogs));

  // Call OpenAI API for dynamic answers
  try{
    const response = await fetch("https://api.openai.com/v1/chat/completions", {
      method:"POST",
      headers:{
        "Content-Type":"application/json",
        "Authorization": "Bearer YOUR_OPENAI_API_KEY"
      },
      body: JSON.stringify({
        model: "gpt-3.5-turbo",
        messages: conversation,
        max_tokens: 800
      })
    });

    const data = await response.json();
    let reply = data.choices[0].message.content;

    // Format code snippets nicely
    reply = reply.replace(/```/g, '<pre>').replace(/\n/g,'<br>');

    // Add AI message
    const aiMsg = document.createElement('div');
    aiMsg.innerHTML = `<b>SAI AI:</b><br>${reply}`;
    aiMsg.style.background = "#EAEAEA";
    aiMsg.style.padding = "5px";
    aiMsg.style.margin = "5px";
    aiMsg.style.borderRadius = "5px";
    messagesDiv.appendChild(aiMsg);

    conversation.push({role:'assistant', content: reply});
    messagesDiv.scrollTop = messagesDiv.scrollHeight;

  }catch(err){
    const aiMsg = document.createElement('div');
    aiMsg.innerHTML = `<b>SAI AI:</b> Sorry, I am currently unavailable. Please try later.`;
    aiMsg.style.background = "#EAEAEA";
    aiMsg.style.padding = "5px";
    aiMsg.style.margin = "5px";
    aiMsg.style.borderRadius = "5px";
    messagesDiv.appendChild(aiMsg);
  }
}
</script>
