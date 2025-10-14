
<html lang="en">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>SAI - Student — Smart Academic Institutions</title>
<style>
:root{--bg:#f6fbff;--card:#fff;--brand:#0b63c4;--accent:#00b894}
body{margin:0;font-family:Inter,Segoe UI,Arial;background:var(--bg);color:#112}
header{background:linear-gradient(90deg,#0b63c4,#1976d2);color:white;padding:14px;text-align:center;font-size:1.2rem}
.wrap{max-width:1100px;margin:18px auto;padding:12px}
.card{background:var(--card);border-radius:10px;padding:12px;box-shadow:0 8px 30px rgba(3,13,26,0.06);margin-bottom:12px}
.row{display:flex;gap:12px}
.col{flex:1}
.sidebar{width:320px}
@media(max-width:980px){.row{flex-direction:column}.sidebar{width:100%}}
select,input,textarea{padding:8px;border-radius:6px;border:1px solid #dbeaf7;width:100%}
button{background:linear-gradient(90deg,var(--accent),#00a3d1);color:#00121a;border:none;padding:8px 12px;border-radius:8px;cursor:pointer}
button.ghost{background:#fff;border:1px solid #dbeaf7}
.floating-sai{position:fixed;right:20px;bottom:20px;width:64px;height:64px;border-radius:50%;background:linear-gradient(45deg,#00c4a7,#00a3d1);display:flex;align-items:center;justify-content:center;color:#00121a;font-weight:800;cursor:pointer;z-index:999}
.chat-modal{position:fixed;right:20px;bottom:100px;width:380px;background:white;border-radius:10px;padding:10px;box-shadow:0 12px 36px rgba(3,13,26,0.18);z-index:1000}
.chat-window{height:260px;overflow:auto;padding:8px;border-radius:8px;background:#fbfeff;border:1px solid #e7f6fb}
.msg{margin:8px 0;padding:8px;border-radius:10px}
.msg.user{background:#d9e8ff;color:#052a66;text-align:right}
.msg.ai{background:#e9f7f5;color:#023a3b;text-align:left}
.question{background:#fcfeff;border:1px solid #eef6fb;padding:10px;border-radius:8px;margin:8px 0}
.q-actions{margin-top:8px;display:flex;gap:8px;flex-wrap:wrap}
.badge{display:inline-block;padding:6px 8px;border-radius:999px;background:#eef7ff;color:#023a69;font-weight:700}
.work-area{margin-top:12px}
.meet-frame{width:100%;height:420px;border-radius:8px;border:1px solid #e6f3f8}
.small{font-size:0.9rem;color:#4f6b7a}
</style>
</head>
<body>
<header>SAI — Smart Academic Institutions (Student)</header>
<div class="wrap">

  <div class="card" id="demoCard">
    <h3>Book Demo Class</h3>
    <div class="small">Timing: <b>4:30 PM — 6:30 PM</b></div>
    <label>Pick a date:</label>
    <input id="demoDate" type="date" min="2025-01-01">
    <div style="margin-top:8px">
      <button onclick="bookDemo()">Book Demo</button>
      <button class="ghost" onclick="openLogin()">Already booked? Login</button>
    </div>
    <p id="demoMsg" class="small"></p>
  </div>

  <div class="card hidden" id="loginCard">
    <h3>Join SAI — Login</h3>
    <label>Your full name</label>
    <input id="stuName" placeholder="e.g. Ananya Sharma">
    <label>Grade</label>
    <select id="stuGrade"></select>
    <div style="margin-top:8px"><button onclick="studentLogin()">Create Account</button></div>
    <p id="loginMsg" class="small"></p>
  </div>

  <div class="card hidden" id="mainCard">
    <div style="display:flex;justify-content:space-between;align-items:center">
      <div>
        <h3 id="welcome">Welcome</h3>
        <div class="small">Status: <span id="statusBadge" class="badge">Not Paid</span> • Code: <span id="studentCode">—</span></div>
      </div>
      <div class="small">Demo: <span id="demoShow">—</span></div>
    </div>

    <hr>

    <div class="row">
      <div class="col">
        <div style="display:flex;gap:8px;align-items:center">
          <label style="margin:0">Grade</label>
          <select id="gradeSelect" onchange="onGradeChange()"></select>
          <label style="margin:0">Subject</label>
          <select id="subjectSelect" onchange="renderWorksheet()"></select>
        </div>

        <div id="worksheetArea" class="work-area"></div>
      </div>

      <div class="sidebar">
        <div class="card">
          <h4>SAI Meet</h4>
          <div style="display:flex;gap:8px">
            <button onclick="joinMeet()">Join Meet</button>
            <button class="ghost" onclick="leaveMeet()">Leave</button>
          </div>
          <div id="meetContainer" style="margin-top:8px"></div>
          <div class="small" style="margin-top:8px">Phone join: <a href="tel:+911234567896">+91 1234567896</a></div>
        </div>

        <div class="card" style="margin-top:12px">
          <h4>My Info</h4>
          <div class="small">Name: <span id="infoName">—</span></div>
          <div class="small">Grade: <span id="infoGrade">—</span></div>
          <div style="margin-top:8px"><button onclick="openChatModal()">Ask SAI (help)</button></div>
        </div>
      </div>
    </div>
  </div>
</div>

<!-- floating -->
<div class="floating-sai" title="Ask SAI" onclick="openChatModal()">SAI</div>

<!-- chat modal -->
<div id="chatModal" class="chat-modal hidden" role="dialog" aria-hidden="true">
  <h4 style="margin:6px 0">SAI Help</h4>
  <div id="chatWindow" class="chat-window"></div>
  <div style="margin-top:8px;display:flex;gap:8px">
    <input id="helpInput" placeholder="Ask for hint or 'give me my code'">
    <button onclick="sendHelp()">Send</button>
  </div>
  <div style="margin-top:8px"><button class="ghost" onclick="closeChatModal()">Close</button></div>
</div>

<script>
/* ===== Data & storage helpers =====
Keys:
- sai_students
- sai_chats
- sai_help_requests
- sai_meet_joins
Worksheets generated programmatically (20 Qs per subject).
*/
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

// Subjects by grade
const SUBJECTS = {
  1:['English','Mathematics','Science','Social Studies','General Knowledge'],
  2:['English','Mathematics','Science','Social Studies','General Knowledge'],
  3:['English','Mathematics','Science','Social Studies','General Knowledge'],
  4:['English','Mathematics','Science','Social Studies','General Knowledge'],
  5:['English','Mathematics','Science','Social Studies','General Knowledge'],
  6:['English','Mathematics','Physics','Chemistry','Biology','History & Civics','Geography'],
  7:['English','Mathematics','Physics','Chemistry','Biology','History & Civics','Geography'],
  8:['English','Mathematics','Physics','Chemistry','Biology','History & Civics','Geography'],
  9:['English','Mathematics','Physics','Chemistry','Biology','History & Civics','Geography'],
 10:['English','Mathematics','Physics','Chemistry','Biology','History & Civics','Geography']
};

// deterministic question/answer generator: returns 20 Qs per subject per grade
function generateWorksheet(grade,subject){
  // Use templates per subject; produce exactly 20 items with id, type, text, answer, steps
  const qs=[];
  let i=1;
  while(qs.length<20){
    const id = `${grade}_${subject.replace(/\s+/g,'')}_q${i}`;
    let q={}, n=i;
    if(subject==='Mathematics'){
      // mix: arithmetic, fractions, basic algebra for higher grades
      if(grade<=3){
        q.type='short'; q.text=`What is ${n+1} + ${n+2}?`; q.answer=(n+1)+(n+2);
        q.steps=`Add ${n+1} and ${n+2}: ${q.answer}.`;
      } else if(grade<=5){
        q.type = (i%3===0)?'word':'short';
        q.text = (i%3===0)?`If you have ${n*2} apples and give away ${n}, how many left?`:`Calculate ${n*3} - ${n+1}.`;
        q.answer = (i%3===0)? (n*2 - n) : (n*3 - (n+1));
        q.steps = `Compute: ${q.answer}.`;
      } else {
        // grade 6-10: include multiplication, division, simple algebra
        if(i%4===0){
          q.type='algebra'; q.text=`Solve for x: ${n}x + ${n} = ${n*3}`; q.answer=(n*2)/n; q.answer = 2;
          q.steps=`${n}x + ${n} = ${n*3}. Subtract ${n}: ${n}x = ${n*2}. So x = ${ (n*2)/n }`;
        } else {
          q.type='short'; q.text=`Calculate ${n*6} ÷ ${n+1}`; q.answer=(n*6)/(n+1); q.steps=`Divide ${n*6} by ${n+1}: result ${q.answer}.`;
        }
      }
    } else if(subject==='English'){
      if(grade<=3){
        q.type='short'; q.text=`Fill in the blank: "The cat ___ on the mat." (sit/sits)`; q.answer='sits'; q.steps=`Subject 'cat' (singular) takes 'sits'.`;
      } else if(grade<=5){
        q.type='short'; q.text=`Make plural: ${['box','lady','toy'][i%3]}`; q.answer = (['box','lady','toy'][i%3]==='lady')?'ladies':(['box','lady','toy'][i%3]+'s'); q.steps=`Apply plural rules.`; 
      } else {
        q.type='short'; q.text=`Identify the tense: "She had finished her homework before dinner."`; q.answer='past perfect'; q.steps=`'had finished' indicates past perfect tense.`;
      }
    } else if(subject==='Science' || subject==='Physics' || subject==='Chemistry' || subject==='Biology'){
      // general science templates
      if(grade<=5){
        q.type='short'; q.text=`What does a plant need to make food?`; q.answer='sunlight, water, air, soil'; q.steps=`Photosynthesis needs sunlight, water and CO2.`;
      } else {
        // grade 6+ detailed
        if(subject==='Physics'){
          q.type='short'; q.text=`State the unit of force.`; q.answer='Newton (N)'; q.steps=`Force unit is Newton, defined as kg·m/s².`;
        } else if(subject==='Chemistry'){
          q.type='short'; q.text=`What is the symbol for Sodium?`; q.answer='Na'; q.steps=`Sodium chemical symbol is Na.`;
        } else if(subject==='Biology'){
          q.type='short'; q.text=`What is the basic unit of life?`; q.answer='Cell'; q.steps=`Cell is the structural and functional unit of life.`;
        } else {
          q.type='short'; q.text=`Name one source of energy.`; q.answer='Sun'; q.steps=`Sun is primary source of energy for Earth.`;
        }
      }
    } else if(subject==='Social Studies' || subject==='History & Civics' || subject==='Geography'){
      if(grade<=5){
        q.type='short'; q.text=`Which is the capital city of India?`; q.answer='New Delhi'; q.steps=`New Delhi is the national capital of India.`;
      } else {
        if(subject==='Geography'){
          q.type='short'; q.text=`Define 'latitude'.`; q.answer='distance north/south of Equator'; q.steps=`Latitude measures distance north or south of the Equator in degrees.`;
        } else {
          q.type='short'; q.text=`What is democracy? (short)`; q.answer='government by the people'; q.steps=`Democracy means rule by the people, through elections.`;
        }
      }
    } else if(subject==='General Knowledge'){
      q.type='short'; q.text=`Who is the current Prime Minister of India? (as of 2025)`; q.answer='(Varies)'; q.steps=`Check latest; this is a placeholder for GK.`;
    } else {
      q.type='short'; q.text=`Sample Q`; q.answer='Sample A'; q.steps='Sample steps';
    }

    qs.push({id, type:q.type, text:q.text, answer: String(q.answer), steps:q.steps});
    i++;
  }
  return qs;
}

// ===== App state =====
let currentStudent = null; // {name,code,grade,date,status,...}
function openLogin(){ document.getElementById('loginCard').classList.remove('hidden'); }
function bookDemo(){
  const dt = document.getElementById('demoDate').value;
  if(!dt){ alert('Pick a date'); return;}
  localStorage.setItem('sai_demo_date', dt);
  document.getElementById('demoMsg').innerText = `Demo booked on ${dt}`;
  setTimeout(()=>openLogin(),600);
}

// populate grade select
(function init(){
  const sel= document.getElementById('stuGrade');
  for(let g=1; g<=10; g++){ sel.innerHTML += `<option value="${g}">Grade ${g}</option>`; }
})();

function studentLogin(){
  const name = document.getElementById('stuName').value.trim();
  const grade = document.getElementById('stuGrade').value;
  if(!name || !grade){ alert('Enter name and grade'); return; }
  // store student
  const students = LS.students();
  // ensure unique name
  let uname = name; let idx=1;
  while(students.find(s=>s.name===uname)) { idx++; uname = name + ' ' + idx; }
  const code = generateStudentCode();
  const date = localStorage.getItem('sai_demo_date') || null;
  const st = {name:uname, code, grade, date, status:'Not Paid', isSAIStudent:false, joinedAt:null};
  students.push(st); LS.setStudents(students);
  // chats placeholder
  const chats = LS.chats(); chats[code] = chats[code] || []; LS.setChats(chats);
  // set current
  currentStudent = st;
  showMainUI();
  localStorage.setItem('sai_event','student_created_'+Date.now());
}
function generateStudentCode(){ return 'DA'+Math.floor(Math.random()*90000+10000); }

function showMainUI(){
  document.getElementById('demoCard').classList.add('hidden');
  document.getElementById('loginCard').classList.add('hidden');
  document.getElementById('mainCard').classList.remove('hidden');
  document.getElementById('welcome').innerText = `Welcome, ${currentStudent.name}`;
  document.getElementById('statusBadge').innerText = currentStudent.status;
  document.getElementById('studentCode').innerText = currentStudent.code;
  document.getElementById('demoShow').innerText = currentStudent.date || '—';
  document.getElementById('infoName').innerText = currentStudent.name;
  document.getElementById('infoGrade').innerText = 'Grade '+currentStudent.grade;
  // populate grade & subject selects
  const gsel = document.getElementById('gradeSelect'); gsel.innerHTML='';
  for(let g=1; g<=10; g++){ gsel.innerHTML += `<option value="${g}">Grade ${g}</option>`; }
  gsel.value = currentStudent.grade;
  renderSubjects();
  window.addEventListener('storage', onStorage);
  renderChatWindow();
}

function onGradeChange(){ renderSubjects(); renderWorksheet(); }
function renderSubjects(){
  const grade = document.getElementById('gradeSelect').value || currentStudent.grade;
  const subs = SUBJECTS[grade];
  const ssel = document.getElementById('subjectSelect'); ssel.innerHTML='';
  subs.forEach(s=> ssel.innerHTML += `<option value="${s}">${s}</option>`);
  renderWorksheet();
}

function renderWorksheet(){
  const grade = document.getElementById('gradeSelect').value || currentStudent.grade;
  const subject = document.getElementById('subjectSelect').value;
  const area = document.getElementById('worksheetArea');
  area.innerHTML = `<h4>Grade ${grade} — ${subject}</h4><div class="small">20 questions. Use SAI for hints (press SAI circle). For full step solutions ask Admin.</div>`;
  const ws = generateWorksheet(grade,subject);
  ws.forEach((q, idx)=>{
    area.innerHTML += `
      <div class="question" id="q_${q.id}">
        <div><b>Q${idx+1}.</b> ${q.text}</div>
        <div class="q-actions">
          <button class="btn-small" onclick="requestHelp('${q.id}','${grade}','${subject}',${idx})">Ask SAI (hint)</button>
          <button class="btn-small ghost" onclick="showLocalAnswer('${grade}','${subject}',${idx})">Show Answer (self-check)</button>
        </div>
        <div id="ans_${q.id}" class="small hidden"></div>
      </div>`;
  });
}

// show student self-check answer (brief)
function showLocalAnswer(grade,subject,idx){
  const ws = generateWorksheet(grade,subject);
  const q = ws[idx];
  const elAns = document.getElementById('ans_'+q.id);
  elAns.innerHTML = `<b>Answer:</b> ${q.answer}`;
  elAns.classList.remove('hidden');
}

// --- chat/help modal
function openChatModal(){ document.getElementById('chatModal').classList.remove('hidden'); renderChatWindow(); }
function closeChatModal(){ document.getElementById('chatModal').classList.add('hidden'); }
function sendHelp(){
  const text = document.getElementById('helpInput').value.trim();
  if(!text) return alert('Type a question');
  const code = currentStudent.code;
  const chats = LS.chats(); chats[code]=chats[code]||[]; chats[code].push({from:'user', text, ts:Date.now()}); LS.setChats(chats);
  const help = LS.help(); help.push({id:'HR'+Date.now(), code, name:currentStudent.name, grade:currentStudent.grade, subject:'General', qtext:text, status:'open', adminReply:null, ts:Date.now()}); LS.setHelp(help);
  localStorage.setItem('sai_event','help_sent_'+Date.now());
  document.getElementById('helpInput').value=''; renderChatWindow(); alert('Help request sent to Admin.');
}

// request help for a specific question (hint mode)
function requestHelp(qid, grade, subject, qidx){
  if(!currentStudent) return alert('Please login');
  // create help request with context
  const ws = generateWorksheet(grade,subject);
  const q = ws[qidx];
  const helpReq = LS.help(); helpReq.push({id:'HR'+Date.now(), code:currentStudent.code, name:currentStudent.name, grade, subject, qid, qtext:q.text, status:'open', adminReply:null, ts:Date.now()}); LS.setHelp(helpReq);
  // append chat user message
  const chats = LS.chats(); chats[currentStudent.code]=chats[currentStudent.code]||[]; chats[currentStudent.code].push({from:'user', text:`Help request on ${subject}: ${q.text}`, ts:Date.now()}); LS.setChats(chats);
  localStorage.setItem('sai_event','help_req_'+Date.now());
  // immediate hint from local SAI: hints only (if Paid -> richer hints)
  let hint = 'Try breaking the problem into smaller steps.';
  const students = LS.students(); const s = students.find(x=>x.code===currentStudent.code);
  if(s && s.status==='Paid'){ hint = 'Start by writing known values. Then apply formulas step by step.'; }
  const chats2 = LS.chats(); chats2[currentStudent.code].push({from:'ai', text: 'SAI hint: '+hint, ts:Date.now()}); LS.setChats(chats2);
  renderChatWindow();
  alert('A hint has been added. Admin will send full steps.');
}

// render student's chat window
function renderChatWindow(){
  const win = document.getElementById('chatWindow'); win.innerHTML='';
  if(!currentStudent) return;
  const chats = LS.chats()[currentStudent.code] || [];
  chats.forEach(m=>{ const d=document.createElement('div'); d.className='msg '+(m.from==='user'?'user':'ai'); d.innerText=m.text; win.appendChild(d); });
  // also show admin answered help requests
  const helpAnswered = LS.help().filter(h=>h.code===currentStudent.code && h.status==='answered');
  helpAnswered.forEach(h=>{ const d=document.createElement('div'); d.className='msg ai'; d.innerText = 'Admin: ' + h.adminReply; win.appendChild(d); });
  win.scrollTop = win.scrollHeight;
}

// Meet
function joinMeet(){
  if(!currentStudent) return alert('Login first');
  // record join
  const joins = LS.meets(); joins.push({code:currentStudent.code, name:currentStudent.name, ts:Date.now()}); LS.setMeets(joins);
  localStorage.setItem('sai_event','meet_join_'+Date.now());
  document.getElementById('meetContainer').innerHTML = `<iframe class="meet-frame" src="https://meet.jit.si/SAI2025MEET" allow="camera; microphone; fullscreen"></iframe>`;
  // update student joinedAt
  const students = LS.students(); const s = students.find(x=>x.code===currentStudent.code); if(s){ s.joinedAt = Date.now(); LS.setStudents(students); localStorage.setItem('sai_event','student_joined_'+Date.now()); }
}
function leaveMeet(){ document.getElementById('meetContainer').innerHTML=''; }

// storage listener to update status/chat if admin changes things
window.addEventListener('storage', function(){ if(!currentStudent) return; const students = LS.students(); const s = students.find(x=>x.code===currentStudent.code); if(s){ currentStudent=s; document.getElementById('statusBadge').innerText=s.status; renderChatWindow(); } });

// restore previous student if exists locally (convenience)
(function restore(){
  const st = LS.students(); if(st.length>0){ const last = st[st.length-1]; currentStudent=last; showMainUI(); }
})();
</script>
</body>
</html>
