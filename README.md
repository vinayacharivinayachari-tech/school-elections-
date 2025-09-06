# school-elections-
# Creating the election site files and packaging into a ZIP for download.
# This Python code will create the site files under /mnt/data/school_election_final/ and zip them.
# After running, it will list the generated zip path for the user to download.
import os, json, zipfile, textwrap

base_dir = '/mnt/data/school_election_final'
os.makedirs(base_dir, exist_ok=True)
os.makedirs(os.path.join(base_dir, 'js'), exist_ok=True)
os.makedirs(os.path.join(base_dir, 'css'), exist_ok=True)
os.makedirs(os.path.join(base_dir, 'assets'), exist_ok=True)

# File contents
index_html = r"""<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>School Election - Home</title>
<link rel="stylesheet" href="css/style.css">
</head>
<body>
<header class="site-header">
  <div class="header-left">
    <img id="schoolLogo" src="" alt="School Logo" style="height:64px;display:none;border-radius:8px;margin-right:12px;">
    <div id="schoolName">G M H P S GILLESUGUR, Tq/Dist - RAICHUR</div>
  </div>
  <div class="header-right">
    <div class="login-box">
      <h3>Admin / Teacher Login</h3>
      <input id="loginUser" placeholder="Username" />
      <input id="loginPass" placeholder="Password" type="password" />
      <button id="btnLogin">Login</button>
    </div>
  </div>
</header>

<main class="container">
  <section class="board">
    <h2>📊 Live Candidate Vote Board</h2>
    <div id="totalVotes">Total Votes: 0</div>
    <div id="candidatesGrid" class="grid"></div>
  </section>

  <section class="actions">
    <a class="btn" href="voter.html">Voter Login (Roll No)</a>
    <a class="btn" href="admin.html">Admin Dashboard</a>
  </section>
</main>

<script src="js/app.js"></script>
<script>
// login handling on index
document.getElementById('btnLogin').addEventListener('click', function(){
  const u = document.getElementById('loginUser').value.trim();
  const p = document.getElementById('loginPass').value.trim();
  if(!u || !p){ alert('Enter credentials'); return; }
  const teachers = app.getTeachers();
  const admin = app.getAdminCred();
  if(u === admin.user && p === admin.pass){
    sessionStorage.setItem('role','admin');
    location.href = 'admin.html';
    return;
  }
  const t = teachers.find(x=>x.username===u && x.password===p);
  if(t){ sessionStorage.setItem('role','teacher'); sessionStorage.setItem('teacher', JSON.stringify(t)); location.href='teacher.html'; return; }
  alert('Invalid credentials');
});

// show live candidate board
function renderIndexBoard(){ app.renderCandidatesGrid(document.getElementById('candidatesGrid')); document.getElementById('totalVotes').textContent = 'Total Votes: '+app.getTotalVotes(); const sc = app.getSchool(); if(sc.logo){ const img=document.getElementById('schoolLogo'); img.src=sc.logo; img.style.display='inline-block'; } }
renderIndexBoard();
window.addEventListener('storage', renderIndexBoard);
</script>
</body>
</html>"""

