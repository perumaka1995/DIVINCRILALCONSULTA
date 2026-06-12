<!DOCTYPE html>
<html lang="es">

<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>DIVINCRI LA LIBERTAD</title>

<script src="https://cdn.tailwindcss.com"></script>

<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">

<!-- FIREBASE COMPAT (IMPORTANTE) -->
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-storage-compat.js"></script>

<style>
body{
  font-family: Arial;
  background:#e7f1e7;
}

.header{
  background:linear-gradient(90deg,#064b22,#0b5a2a);
  color:white;
  text-align:center;
  padding:30px;
}

.title{
  font-size:3rem;
  font-weight:900;
  color:#ffd700;
}

.card{
  background:white;
  padding:20px;
  margin:20px auto;
  max-width:1000px;
  border-radius:10px;
  border-left:6px solid #075826;
}

input{
  width:100%;
  padding:10px;
  margin:5px 0;
  border:1px solid #ccc;
  border-radius:6px;
}

.btn{
  background:#075826;
  color:white;
  padding:10px;
  border-radius:6px;
  width:100%;
  margin-top:10px;
}

.btn:hover{background:#043f1a}

table{
  width:100%;
  border-collapse:collapse;
}

th{
  background:#075826;
  color:white;
  padding:10px;
}

td{
  border:1px solid #ddd;
  padding:10px;
  text-align:center;
}
</style>

</head>

<body>

<!-- HEADER -->
<div class="header">
  <h1 class="title">DIVINCRI LA LIBERTAD</h1>
  <p>Sistema de Registro y Consulta</p>
</div>

<!-- LOGIN -->
<div id="loginBox" class="card">
  <h2 class="text-xl font-bold mb-3">LOGIN</h2>

  <input id="email" placeholder="Correo">
  <input id="password" type="password" placeholder="Contraseña">

  <button class="btn" onclick="login()">Ingresar</button>
  <button class="btn" onclick="register()">Crear cuenta</button>
</div>

<!-- APP -->
<div id="app" style="display:none">

<!-- REGISTRO -->
<div class="card">

<h2 class="text-xl font-bold mb-3">Registrar Número</h2>

<input id="numero" placeholder="Número Telefónico">
<input id="solicitante" placeholder="Solicitante">
<input id="observacion" placeholder="Observación">
<input id="vinculados" placeholder="Vinculaciones">

<input type="file" id="archivo">

<button class="btn" onclick="guardar()">Guardar</button>

</div>

<!-- TABLA -->
<div class="card">

<h2 class="text-xl font-bold mb-3">Registros</h2>

<table>
<thead>
<tr>
<th>Número</th>
<th>Solicitante</th>
<th>Observación</th>
<th>Vinculados</th>
<th>Archivo</th>
<th>Acción</th>
</tr>
</thead>

<tbody id="tabla"></tbody>
</table>

</div>

</div>

<script>

// 🔥 TU CONFIG REAL (IMPORTANTE)
const firebaseConfig = {
  apiKey: "AIzaSyCDlz4fwWbfAEXQYrlarBs0B2zwDzdD454",
  authDomain: "divincri-la-libertad.firebaseapp.com",
  projectId: "divincri-la-libertad",
  storageBucket: "divincri-la-libertad.appspot.com",
  messagingSenderId: "1004762981616",
  appId: "1:1004762981616:web:e889f1cc1f4ce373d48e35"
};

firebase.initializeApp(firebaseConfig);

const auth = firebase.auth();
const db = firebase.firestore();
const storage = firebase.storage();

let editId = null;

// LOGIN
function login(){
auth.signInWithEmailAndPassword(email.value,password.value)
.then(()=>{
loginBox.style.display="none";
app.style.display="block";
load();
})
.catch(e=>alert(e.message));
}

// REGISTER
function register(){
auth.createUserWithEmailAndPassword(email.value,password.value)
.then(()=>alert("Cuenta creada"))
.catch(e=>alert(e.message));
}

// AUTH STATE
auth.onAuthStateChanged(user=>{
if(user){
loginBox.style.display="none";
app.style.display="block";
load();
}
});

// GUARDAR
function guardar(){

let file=document.getElementById("archivo").files[0];

let data={
numero:numero.value,
solicitante:solicitante.value,
observacion:observacion.value,
vinculados:vinculados.value,
fecha:new Date().toLocaleDateString(),
archivo:""
};

function save(url){
data.archivo=url;

if(editId){
db.collection("registros").doc(editId).set(data);
editId=null;
}else{
db.collection("registros").add(data);
}

clear();
load();
}

if(file){
let ref=storage.ref("archivos/"+file.name);
ref.put(file).then(s=>{
s.ref.getDownloadURL().then(url=>save(url));
});
}else{
save("");
}

}

// CARGAR
function load(){

db.collection("registros").onSnapshot(snap=>{

let t=document.getElementById("tabla");
t.innerHTML="";

snap.forEach(doc=>{
let d=doc.data();

t.innerHTML+=`
<tr>
<td>${d.numero}</td>
<td>${d.solicitante}</td>
<td>${d.observacion}</td>
<td>${d.vinculados}</td>
<td>${d.archivo?`<a href="${d.archivo}" target="_blank">Descargar</a>`:"-"}</td>
<td>
<button onclick="edit('${doc.id}',\`${d.numero}\`,\`${d.solicitante}\`,\`${d.observacion}\`,\`${d.vinculados}\`)">Editar</button>
<button onclick="del('${doc.id}')">Eliminar</button>
</td>
</tr>
`;
});

});

}

// EDITAR
function edit(id,n,s,o,v){
numero.value=n;
solicitante.value=s;
observacion.value=o;
vinculados.value=v;
editId=id;
}

// ELIMINAR
function del(id){
db.collection("registros").doc(id).delete();
}

// LIMPIAR
function clear(){
numero.value="";
solicitante.value="";
observacion.value="";
vinculados.value="";
archivo.value="";
}

</script>

</body>
</html>
