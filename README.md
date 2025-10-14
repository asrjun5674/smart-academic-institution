
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>SAI - Smart Academic Institutions (Student)</title>
<style>
  :root{--bg:#f0f4f8;--card:#fff;--brand:#007bff;--accent:#00c4a7;}
  body{margin:0;font-family:Inter,Segoe UI,Arial;background:var(--bg);color:#16324f}
  header{background:linear-gradient(90deg,#0f3d91,#1b6fb3);color:white;padding:18px;text-align:center;font-size:1.4rem}
  .wrap{max-width:980px;margin:20px auto;padding:16px}
  .card{background:var(--card);border-radius:12px;padding:16px;box-shadow:0 8px 30px rgba(3,13,26,0.08);margin-bottom:16px}
  label{display:block;margin:8px 0;font-weight:600}
  input[type="text"], input[type="date"]{width:100%;padding:10px;border-radius:8px;border:1px solid #d8e6f2}
  button{background:linear-gradient(90deg,var(--accent),#00a3d1);color:#00121a;border:none;padding:10px 14px;border-radius:8px;cursor:pointer;font-weight:700}
  button.ghost{background:transparent;border:1px solid #d8e6f2;color:#16324f}
  .muted{color:#6b7a86;font-size:0.95rem}
  .hidden{display:none}
  /* chat */
  .ai-box{display:flex;flex-direction:column;gap:8px}
  .chat-window{height:320px;overflow:auto;padding:12px;border-radius:10px;background:#f7fbfd;border:1px solid #e6f3f8}
  .msg-row{display:flex;margin:6px 0}
  .msg.ai{justify-content:flex-start}
  .msg.user{justify-content:flex-end}
  .bubble{max-width:78%;padding:10px 12px;border-radius:14px;line-height:1.3}
  .bubble.ai{background:#e9f7f5;color:#013a2f;border:1px solid #d7f0ea}
  .bubble.user{background:#d9e8ff;color:#032a66}
  .clearfix::after{content:"";display:table;clear:both}
  /* meet */
  .meet-frame{width:100%;height:520px;border-radius:10px;border:1px solid #e6f3f8}
  .row{display:flex;gap:12px}
  @media(max-width:820px){ .row{flex-direction:column} .meet-frame{height:360px} }
  .small{font-size:0.9rem;color:#4b6b7a}
  .badge{display:inline-block;padding:6px 10px;border-radius:999px;background:#eef7ff;color:#023a69;font-weight:700}
</style>
</head>
<body>
  <header>SAI — Smart Academic Institutions (Student)</header>
  <div class="wrap">

    <!-- DEMO BOOKING -->
    <div class="card" id="demoCard">
      <h3>🎓 Book Demo Class</h3>
      <p class="muted">Available timing: <b>4:30 PM — 6:30 PM</b> (choose any date from 2025)</p>

      <label for="demoDate">Select Date</label>
      <input type="date" id="demoDate" min="2025-01-01">

      <div style="margin-top:12px">
        <button onclick="bookDemo()">Book Demo Date & Time</button>
        <button class="ghost" onclick="loadDemo()">Already booked? Show login</button>
      </div>
      <p id="demoMsg" class="small"></p>
    </div>

    <!-- LOGIN AFTER BOOKING -->
    <div class="card hidden" id="loginCard">
      <h3>Join SAI — Login</h3>
      <p class="small">Enter your name to create an account and join the demo.</p>
      <label for="stuName">Your name</label>
      <input type="text" id="stuName" placeholder="e.g. Ravi Kumar">
      <div style="margin-top:12px"><button onclick="studentLogin()">Login & Create</button></div>
      <p id="loginMsg" class="small"></p>
    </div>

    <!-- AI CHAT + MEET -->
    <div class="card hidden" id="mainCard">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div>
          <h3 id="welcomeTitle">Welcome</h3>
          <div class="small">Status: <span id="statusBadge" class="badge">Not Paid</span> • Code: <span id="studentCode">—</span></div>
        </div>
        <div class="small">Demo date: <span id="demoDateShow">—</span></div>
      </div>

      <hr style="margin:12px 0">

      <div class="row">
        <!-- CHAT -->
        <div style="flex:1" class="card" id="chatCard">
          <h4>🤖 SAI AI</h4>
          <div id="chatWindow" class="chat-window"></div>
          <div style="display:flex;gap:8px;margin-top:8px">
            <input id="chatInput" type="text" placeholder="Ask SAI (math, science, english...)"
                   style="flex:1;padding:10px;border-radius:8px;border:1px solid #dfeef8">
            <button onclick="sendChat()">Send</button>
          </div>
          <div class="small" style="margin-top:8px">Tip: Ask "give me my code" — SAI will give it only if admin marked you Paid.</div>
        </div>

        <!-- MEET -->
        <div style="width:420px" class="card">
          <h4>📹 SAI Meet (room: <b>SAI2025MEET</b>)</h4>
          <div style="display:flex;gap:8px;margin-bottom:8px">
            <button onclick="openMeet()">Join Meet</button>
            <button class="ghost" onclick="leaveMeet()">Leave</button>
          </div>
          <div class="small">Phone join: <a href="tel:+911234567896">+91 1234567896</a></div>
          <div id="meetContainer" style="margin-top:12px"></div>
        </div>
      </div>
    </div>

  </div>

<script>
/* Shared storage keys:
  - sai_students: array of student objects {name, code, date, status, isSAIStudent, joinedAt}
  - sai_chats: object mapping code -> [{from:'user'|'ai'|'admin', text, ts}]
  - sai_meet_joins: array of join events {code,name,ts}
*/

// helpers
const LS = {
  students(){return JSON.parse(localStorage.getItem('sai_students')||'[]')},
  setStudents(v){localStorage.setItem('sai_students',JSON.stringify(v))},
  chats(){return JSON.parse(localStorage.getItem('sai_chats')||'{}')},
  setChats(v){localStorage.setItem('sai_chats',JSON.stringify(v))},
  meetJoins(){return JSON.parse(localStorage.getItem('sai_meet_joins')||'[]')},
  setMeetJoins(v){localStorage.setItem('sai_meet_joins',JSON.stringify(v))}
};

let currentStudent = null; // {name,code,date,status}

// --- Demo booking ---
function bookDemo(){
  const date = document.getElementById('demoDate').value;
  if(!date){ alert('Please pick a date'); return; }
  localStorage.setItem('sai_demo_date', date);
  document.getElementById('demoMsg').innerText = `🎉 Demo booked for ${date}. Now click "Already booked? Show login" or wait to continue.`;
  // show login after short delay
  setTimeout(()=>{ document.getElementById('loginCard').classList.remove('hidden'); }, 700);
}

// if student already booked on this browser
function loadDemo(){
  const date = localStorage.getItem('sai_demo_date');
  if(date){ document.getElementById('demoMsg').innerText = `Demo date: ${date}`; document.getElementById('loginCard').classList.remove('hidden'); }
  else alert('No demo date found. Please pick a date first.');
}

// --- Student login/create ---
function studentLogin(){
  const name = document.getElementById('stuName').value.trim();
  const date = localStorage.getItem('sai_demo_date') || document.getElementById('demoDate').value;
  if(!name){ alert('Enter your name'); return; }
  const students = LS.students();
  // prevent duplicate names: append a number if exists
  const exists = students.find(s=>s.name.toLowerCase()===name.toLowerCase());
  let uniqueName=name;
  if(exists){
    let n=2; while(students.find(s=>s.name.toLowerCase()=== (name+' '+n).toLowerCase())) n++;
    uniqueName = name + ' ' + n;
  }
  const code = generateCode();
  const student = {name:uniqueName, code, date:date || null, status:'Not Paid', isSAIStudent:false, joinedAt:null};
  students.push(student);
  LS.setStudents(students);
  // ensure chat object for this code
  const chats = LS.chats();
  if(!chats[code]) chats[code]=[];
  LS.setChats(chats);

  currentStudent = student;
  showMainUI();
  // notify admin via storage event by updating a small key
  localStorage.setItem('sai_event','student_created_'+Date.now());
}

function generateCode(){ return 'DA'+Math.floor(Math.random()*90000+10000); }

// --- show main UI after login ---
function showMainUI(){
  document.getElementById('demoCard').classList.add('hidden');
  document.getElementById('loginCard').classList.add('hidden');
  document.getElementById('mainCard').classList.remove('hidden');
  document.getElementById('welcomeTitle').innerText = `Welcome, ${currentStudent.name}`;
  document.getElementById('studentCode').innerText = currentStudent.code;
  document.getElementById('demoDateShow').innerText = currentStudent.date || '—';
  updateStatusBadge();
  renderChat();
  // listen for storage updates to update status (paid change by admin)
  window.addEventListener('storage', onStorage);
}

// --- status badge update from storage
function updateStatusBadge(){
  const students = LS.students();
  const s = students.find(x=>x.code===currentStudent.code);
  if(!s) return;
  currentStudent = s;
  document.getElementById('statusBadge').innerText = s.status + (s.isSAIStudent ? ' • SAI Student' : '');
  document.getElementById('statusBadge').style.background = s.status==='Paid' ? '#e8fffb' : '#fff2f0';
  document.getElementById('statusBadge').style.color = s.status==='Paid' ? '#064b3b' : '#6b2b00';
}

// --- chat logic
function sendChat(){
  sendChat(); // accessible for keyboard or button alias
}
function sendChat(){
  const input = document.getElementById('chatInput');
  const text = input.value.trim();
  if(!text) return;
  pushMessage(currentStudent.code, 'user', text);
  input.value='';
  renderChat();
  // small delay then SAI response
  setTimeout(()=>aiReply(currentStudent.code, text), 700);
}

function pushMessage(code, from, text){
  const chats = LS.chats();
  if(!chats[code]) chats[code]=[];
  chats[code].push({from, text, ts: Date.now()});
  LS.setChats(chats);
  // trigger storage event
  localStorage.setItem('sai_event','chat_'+code+'_'+Date.now());
}

function aiReply(code, userText){
  const students = LS.students();
  const s = students.find(x=>x.code===code);
  let reply = '';
  if(!s || s.status !== 'Paid'){
    reply = "Fees not paid — can't use SAI AI.";
  } else {
    const q = userText.toLowerCase();
    if(q.includes('math')) reply = "Math tip: Break problems into steps, write formulas, and practice example problems.";
    else if(q.includes('science')) reply = "Science tip: Draw diagrams, visualize processes and relate to everyday examples.";
    else if(q.includes('english')) reply = "English tip: Read a short passage, underline new words and write summaries.";
    else if(q.includes('give me my code') || q.includes('my code')) reply = `Your access code is: ${s.code}`;
    else reply = "I will explain step-by-step. Ask follow-up questions if you need clarity.";
  }
  pushMessage(code, 'ai', reply);
  renderChat();
}

// render chat for currentStudent
function renderChat(){
  const chatWindow = document.getElementById('chatWindow');
  chatWindow.innerHTML='';
  const chats = LS.chats();
  const arr = chats[currentStudent.code] || [];
  arr.forEach(m=>{
    const row = document.createElement('div');
    row.className = 'msg-row ' + (m.from==='user' ? 'msg user' : 'msg ai');
    const bubble = document.createElement('div');
    bubble.className = 'bubble ' + (m.from==='user' ? 'user' : 'ai');
    bubble.innerText = m.text;
    row.appendChild(bubble);
    chatWindow.appendChild(row);
  });
  chatWindow.scrollTop = chatWindow.scrollHeight;
}

// --- Meet logic (embed Jitsi) ---
let meetFrame = null;
function openMeet(){
  if(!currentStudent){ alert('Please login first'); return; }
  // record join
  const joins = LS.meetJoins();
  joins.push({code:currentStudent.code, name:currentStudent.name, ts: Date.now()});
  LS.setMeetJoins(joins);
  localStorage.setItem('sai_event','meet_join_'+Date.now());
  // embed iframe
  const container = document.getElementById('meetContainer');
  container.innerHTML = `<iframe class="meet-frame" src="https://meet.jit.si/SAI2025MEET" allow="camera; microphone; fullscreen"></iframe>`;
}

function leaveMeet(){
  const container = document.getElementById('meetContainer');
  container.innerHTML = '';
}

// storage handler for updates from admin (paid etc)
function onStorage(e){
  // any change refresh local state
  renderLocalChanges();
}
function renderLocalChanges(){
  // if student data changed update badge and chat
  const students = LS.students();
  const s = students.find(x=> x.code=== (currentStudent && currentStudent.code));
  if(s){ currentStudent = s; updateStatusBadge(); renderChat(); }
}

// generate student from persisted data if user previously created and reload
(function initFromLS(){
  const students = LS.students();
  // if there is a last created student on this browser, set currentStudent to it
  if(students && students.length){
    const last = students[students.length-1];
    // if last was created on this browser in last hour, restore
    currentStudent = last;
    // show main UI for convenience
    document.getElementById('demoCard').classList.add('hidden');
    document.getElementById('loginCard').classList.add('hidden');
    document.getElementById('mainCard').classList.remove('hidden');
    document.getElementById('welcomeTitle').innerText = `Welcome, ${currentStudent.name}`;
    document.getElementById('studentCode').innerText = currentStudent.code;
    document.getElementById('demoDateShow').innerText = currentStudent.date || '—';
    updateStatusBadge();
    renderChat();
    window.addEventListener('storage', onStorage);
  }
})();
</script>
</body>
</html>