admin_html = r"""<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1">
<title>Admin - School Election</title>
<link rel="stylesheet" href="css/style.css">
</head>
<body>
<header class="site-header">
  <div class="header-left">
    <img id="schoolLogo" src="" alt="School Logo" style="height:64px;display:none;border-radius:8px;margin-right:12px;">
    <div id="schoolName">G M H P S GILLESUGUR, Tq/Dist - RAICHUR</div>
  </div>
  <div class="header-right">
    <button id="btnLogout" class="btn">Logout</button>
  </div>
</header>
<main class="container">
  <h1>Admin Dashboard</h1>
  <section class="admin-grid">
    <div class="card">
      <h3>School Logo / Header</h3>
      <input type="file" id="schoolLogoInput" accept="image/*"><br><br>
      <button id="saveSchool">Save Header</button>
    </div>
    <div class="card">
      <h3>Manage Candidates</h3>
      <input id="candName" placeholder="Candidate Name">
      <input id="candClass" placeholder="Class (optional)">
      <input type="file" id="candPhoto" accept="image/*"><br>
      <button id="addCandidate">Add Candidate</button>
      <div id="cList"></div>
    </div>
    <div class="card">
      <h3>Manage Voters</h3>
      <input id="vName" placeholder="Student Name">
      <input id="vRoll" placeholder="Roll Number">
      <input id="vClass" placeholder="Class (1-7)">
      <input id="vSection" placeholder="Section (A/B)">
      <button id="addVoter">Add Voter</button>
      <div id="vList"></div>
    </div>
    <div class="card">
      <h3>Manage Teachers</h3>
      <input id="tUser" placeholder="Username">
      <input id="tPass" placeholder="Password">
      <input id="tClass" placeholder="Class (1-7)">
      <input id="tSection" placeholder="Section (A/B)">
      <button id="addTeacher">Add Teacher</button>
      <div id="tList"></div>
    </div>

    <div class="card full">
      <h3>Live Results</h3>
      <div id="resultsArea"></div>
    </div>

    <div class="card">
      <h3>Backup / Reset</h3>
      <button id="exportBtn">Export Backup</button>
      <input type="file" id="importFile" accept=".json">
      <button id="importBtn">Import Backup</button>
      <hr>
      <button id="resetBtn" style="background:#c33;color:#fff">Reset All Data</button>
    </div>
  </section>
</main>
<script src="js/app.js"></script>
<script>
// admin auth check
(function(){
  const role = sessionStorage.getItem('role');
  if(role!=='admin'){ alert('Please login as Admin'); location.href='index.html'; return; }
  // render school header
  const sc = app.getSchool(); if(sc.logo){ document.getElementById('schoolLogo').src = sc.logo; document.getElementById('schoolLogo').style.display='inline-block'; }
  document.getElementById('saveSchool').addEventListener('click', function(){
    const f = document.getElementById('schoolLogoInput').files[0];
    if(!f){ alert('Choose image'); return; }
    app.readFileAsDataURL(f, function(data){
      app.saveSchool({name:'G M H P S GILLESUGUR, Tq/Dist - RAICHUR', logo:data}); location.reload();
    });
  });

  // candidates
  function refreshCandidates(){ app.renderCandidateList(document.getElementById('cList')); app.renderResults(document.getElementById('resultsArea')); }
  document.getElementById('addCandidate').addEventListener('click', function(){
    const name=document.getElementById('candName').value.trim(); const cls=document.getElementById('candClass').value.trim();
    const f = document.getElementById('candPhoto').files[0];
    if(!name || !f){ alert('Name and photo required'); return; }
    app.readFileAsDataURL(f, function(data){ app.addCandidate({name, class:cls, photo:data}); refreshCandidates(); });
  });

  // voters
  function refreshVoters(){ app.renderVoterList(document.getElementById('vList')); }
  document.getElementById('addVoter').addEventListener('click', function(){
    const name=document.getElementById('vName').value.trim(); const roll=document.getElementById('vRoll').value.trim(); const cls=document.getElementById('vClass').value.trim(); const sec=document.getElementById('vSection').value.trim()||'A';
    if(!name||!roll||!cls){ alert('Name/Roll/Class required'); return; }
    app.addVoter({name, roll, class:cls, section:sec}); refreshVoters();
  });

  // teachers
  function refreshTeachers(){ app.renderTeacherList(document.getElementById('tList')); }
  document.getElementById('addTeacher').addEventListener('click', function(){
    const u=document.getElementById('tUser').value.trim(); const p=document.getElementById('tPass').value.trim(); const cls=document.getElementById('tClass').value.trim(); const sec=document.getElementById('tSection').value.trim()||'A';
    if(!u||!p||!cls){ alert('Username/Password/Class required'); return; }
    app.addTeacher({username:u,password:p,class:cls,section:sec}); refreshTeachers();
  });

  // export/import/reset
  document.getElementById('exportBtn').addEventListener('click', function(){ app.exportBackup(); });
  document.getElementById('importBtn').addEventListener('click', function(){
    const f=document.getElementById('importFile').files[0]; if(!f){ alert('Choose backup file'); return; }
    const r=new FileReader(); r.onload=function(e){ try{ app.importBackup(JSON.parse(e.target.result)); alert('Imported'); location.reload(); }catch(err){ alert('Invalid file'); } }; r.readAsText(f);
  });
  document.getElementById('resetBtn').addEventListener('click', function(){ if(confirm('Reset all data?')){ app.resetAll(); alert('Reset done'); location.reload(); } });

  // logout
  document.getElementById('btnLogout').addEventListener('click', function(){ sessionStorage.clear(); location.href='index.html'; });

  // initial render
  refreshCandidates(); refreshVoters(); refreshTeachers();
})();
</script>
</body>
</html>"""

