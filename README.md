# SERVICE-EDUCATION
Logiciel de gestion de la vie scolaire 

<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Inspecteur d'Education CI - Lycée</title>

<script src="https://cdn.tailwindcss.com"></script>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css">

<style>
body { font-family: Arial; }
.sidebar { background: linear-gradient(180deg,#1e3a8a,#3b82f6); }
.nav-active { background: rgba(255,255,255,0.2); }
.card-hover:hover { transform: translateY(-4px); }
</style>
</head>

<body class="bg-gray-50">

<div class="flex h-screen">

<!-- SIDEBAR -->
<div class="w-72 sidebar text-white flex flex-col">

<div class="p-6 border-b border-white/20">
<h1 class="text-2xl font-bold">Inspecteur d'Education CI</h1>
<p class="text-sm opacity-70">Vie Scolaire</p>
</div>

<nav class="flex-1 p-3 space-y-2">

<button onclick="showPage('dashboard')" class="nav-btn w-full p-3 text-left nav-active">Dashboard</button>
<button onclick="showPage('eleves')" class="nav-btn w-full p-3 text-left">Élèves</button>
<button onclick="showPage('educateurs')" class="nav-btn w-full p-3 text-left">Éducateurs</button>
<button onclick="showPage('incidents')" class="nav-btn w-full p-3 text-left">Incidents</button>
<button onclick="showPage('notifications')" class="nav-btn w-full p-3 text-left">Notifications</button>
<button onclick="showPage('rapports')" class="nav-btn w-full p-3 text-left">Rapports</button>
<button onclick="showPage('stats')" class="nav-btn w-full p-3 text-left">Stats</button>

</nav>

</div>

<!-- CONTENT -->
<div class="flex-1 p-6 overflow-auto">
<h2 id="title" class="text-2xl font-bold mb-6"></h2>
<div id="content"></div>
</div>

</div>

<div id="toast" class="fixed bottom-5 right-5 bg-green-600 text-white px-6 py-3 rounded hidden"></div>

<script>

// ================= DATA =================
let eleves = JSON.parse(localStorage.getItem('eleves')) || [];
let educateurs = JSON.parse(localStorage.getItem('educateurs')) || [];
let incidents = JSON.parse(localStorage.getItem('incidents')) || [];
let notifications = JSON.parse(localStorage.getItem('notifications')) || [];

// ================= SAVE =================
function save(){
localStorage.setItem('eleves',JSON.stringify(eleves));
localStorage.setItem('educateurs',JSON.stringify(educateurs));
localStorage.setItem('incidents',JSON.stringify(incidents));
localStorage.setItem('notifications',JSON.stringify(notifications));
}

// ================= TOAST =================
function toast(msg){
let t=document.getElementById("toast");
t.textContent=msg;
t.classList.remove("hidden");
setTimeout(()=>t.classList.add("hidden"),2000);
}

// ================= NAV =================
function showPage(page){

document.querySelectorAll(".nav-btn").forEach(b=>b.classList.remove("nav-active"));
event.target.classList.add("nav-active");

document.getElementById("title").innerText=page.toUpperCase();

if(page==="dashboard") dashboard();
if(page==="eleves") pageEleves();
if(page==="educateurs") pageEduc();
if(page==="incidents") pageIncidents();
if(page==="notifications") pageNotif();
if(page==="rapports") pageRapport();
if(page==="stats") pageStats();
}

// ================= DASHBOARD =================
function dashboard(){
document.getElementById("content").innerHTML=`
<div class="grid grid-cols-4 gap-4">
<div class="bg-white p-5 rounded">Élèves ${eleves.length}</div>
<div class="bg-white p-5 rounded">Incidents ${incidents.length}</div>
<div class="bg-white p-5 rounded">Éducateurs ${educateurs.length}</div>
<div class="bg-white p-5 rounded">Notifications ${notifications.length}</div>
</div>
`;
}

// ================= ELEVE =================
function pageEleves(){
document.getElementById("content").innerHTML=`

<input id="search" placeholder="Recherche..." class="mb-4 p-2 border w-full">

<div class="grid grid-cols-4 gap-2 mb-4">
<input id="nom" placeholder="Nom" class="border p-2">
<input id="prenom" placeholder="Prénom" class="border p-2">
<input id="classe" placeholder="Classe" class="border p-2">
<button onclick="addEleve()" class="bg-blue-600 text-white">Ajouter</button>
</div>

<table class="w-full bg-white">
<thead><tr><th>Nom</th><th>Classe</th><th></th></tr></thead>
<tbody id="list"></tbody>
</table>
`;

renderEleves();
document.getElementById("search").oninput=searchEleve;
}

function renderEleves(list=eleves){
document.getElementById("list").innerHTML=list.map(e=>`
<tr>
<td>${e.nom} ${e.prenom}</td>
<td>${e.classe}</td>
<td><button onclick="delEleve(${e.id})">❌</button></td>
</tr>
`).join("");
}

function addEleve(){
let nom=document.getElementById("nom").value;
let prenom=document.getElementById("prenom").value;
let classe=document.getElementById("classe").value;

if(!nom||!prenom||!classe){toast("Remplis tout");return;}

eleves.push({id:Date.now(),nom,prenom,classe});
save();
pageEleves();
toast("Ajouté");
}

function delEleve(id){
eleves=eleves.filter(e=>e.id!==id);
save();
pageEleves();
}

function searchEleve(){
let v=document.getElementById("search").value.toLowerCase();
renderEleves(eleves.filter(e=>e.nom.toLowerCase().includes(v)||e.prenom.toLowerCase().includes(v)));
}

// ================= EDUC =================
function pageEduc(){
document.getElementById("content").innerHTML=`
<button onclick="addEduc()" class="mb-4 bg-green-600 text-white p-2">Ajouter</button>
<div id="educList"></div>
`;

document.getElementById("educList").innerHTML=educateurs.map(e=>`
<div class="bg-white p-4 mb-2">${e.nom} - ${e.contact}</div>
`).join("");
}

function addEduc(){
let nom=prompt("Nom");
let contact=prompt("Contact");
if(nom&&contact){
educateurs.push({id:Date.now(),nom,contact});
save();
pageEduc();
}
}

// ================= INCIDENT =================
function pageIncidents(){
document.getElementById("content").innerHTML=`

<select id="elev">
${eleves.map(e=>`<option value="${e.id}">${e.nom}</option>`).join("")}
</select>

<input id="type" placeholder="Type" class="border p-2">
<button onclick="addIncident()" class="bg-red-600 text-white p-2">Ajouter</button>

<div id="incList"></div>
`;

renderInc();
}

function addIncident(){
let id=parseInt(document.getElementById("elev").value);
let type=document.getElementById("type").value;

incidents.push({id:Date.now(),eleve:id,type});
save();
pageIncidents();
}

function renderInc(){
document.getElementById("incList").innerHTML=incidents.map(i=>{
let e=eleves.find(x=>x.id===i.eleve);
return `<div class="bg-white p-3">${e?e.nom:"?"} - ${i.type}</div>`;
}).join("");
}

// ================= NOTIF =================
function pageNotif(){
document.getElementById("content").innerHTML=`
<textarea id="msg" class="border p-2 w-full"></textarea>
<button onclick="sendNotif()" class="bg-green-600 text-white p-2">Envoyer</button>
`;
}

function sendNotif(){
notifications.push({msg:document.getElementById("msg").value});
save();
toast("Envoyé");
}

// ================= RAPPORT =================
function pageRapport(){
document.getElementById("content").innerHTML=`
<button onclick="genPDF()" class="bg-blue-600 text-white p-3">PDF</button>
`;
}

function genPDF(){
const {jsPDF}=window.jspdf;
let doc=new jsPDF();
doc.text("Rapport scolaire",20,20);
doc.save("rapport.pdf");
}

// ================= STATS =================
function pageStats(){
let map={};
incidents.forEach(i=>{
let e=eleves.find(x=>x.id===i.eleve);
if(e){map[e.nom]=(map[e.nom]||0)+1;}
});

document.getElementById("content").innerHTML=`<canvas id="chart"></canvas>`;

new Chart(document.getElementById("chart"),{
type:"bar",
data:{
labels:Object.keys(map),
datasets:[{label:"Incidents",data:Object.values(map)}]
}
});
}

// ================= INIT =================
showPage("dashboard");

</script>

</body>
</html>
