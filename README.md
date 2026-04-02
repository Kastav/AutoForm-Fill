<html lang="en"> 
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Govt Exam Assistant Simple</title>

<!-- Firebase SDK (simple) -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-storage.js"></script>

<style>
body {font-family: Arial; background:#0f172a; color:white; padding:20px; transition:0.3s;}
.light {background:white; color:black;}
h1{text-align:center; color:#38bdf8;}
.card {background:#1e293b; padding:15px; margin:15px 0; border-radius:10px;}
.light .card {background:#f1f1f1;}
input, select {width:100%; padding:10px; margin:8px 0; border-radius:5px; border:none;}
button {padding:10px; background:#38bdf8; border:none; width:100%; border-radius:5px; font-weight:bold; margin-top:5px;}
.result {background:#334155; padding:10px; margin-top:10px; border-radius:5px; height:auto;}
.light .result {background:#ddd;}
#chatResult {height:120px; overflow-y:auto;}
</style>
</head>
<body>

<h1>🚀 Govt Exam Assistant</h1>

<button onclick="toggleMode()">🌙 Toggle Dark/Light</button>
<button onclick="toggleLang()">🌐 हिंदी / English</button>

<div class="card">
<h3>🔑 Login / Signup</h3>
<input id="email" placeholder="Email">
<input id="password" type="password" placeholder="Password">
<button onclick="signup()">Signup</button>
<button onclick="login()">Login</button>
<button onclick="logout()">Logout</button>
<div id="authStatus" class="result"></div>
</div>

<div class="card">
<h3 id="profileTitle">👤 Profile</h3>
<input id="name" placeholder="Name">
<input id="age" type="number" placeholder="Age">
<select id="education">
<option value="">Select Education</option>
<option>12th</option>
<option>Graduate</option>
</select>
<button onclick="startVoice()">🎤 Speak Name</button>
<button onclick="saveProfile()">Save Profile</button>
<div id="profileResult" class="result"></div>
</div>

<div class="card">
<h3>🎯 Exams</h3>
<button onclick="suggestExams()">Find Exams</button>
<div id="exams" class="result"></div>
</div>

<div class="card">
<h3>📄 Document Upload</h3>
<input type="file" id="photo">
<button onclick="uploadDoc()">Upload Document</button>
<div id="docResult" class="result"></div>
</div>

<div class="card">
<h3>🤖 AI Chat</h3>
<input id="chatInput" placeholder="Ask something...">
<button onclick="chatAI()">Ask</button>
<div id="chatResult" class="result"></div>
</div>

<script>
// 🔹 Firebase simple initialization (only apiKey)
const firebaseConfig = { apiKey: "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" };
firebase.initializeApp(firebaseConfig);
const auth = firebase.auth();
const db = firebase.firestore();
const storage = firebase.storage();

// Global
let lang="en";

// Dark/Light Mode
function toggleMode(){ document.body.classList.toggle("light"); }

// Language
function toggleLang(){
  lang = lang==="en"?"hi":"en";
  document.getElementById("profileTitle").innerText = lang==="en"?"👤 Profile":"👤 प्रोफाइल";
}

// Authentication
function signup(){
  let email=document.getElementById("email").value;
  let pass=document.getElementById("password").value;
  auth.createUserWithEmailAndPassword(email,pass)
    .then(()=>document.getElementById("authStatus").innerText="✅ Signup Success")
    .catch(e=>document.getElementById("authStatus").innerText=e.message);
}
function login(){
  let email=document.getElementById("email").value;
  let pass=document.getElementById("password").value;
  auth.signInWithEmailAndPassword(email,pass)
    .then(()=>document.getElementById("authStatus").innerText="✅ Login Success")
    .catch(e=>document.getElementById("authStatus").innerText=e.message);
}
function logout(){ auth.signOut().then(()=>document.getElementById("authStatus").innerText="Logged out"); }

// Profile Save
function saveProfile(){
  let user = auth.currentUser;
  if(!user){ alert("Login first!"); return; }
  let profileData={
    name: document.getElementById("name").value,
    age: document.getElementById("age").value,
    education: document.getElementById("education").value,
    timestamp: new Date()
  };
  db.collection("users").doc(user.uid).set(profileData)
    .then(()=> document.getElementById("profileResult").innerText="✅ Profile Saved")
    .catch(e=> document.getElementById("profileResult").innerText=e.message);
}

// Exam Suggestions
function suggestExams(){
  let user=auth.currentUser;
  if(!user){ alert("Login first!"); return; }
  db.collection("users").doc(user.uid).get().then(doc=>{
    if(!doc.exists){ alert("Save profile first"); return; }
    let data = doc.data();
    let result = (data.education==="Graduate") ? "UPSC, SSC CGL, IBPS PO" : "SSC CHSL, Railway";
    document.getElementById("exams").innerText=result;
  });
}

// Document Upload
function uploadDoc(){
  let user=auth.currentUser;
  if(!user){ alert("Login first!"); return; }
  let file = document.getElementById("photo").files[0];
  if(!file){ alert("Select file first"); return; }
  let storageRef = storage.ref().child(`documents/${user.uid}/${file.name}`);
  storageRef.put(file)
    .then(()=>document.getElementById("docResult").innerText="✅ Document Uploaded")
    .catch(e=>document.getElementById("docResult").innerText=e.message);
}

// AI Chat
function chatAI(){
  let user = auth.currentUser;
  if(!user){ alert("Login first!"); return; }
  let input = document.getElementById("chatInput").value.toLowerCase();
  let reply="";
  if(input.includes("exam")) reply="You can apply for UPSC, SSC, IBPS.";
  else if(input.includes("age")) reply="Most exams require 18–32 years.";
  else reply="Try asking about application or documents.";
  document.getElementById("chatResult").innerText=reply;
  db.collection("users").doc(user.uid).collection("chat").add({ query: input, response: reply, time: new Date() });
}

// Voice Input
function startVoice(){
  let recognition=new (window.SpeechRecognition || window.webkitSpeechRecognition)();
  recognition.start();
  recognition.onresult=function(event){ document.getElementById("name").value = event.results[0][0].transcript; }
}
</script>
</body>
</html>