teacher_html = r"""<!doctype html>
<html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1"><title>Teacher Dashboard</title><link rel="stylesheet" href="css/style.css"></head><body>
<header class="site-header">
  <div class="header-left">
    <img id="schoolLogo" src="" alt="School Logo" style="height:64px;display:none;border-radius:8px;margin-right:12px;">
    <div id="schoolName">G M H P S GILLESUGUR, Tq/Dist - RAICHUR</div>
  </div>
  <div class="header-right">
    <button id="btnLogout" class="btn">Logout</button>
  </div>
</header>
<main class="container">
  <h1>Teacher Dashboard</h1>
  <div id="teacherInfo"></div>
  <section class="card">
    <h3>Voters (Your Class)</h3>
    <div id="teacherVoters"></div>
  </section>
  <section class="card">
    <h3>Live Results (Your Class)</h3>
    <div id="teacherResults"></div>
  </section>
</main>
<script src="js/app.js"></script>
<script>
(function(){
  const role=sessionStorage.getItem('role'); if(role!=='teacher'){ alert('Teacher login required'); location.href='index.html'; return; }
  const t = JSON.parse(sessionStorage.getItem('teacher')||'null'); if(!t){ alert('Invalid teacher session'); location.href='index.html'; return; }
  document.getElementById('teacherInfo').textContent = 'Logged in: '+t.username+' | Class: '+t.class+' Section: '+t.section;
  document.getElementById('btnLogout').addEventListener('click', ()=>{ sessionStorage.clear(); location.href='index.html'; });
  const sc = app.getSchool(); if(sc.logo){ document.getElementById('schoolLogo').src=sc.logo; document.getElementById('schoolLogo').style.display='inline-block'; }
  function refresh(){ app.renderVotersForTeacher(document.getElementById('teacherVoters'), t); app.renderResults(document.getElementById('teacherResults'), t); }
  refresh(); window.addEventListener('storage', refresh);
})();
</script>
</body></html>"""

voter_html = r"""<!doctype html><html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1"><title>Voter Login</title><link rel="stylesheet" href="css/style.css"></head><body>
<header class="site-header"><div class="header-left"><div id="schoolName">G M H P S GILLESUGUR, Tq/Dist - RAICHUR</div></div></header>
<main class="container"><h1>Voter Login</h1>
<div class="card">
  <input id="rollInput" placeholder="Enter Roll Number">
  <button id="btnCheck">Proceed to Vote</button>
  <div id="msg"></div>
</div>
</main>
<script src="js/app.js"></script>
<script>
document.getElementById('btnCheck').addEventListener('click', function(){
  const roll=document.getElementById('rollInput').value.trim(); if(!roll){ alert('Enter roll'); return; }
  const voter = app.findVoterByRoll(roll);
  if(!voter){ document.getElementById('msg').textContent='Not a registered voter.'; return; }
  if(voter.voted){ document.getElementById('msg').textContent='You have already voted.'; return; }
  sessionStorage.setItem('currentVoter', JSON.stringify(voter)); location.href='vote.html';
});
</script>
</body></html>"""

vote_html = r"""<!doctype html><html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1"><title>Vote</title><link rel="stylesheet" href="css/style.css"></head><body>
<header class="site-header"><div class="header-left"><div id="schoolName">G M H P S GILLESUGUR, Tq/Dist - RAICHUR</div></div></header>
<main class="container">
  <h1>Cast Your Vote</h1>
  <div id="voterInfo"></div>
  <div id="candidateArea" class="grid"></div>
  <div id="thanks" style="display:none" class="card">Thank you for voting!</div>
</main>
<script src="js/app.js"></script>
<script>
(function(){
  const cv = JSON.parse(sessionStorage.getItem('currentVoter')||'null'); if(!cv){ alert('Please login as voter first'); location.href='voter.html'; return; }
  document.getElementById('voterInfo').textContent = 'Voter: '+cv.name+' | Roll: '+cv.roll+' | Class: '+cv.class+' Section: '+cv.section;
  function render(){ app.renderCandidatesForVoting(document.getElementById('candidateArea'), cv); }
  render();
  window.addEventListener('storage', render);
})();
</script>
</body></html>"""

