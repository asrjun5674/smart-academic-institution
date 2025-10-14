
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>SAI - Smart Academic Institutions (Student)</title>
<style>
  :root{--bg:#f0f6fb;--card:#fff;--brand:#0b63c4;--accent:#00b894}
  body{margin:0;font-family:Inter,Segoe UI,Arial;background:var(--bg);color:#102233}
  header{background:linear-gradient(90deg,#0b63c4,#1976d2);color:white;padding:18px;text-align:center;font-size:1.3rem}
  .wrap{max-width:1100px;margin:18px auto;padding:12px}
  .card{background:var(--card);border-radius:12px;padding:14px;box-shadow:0 6px 30px rgba(3,13,26,0.06);margin-bottom:12px}
  label{display:block;margin:8px 0;font-weight:600}
  input[type="text"], input[type="date"], select, textarea{width:100%;padding:10px;border-radius:8px;border:1px solid #d8e6f2}
  button{background:linear-gradient(90deg,var(--accent),#00a3d1);color:#00121a;border:none;padding:10px 14px;border-radius:8px;cursor:pointer;font-weight:700}
  button.ghost{background:#fff;border:1px solid #d8e6f2}
  .muted{color:#6b7a86;font-size:0.95rem}
  .hidden{display:none}
  /* layout */
  .row{display:flex;gap:12px}
  .col{flex:1}
  .sidebar{width:320px}
  @media(max-width:980px){ .row{flex-direction:column}.sidebar{width:100%} }
  /* chat */
  .floating-sai{position:fixed;right:22px;bottom:22px;width:64px;height:64px;border-radius:50%;
              background:linear-gradient(45deg,#00c4a7,#00a3d1);display:flex;align-items:center;justify-content:center;
              color:#00121a;font-weight:800;font-size:18px;cursor:pointer;box-shadow:0 8px 24px rgba(3,13,26,0.25);z-index:999}
  .chat-modal{position:fixed;right:22px;bottom:100px;width:360px;background:white;border-radius:12px;padding:10px;
              box-shadow:0 16px 40px rgba(3,13,26,0.2);z-index:1000}
  .chat-window{height:260px;overflow:auto;padding:8px;border-radius:8px;border:1px solid #eef6fb;background:#fbfeff}
  .msg{margin:8px 0;padding:8px;border-radius:10px;max-width:78%}
  .msg.user{margin-left:auto;background:#d9e8ff;color:#052a66}
  .msg.ai{margin-right:auto;background:#e9f7f5;color:#023a3b}
  /* worksheet */
  .worksheet q-list{display:block}
  .question {background:#fcfeff;border:1px solid #eef6fb;padding:10px;border-radius:8px;margin:8px 0}
  .btn-small{padding:6px 10px;border-radius:8px;border:none;background:#007bff;color:white;cursor:pointer}
  .answers{background:#f7fbfc;padding:10px;border-radius:8px;border:1px solid #e6f3f8}
  .badge{display:inline-block;padding:6px 10px;border-radius:999px;background:#eef7ff;color:#023a69;font-weight:700}
</style>
</head>
<body>
<header>SAI — Smart Academic Institutions (Student)</header>
<div class="wrap">

  <!-- DEMO -->
  <div class="card" id="demoCard">
    <h3>🎓 Book Demo Class</h3>
    <p class="muted">Timing: <b>4:30 PM — 6:30 PM</b> (choose any date from 2025)</p>
    <label>Choose date</label>
    <input type="date" id="demoDate" min="2025-01-01">
    <div style="margin-top:12px">
      <button onclick="bookDemo()">Book Demo</button>
      <button class="ghost" onclick="showLogin()">Already booked? Login</button>
    </div>
    <p id="demoMsg" class="muted"></p>
  </div>

  <!-- LOGIN -->
  <div class="card hidden" id="loginCard">
    <h3>Join SAI — Login</h3>
    <label>Your name</label>
    <input id="stuName" placeholder="Full name">
    <label>Select Grade</label>
    <select id="stuGrade">
      <option value="">-- Grade --</option>
      <!-- 1..10 -->
      <script>for(let g=1;g<=10;g++){document.write(`<option value="${g}">Grade ${g}</option>`)};</script>
    </select>
    <div style="margin-top:12px">
      <button onclick="studentLogin()">Create & Continue</button>
    </div>
    <p id="loginMsg" class="muted"></p>
  </div>

  <!-- MAIN UI: worksheets + AI + Meet -->
  <div class="card hidden" id="mainCard">
    <div style="display:flex;justify-content:space-between;align-items:center">
      <div>
        <h3 id="welcomeTitle">Welcome</h3>
        <div class="muted">Status: <span id="statusBadge" class="badge">Not Paid</span> — Code: <span id="studentCode">—</span></div>
      </div>
      <div class="muted">Demo date: <span id="demoDateShow">—</span></div>
    </div>

    <hr style="margin:12px 0">

    <div class="row">
      <!-- Left: worksheets -->
      <div class="col">
        <div style="display:flex;gap:8px;align-items:center;">
          <label style="margin:0">Select Grade</label>
          <select id="gradeSelect" onchange="renderSubjects()"></select>
          <label style="margin:0">Subject</label>
          <select id="subjectSelect" onchange="renderWorksheet()"></select>
        </div>

        <div id="worksheetArea" style="margin-top:12px"></div>
      </div>

      <!-- Right: meet + small info -->
      <div class="sidebar">
        <div class="card">
          <h4>📹 SAI Meet</h4>
          <p class="muted">Room: <b>SAI2025MEET</b></p>
          <div style="display:flex;gap:8px;margin-bottom:8px">
            <button onclick="joinMeet()">Join Meet</button>
            <button class="ghost" onclick="leaveMeet()">Leave</button>
          </div>
          <div id="meetContainer"></div>
          <p class="muted" style="margin-top:6px">Phone join: <a href="tel:+911234567896">+91 1234567896</a></p>
        </div>

        <div class="card" style="margin-top:12px">
          <h4>My Bookings</h4>
          <div class="muted">Name: <span id="showName">—</span></div>
          <div class="muted">Code: <span id="showCode">—</span></div>
          <div class="muted">Grade: <span id="showGrade">—</span></div>
          <div style="margin-top:8px"><button onclick="openChatModal()">Ask SAI (Help)</button></div>
        </div>
      </div>
    </div>
  </div>
</div>

<!-- floating SAI button -->
<div class="floating-sai" title="Ask SAI" onclick="openChatModal()">SAI</div>

<!-- Chat / Help modal -->
<div id="chatModal" class="chat-modal hidden" role="dialog" aria-hidden="true">
  <h4 style="margin:6px 0">SAI Help & Chat</h4>
  <div id="chatWindow" class="chat-window"></div>
  <div style="margin-top:8px;display:flex;gap:8px">
    <input id="helpInput" placeholder="Ask for help or 'give me my code'">
    <button onclick="sendHelp()">Send</button>
  </div>
  <div style="margin-top:8px"><button class="ghost" onclick="closeChatModal()">Close</button></div>
</div>

<script>
/* Data structures in localStorage:
 - sai_students: [{name,code,grade,date,status,isSAIStudent,joinedAt}]
 - sai_chats: { code: [ {from:'user'|'ai'|'admin', text, ts} ] }
 - sai_help_requests: [ {id, code, name, grade, subject, qid, questionText, status:'open'|'answered', adminReply } ]
 - sai_meet_joins: [{code,name,ts}]
 - worksheets: embedded in code below
*/

// --- worksheets: small sample Qs for each subject/grade
const WORKSHEETS = (function(){
  const common = {
    1: { subjects:['English','Mathematics','Science','Social Studies','General Knowledge'] },
    2: { subjects:['English','Mathematics','Science','Social Studies','General Knowledge'] },
    3: { subjects:['English','Mathematics','Science','Social Studies','General Knowledge'] },
    4: { subjects:['English','Mathematics','Science','Social Studies','General Knowledge'] },
    5: { subjects:['English','Mathematics','Science','Social Studies','General Knowledge'] },
    6: { subjects:['English','Mathematics','Physics','Chemistry','Biology','History & Civics','Geography'] },
    7: { subjects:['English','Mathematics','Physics','Chemistry','Biology','History & Civics','Geography'] },
    8: { subjects:['English','Mathematics','Physics','Chemistry','Biology','History & Civics','Geography'] },
    9: { subjects:['English','Mathematics','Physics','Chemistry','Biology','History & Civics','Geography'] },
    10:{ subjects:['English','Mathematics','Physics','Chemistry','Biology','History & Civics','Geography'] }
  };
  // For brevity include 3 questions per subject with answers and step-by-step answers for admin to send.
  function q(id,text,answer,steps){ return {id,text,answer,steps}; }
  const sheets = {};
  for(let g=1; g<=10; g++){
    sheets[g] = {};
    common[g].subjects.forEach(sub=>{
      sheets[g][sub]=[
        q('q1', `Sample Q1 for Grade ${g} ${sub}`, `Ans1-${g}-${sub}`, `Step-by-step answer for Q1 of ${sub}`),
        q('q2', `Sample Q2 for Grade ${g} ${sub}`, `Ans2-${g}-${sub}`, `Step-by-step answer for Q2 of ${sub}`),
        q('q3', `Sample Q3 for Grade ${g} ${sub}`, `Ans3-${g}-${sub}`, `Step-by-step answer for Q3 of ${sub}`),
      ];
    });
  }
  return sheets;
})();

// localStorage helpers
const LS = {
  students(){return JSON.parse(localStorage.getItem('sai_students')||'[]')},
  setStudents(v){localStorage.setItem('sai_students',JSON.stringify(v))},
  chats(){return JSON.parse(localStorage.getItem('sai_chats')||'{}')},
  setChats(v){localStorage.setItem('sai_chats',JSON.stringify(v))},
  help(){return JSON.parse(localStorage.getItem('sai_help_requests')||'[]')},
  setHelp(v){localStorage.setItem('sai_help_requests',JSON.stringify(v))},
  meets(){return JSON.parse(localStorage.getItem('sai_meet_joins')||'[]')},
  setMeets(v){localStorage.setItem('sai_meet_joins',JSON.stringify(v))}
};

// small util
function generateCode(){ return 'DA'+Math.floor(Math.random()*90000+10000); }
function nowTS(){ return Date.now(); }
function el(id){return document.getElementById(id)}

// --- Demo booking ---
function bookDemo(){
  const date = el('demoDate').value;
  if(!date){ alert('Pick a date'); return; }
  localStorage.setItem('sai_demo_date', date);
  el('demoMsg').innerText = `🎉 Demo booked for ${date}. Click "Already booked? Login" or continue.`;
  setTimeout(()=>showLogin(),600);
}
function showLogin(){ el('loginCard').classList.remove('hidden'); }

// --- Student create/login ---
let current = null;
function studentLogin(){
  const name = el('stuName').value.trim();
  const grade = el('stuGrade').value;
  const date = localStorage.getItem('sai_demo_date') || el('demoDate').value;
  if(!name || !grade){ alert('Enter name and grade'); return; }
  const students = LS.students();
  let uniqueName = name;
  // avoid identical names: append count
  let count=1; while(students.find(s=>s.name===uniqueName)){ count++; uniqueName = name + ' ' + count; }
  const code = generateCode();
  const student = {name:uniqueName, code, grade, date, status:'Not Paid', isSAIStudent:false, joinedAt:null};
  students.push(student); LS.setStudents(students);
  // ensure chat object
  const chats = LS.chats(); chats[code]=chats[code]||[]; LS.setChats(chats);
  current = student;
  postLoginUI();
  // notify admin (storage event)
  localStorage.setItem('sai_event','student_created_'+nowTS());
}

function postLoginUI(){
  el('demoCard').classList.add('hidden'); el('loginCard').classList.add('hidden'); el('mainCard').classList.remove('hidden');
  el('welcomeTitle').innerText = `Welcome, ${current.name}`;
  el('statusBadge').innerText = current.status;
  el('studentCode').innerText = current.code;
  el('demoDateShow').innerText = current.date || '—';
  el('showName').innerText = current.name; el('showCode').innerText = current.code; el('showGrade').innerText = 'Grade '+current.grade;
  // populate grade/subject selectors
  const gradeSel = el('gradeSelect'); gradeSel.innerHTML='';
  for(let g=1;g<=10;g++){ gradeSel.innerHTML += `<option value="${g}">Grade ${g}</option>`; }
  gradeSel.value = current.grade;
  renderSubjects();
  // listen storage updates
  window.addEventListener('storage', onStorageEvent);
  renderChatWindow();
}

// --- subjects & worksheets rendering
function renderSubjects(){
  const g = el('gradeSelect').value || current.grade;
  const subs = Object.keys(WORKSHEETS[g]);
  const ss = el('subjectSelect'); ss.innerHTML=''; subs.forEach(s=> ss.innerHTML += `<option value="${s}">${s}</option>`);
  renderWorksheet();
}
function renderWorksheet(){
  const g = el('gradeSelect').value || current.grade;
  const sub = el('subjectSelect').value;
  const list = WORKSHEETS[g][sub] || [];
  const area = el('worksheetArea'); area.innerHTML = `<h4>Grade ${g} — ${sub}</h4>`;
  list.forEach((q, idx)=>{
    area.innerHTML += `
      <div class="question" id="q_${q.id}_${idx}">
        <div><b>Q${idx+1}:</b> ${q.text}</div>
        <div style="margin-top:8px">
          <button class="btn-small" onclick="requestHelp('${q.id}','${g}','${sub}',${idx})">Ask SAI for help on this Q</button>
          <button class="btn-small ghost" onclick="showAnswer('${g}','${sub}',${idx})">Show Answer (for self-check)</button>
        </div>
        <div id="ans_${q.id}_${idx}" class="answers hidden"></div>
      </div>`;
  });
}

// show answer locally (student can view; admin will also have full answers)
function showAnswer(g,sub,idx){
  const q = WORKSHEETS[g][sub][idx];
  const elAns = document.getElementById(`ans_${q.id}_${idx}`);
  elAns.innerHTML = `<b>Answer:</b> ${q.answer} <br><b>Steps:</b> ${q.steps}`;
  elAns.classList.remove('hidden');
}

// --- help request flow (from student)
function requestHelp(qid, grade, subject, qidx){
  if(!current){ alert('Please login first'); return; }
  const q = WORKSHEETS[grade][subject][qidx];
  const reqs = LS.help();
  const req = {id:'HR'+nowTS(), code:current.code, name:current.name, grade, subject, qid:qid+'_'+qidx, qtext:q.text, status:'open', adminReply:null, ts:nowTS()};
  reqs.push(req); LS.setHelp(reqs);
  // post a chat message so admin preview sees it
  const chats = LS.chats(); chats[current.code] = chats[current.code]||[]; chats[current.code].push({from:'user', text:`Help requested on ${subject} - ${q.text}`, ts:nowTS()}); LS.setChats(chats);
  localStorage.setItem('sai_event','help_req_'+nowTS());
  alert('Help request sent to Admin. They will reply with step-by-step solution.');
  renderChatWindow();
}

// --- chat modal
function openChatModal(){ el('chatModal').classList.remove('hidden'); renderChatWindow(); }
function closeChatModal(){ el('chatModal').classList.add('hidden'); }
function sendHelp(){
  const text = el('helpInput').value.trim();
  if(!text){ alert('Type your help request'); return; }
  // push to chats and help queue as generic question
  const chats = LS.chats(); chats[current.code]=chats[current.code]||[]; chats[current.code].push({from:'user', text, ts:nowTS()}); LS.setChats(chats);
  // also create a help request entry (admin will reply)
  const reqs = LS.help(); reqs.push({id:'HR'+nowTS(), code:current.code, name:current.name, grade:current.grade, subject:'General', qid:null, qtext:text, status:'open', adminReply:null, ts:nowTS()}); LS.setHelp(reqs);
  localStorage.setItem('sai_event','help_sent_'+nowTS());
  el('helpInput').value=''; renderChatWindow(); alert('Help request sent to Admin');
}

// render chat window for student
function renderChatWindow(){
  const cw = el('chatWindow');
  cw.innerHTML='';
  if(!current) return;
  const chats = LS.chats()[current.code] || [];
  chats.forEach(m=>{
    const d = document.createElement('div'); d.className = 'msg ' + (m.from==='user' ? 'user' : 'ai'); d.innerText = m.text; cw.appendChild(d);
  });
  // also show answered help requests (admin replies saved to help list and also appended to chats by admin)
  const helpList = LS.help().filter(h=>h.code===current.code && h.status==='answered');
  helpList.forEach(h=>{
    const d = document.createElement('div'); d.className='msg ai'; d.innerText = 'Admin solution: ' + h.adminReply; cw.appendChild(d);
  });
  cw.scrollTop = cw.scrollHeight;
}

// --- Meet
function joinMeet(){
  if(!current){ alert('Login first'); return; }
  // log join
  const joins = LS.meets(); joins.push({code:current.code, name:current.name, ts:nowTS()}); LS.setMeets(joins);
  localStorage.setItem('sai_event','meet_join_'+nowTS());
  // embed jitsi iframe
  el('meetContainer').innerHTML = `<iframe class="meet-frame" src="https://meet.jit.si/SAI2025MEET" allow="camera; microphone; fullscreen" style="width:100%;height:360px;border:1px solid #e6f3f8;border-radius:8px"></iframe>`;
  // update joinedAt in students
  const students = LS.students(); const s = students.find(x=>x.code===current.code); if(s){ s.joinedAt = nowTS(); LS.setStudents(students); localStorage.setItem('sai_event','student_joined_'+nowTS()); }
}
function leaveMeet(){ el('meetContainer').innerHTML=''; }

// storage event to pick up admin replies/help answers or status change
function onStorageEvent(e){
  // re-load students and update badge
  if(!current) return;
  const students = LS.students(); const s = students.find(x=>x.code===current.code);
  if(s){ current = s; el('statusBadge').innerText = s.status; el('statusBadge').style.background = s.status==='Paid' ? '#e8fffb' : '#fff2f0'; renderChatWindow(); }
  // if admin answered a help request, it will be in LS.help with status answered. renderChatWindow will show it.
}

// initialize if already created student on this browser
(function initRestore(){
  const students = LS.students();
  if(students.length){
    // find last student created locally
    const last = students[students.length-1];
    // restore session for convenience
    current = last;
    postLoginUI();
  } else {
    // pre-fill grade selector in login card for convenience
    for(let g=1;g<=10;g++){ el('stuGrade').innerHTML += `<option value="${g}">Grade ${g}</option>`; }
  }
})();
</script>
</body>
</html>