results_html = r"""<!doctype html><html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1"><title>Results</title><link rel="stylesheet" href="css/style.css"></head><body>
<header class="site-header"><div class="header-left"><div id="schoolName">G M H P S GILLESUGUR, Tq/Dist - RAICHUR</div></div></header>
<main class="container">
  <h1>Live Results</h1>
  <div id="resultsArea"></div>
</main>
<script src="js/app.js"></script>
<script>
(function(){
  const role = sessionStorage.getItem('role');
  const t = JSON.parse(sessionStorage.getItem('teacher')||'null');
  app.renderResults(document.getElementById('resultsArea'), t);
  window.addEventListener('storage', ()=> app.renderResults(document.getElementById('resultsArea'), t));
})();
</script>
</body></html>"""

css_style = r"""*{box-sizing:border-box;font-family:Arial,Helvetica,sans-serif}body{margin:0;background:#f7fafc;color:#111}a{color:inherit;text-decoration:none}
.site-header{display:flex;justify-content:space-between;align-items:center;padding:12px 20px;background:#fff;border-bottom:1px solid #e2e8f0;box-shadow:0 1px 3px rgba(0,0,0,0.03)}
.header-left{display:flex;align-items:center}
.header-right{display:flex;align-items:center}
.container{max-width:1000px;margin:20px auto;padding:10px}
.login-box input{display:block;margin:6px 0;padding:8px;width:200px}
.btn{display:inline-block;padding:8px 12px;background:#2b6cb0;color:#fff;border-radius:6px;border:none;cursor:pointer;margin:6px 4px}
.btn:hover{opacity:0.9}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(160px,1fr));gap:12px;margin-top:12px}
.card{background:#fff;padding:12px;border-radius:8px;box-shadow:0 2px 6px rgba(2,6,23,0.05);margin-bottom:12px}
.admin-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(280px,1fr));gap:12px}
.candidate-card{text-align:center;padding:10px}
.candidate-card img{width:100%;height:120px;object-fit:cover;border-radius:8px}
.small{font-size:0.9rem;color:#555}
.full{grid-column:1/-1}
input,select{padding:8px;margin:6px 0;width:100%}
#cList img{height:40px;border-radius:6px;margin-right:8px}
.results-row{display:flex;align-items:center;gap:12px;padding:8px;border-bottom:1px solid #eee}
.results-row img{width:64px;height:64px;object-fit:cover;border-radius:8px}"""

js_app = r"""// Core application logic for election system using localStorage
const app = (function(){
  const ADMIN_KEY = 'election_admin';
  const CAND_KEY = 'election_candidates';
  const VOTER_KEY = 'election_voters';
  const TEACH_KEY = 'election_teachers';
  const SCHOOL_KEY = 'election_school';

  function ensureAdmin(){
    if(!localStorage.getItem(ADMIN_KEY)){
      localStorage.setItem(ADMIN_KEY, JSON.stringify({user:'GMHPS01', pass:'KE@1102910'}));
    }
  }

  function getAdminCred(){ ensureAdmin(); return JSON.parse(localStorage.getItem(ADMIN_KEY)); }

  function getCandidates(){ return JSON.parse(localStorage.getItem(CAND_KEY) || '[]'); }
  function saveCandidates(arr){ localStorage.setItem(CAND_KEY, JSON.stringify(arr)); window.dispatchEvent(new Event('storage')); }

  function getVoters(){ return JSON.parse(localStorage.getItem(VOTER_KEY) || '[]'); }
  function saveVoters(arr){ localStorage.setItem(VOTER_KEY, JSON.stringify(arr)); window.dispatchEvent(new Event('storage')); }

  function getTeachers(){ return JSON.parse(localStorage.getItem(TEACH_KEY) || '[]'); }
  function saveTeachers(arr){ localStorage.setItem(TEACH_KEY, JSON.stringify(arr)); window.dispatchEvent(new Event('storage')); }

  function getSchool(){ return JSON.parse(localStorage.getItem(SCHOOL_KEY) || '{}'); }
  function saveSchool(obj){ localStorage.setItem(SCHOOL_KEY, JSON.stringify(obj)); window.dispatchEvent(new Event('storage')); }

  function addCandidate(c){
    const arr = getCandidates();
    const id = Date.now() + Math.floor(Math.random()*999);
    arr.push({id,name:c.name,class:c.class||'',photo:c.photo,votes:0});
    saveCandidates(arr);
  }

  function addVoter(v){ const arr=getVoters(); if(arr.find(x=>x.roll===v.roll)){ alert('Roll exists'); return; } arr.push({name:v.name,roll:v.roll,class:v.class,section:v.section||'A',voted:false}); saveVoters(arr); }
  function addTeacher(t){ const arr=getTeachers(); if(arr.find(x=>x.username===t.username)){ alert('username exists'); return; } arr.push(t); saveTeachers(arr); }

  function findVoterByRoll(roll){ return getVoters().find(x=>String(x.roll)===String(roll)); }

  function voteForCandidate(candidateId, roll){
    const cands = getCandidates(); const cand = cands.find(x=>x.id==candidateId); if(!cand) return false;
    const voters = getVoters(); const v = voters.find(x=>String(x.roll)===String(roll)); if(!v || v.voted) return false;
    cand.votes = (cand.votes||0)+1; v.voted = true;
    saveCandidates(cands); saveVoters(voters);
    return true;
  }

  function exportBackup(){
    const data = {candidates:getCandidates(), voters:getVoters(), teachers:getTeachers(), school:getSchool(), admin:getAdminCred()};
    const blob = new Blob([JSON.stringify(data, null, 2)], {type:'application/json'});
    const url = URL.createObjectURL(blob); const a=document.createElement('a'); a.href=url; a.download='backup.json'; document.body.appendChild(a); a.click(); a.remove();
  }

  function importBackup(obj){
    if(obj.candidates) localStorage.setItem(CAND_KEY, JSON.stringify(obj.candidates));
    if(obj.voters) localStorage.setItem(VOTER_KEY, JSON.stringify(obj.voters));
    if(obj.teachers) localStorage.setItem(TEACH_KEY, JSON.stringify(obj.teachers));
    if(obj.school) localStorage.setItem(SCHOOL_KEY, JSON.stringify(obj.school));
    if(obj.admin) localStorage.setItem(ADMIN_KEY, JSON.stringify(obj.admin));
    window.dispatchEvent(new Event('storage'));
  }

  function resetAll(){ localStorage.removeItem(CAND_KEY); localStorage.removeItem(VOTER_KEY); localStorage.removeItem(TEACH_KEY); localStorage.removeItem(SCHOOL_KEY); ensureAdmin(); window.dispatchEvent(new Event('storage')); }

  function readFileAsDataURL(file, cb){ const r=new FileReader(); r.onload=function(e){ cb(e.target.result); }; r.readAsDataURL(file); }

  // render helpers
  function renderCandidatesGrid(container){
    container.innerHTML='';
    const arr = getCandidates();
    arr.forEach(c=>{
      const card=document.createElement('div'); card.className='candidate-card card';
      card.innerHTML = `<img src="${c.photo}" alt="${c.name}"><h4>${c.name}</h4><div class="small">${c.class?('Class '+c.class):''}</div><div><strong>${c.votes||0} Votes</strong></div>`;
      container.appendChild(card);
    });
  }

  function renderCandidateList(container){
    container.innerHTML=''; const arr=getCandidates();
    arr.forEach(c=>{ const d=document.createElement('div'); d.className='results-row'; d.innerHTML=`<img src="${c.photo}"><div><strong>${c.name}</strong><div class="small">Class: ${c.class||'-'}</div></div><div style="margin-left:auto">${c.votes||0} votes</div>`;
    // delete button
    const del=document.createElement('button'); del.className='btn'; del.textContent='Delete'; del.style.marginLeft='8px'; del.onclick=function(){ if(confirm('Delete candidate?')){ saveCandidates(arr.filter(x=>x.id!==c.id)); window.location.reload(); } };
    d.appendChild(del); container.appendChild(d);
    });
  }

  function renderVoterList(container){
    container.innerHTML=''; const arr=getVoters();
    arr.forEach(v=>{ const d=document.createElement('div'); d.className='results-row'; d.innerHTML=`<div><strong>${v.name}</strong><div class="small">Roll: ${v.roll} | Class: ${v.class} | Section: ${v.section}</div></div><div style="margin-left:auto">${v.voted?'<em>Voted</em>':'<em>Not Voted</em>'}</div>`;
    container.appendChild(d);
    });
  }

  function renderTeacherList(container){
    container.innerHTML=''; const arr=getTeachers();
    arr.forEach(t=>{ const d=document.createElement('div'); d.className='results-row'; d.innerHTML=`<div><strong>${t.username}</strong><div class="small">Class: ${t.class} | Section: ${t.section}</div></div>`;
    container.appendChild(d);
    });
  }

  function renderResults(container, teacher=null){
    container.innerHTML=''; const cands=getCandidates();
    let rows=cands;
    if(teacher){ // filter by teacher class/section if teacher.class matches candidate.class
      rows = cands.filter(x=>String(x.class)===String(teacher.class));
    }
    rows.forEach(c=>{ const row=document.createElement('div'); row.className='results-row'; row.innerHTML=`<img src="${c.photo}"><div><strong>${c.name}</strong><div class="small">Class: ${c.class||'-'}</div></div><div style="margin-left:auto"><strong>${c.votes||0}</strong> votes</div>`; container.appendChild(row); });
  }

  function renderVotersForTeacher(container, teacher){
    container.innerHTML=''; const voters = getVoters().filter(x=>String(x.class)===String(teacher.class) && (teacher.section?String(x.section)===String(teacher.section):true));
    voters.forEach(v=>{ const d=document.createElement('div'); d.className='results-row'; d.innerHTML=`<div><strong>${v.name}</strong><div class="small">Roll: ${v.roll} | Section: ${v.section}</div></div><div style="margin-left:auto">${v.voted?'<em>Voted</em>':'<em>Not Voted</em>'}</div>`; container.appendChild(d); });
  }

  function renderCandidatesForVoting(container, voter){
    container.innerHTML=''; const cands = getCandidates();
    cands.forEach(c=>{
      const card = document.createElement('div'); card.className='candidate-card card';
      const img = document.createElement('img'); img.src=c.photo;
      const h = document.createElement('h4'); h.textContent=c.name;
      const btn = document.createElement('button'); btn.className='btn'; btn.textContent='Vote for '+c.name;
      btn.onclick = function(){ if(confirm('Confirm vote for '+c.name+'?')){ const ok = voteForCandidate(c.id, voter.roll); if(ok){ alert('Vote recorded. Thank you!'); sessionStorage.removeItem('currentVoter'); document.getElementById('candidateArea').style.display='none'; document.getElementById('thanks').style.display='block'; }else{ alert('Vote failed (maybe you already voted)'); } } };
      card.appendChild(img); card.appendChild(h); card.appendChild(document.createElement('div')).className='small'; card.appendChild(btn);
      container.appendChild(card);
    });
  }

  function getTotalVotes(){ return getCandidates().reduce((s,c)=>s+(c.votes||0),0); }

  // expose API
  ensureAdmin();
  return {
    getAdminCred, getCandidates, addCandidate, getVoters, addVoter, getTeachers, addTeacher, findVoterByRoll, voteForCandidate,
    exportBackup, importBackup, resetAll, readFileAsDataURL, renderCandidatesGrid, renderCandidateList, renderVoterList,
    renderTeacherList, renderResults, renderVotersForTeacher, renderCandidatesForVoting, getTotalVotes, getSchool, saveSchool,
    getTeachers: getTeachers, getAdminCred: getAdminCred, getSchool: getSchool, saveSchool: saveSchool, exportBackup: exportBackup,
    importBackup: importBackup, resetAll: resetAll, readFileAsDataURL: readFileAsDataURL, addCandidate: addCandidate, addVoter: addVoter,
    addTeacher: addTeacher, findVoterByRoll: findVoterByRoll
  };
})();"""

# write files
files = {
    'index.html': index_html,
    'admin.html': admin_html,
    'teacher.html': teacher_html,
    'voter.html': voter_html,
    'vote.html': vote_html,
    'results.html': results_html,
    'css/style.css': css_style,
    'js/app.js': js_app
}

for path, content in files.items():
    full = os.path.join(base_dir, path)
    d = os.path.dirname(full)
    os.makedirs(d, exist_ok=True)
    with open(full, 'w', encoding='utf-8') as f:
        f.write(content)

# create zip
zip_path = '/mnt/data/school_election_final.zip'
with zipfile.ZipFile(zip_path, 'w', zipfile.ZIP_DEFLATED) as z:
    for root, dirs, filenames in os.walk(base_dir):
        for fn in filenames:
            file_path = os.path.join(root, fn)
            arcname = os.path.relpath(file_path, base_dir)
            z.write(file_path, arcname)

print("Created:", zip_path)
