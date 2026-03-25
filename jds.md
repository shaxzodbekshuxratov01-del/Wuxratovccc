<!DOCTYPE html>
<html lang="uz">
<head>
<meta charset="UTF-8">
<title>Standoff 2 Skin Demo</title>
<style>
/* BODY & HEADER */
body {
    margin: 0; font-family: Arial, sans-serif;
    background: #0f172a; color: white;
    transition: background 0.3s, color 0.3s;
}
body.light { background:#f1f5f9; color:#1e293b; }

.header {
    background: #1b1f2a; padding: 20px;
    text-align: center; font-size: 28px;
    font-weight: bold; position: relative;
}
body.light .header { background:#e2e8f0; color:#1e293b; }

#balance {
    position: absolute; top: 20px; right: 20px;
    background: #161a26; padding: 8px 12px;
    border-radius: 8px; border: 2px solid #22c55e;
    display: flex; align-items: center; gap: 10px;
    transition: background 0.3s, color 0.3s;
}
body.light #balance { background:#cbd5e1; border-color:#16a34a; color:#1e293b; }

#balance button { background:#22c55e; border:none; padding:4px 8px; border-radius:5px; color:white; cursor:pointer; font-weight:bold; }
#balance button:hover { background:#16a34a; }

.navbar {
    background: #161a26; display: flex;
    justify-content: center; padding: 10px; flex-wrap: wrap;
}
body.light .navbar { background:#cbd5e1; color:#1e293b; }
.navbar a { color: white; margin: 0 15px; text-decoration: none; font-weight: bold; transition: color 0.3s; }
body.light .navbar a { color:#1e293b; }
.navbar a:hover { color: #22c55e; }

/* CONTAINER & SKINS */
.container { max-width: 1200px; margin: 20px auto; padding:0 10px; }
.skins {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
    gap: 15px;
}

/* CARD & HOVER */
.card {
    background: #1e293b; border-radius:10px; padding:15px; text-align:center;
    transition: transform 0.3s, box-shadow 0.3s; position:relative; overflow:hidden;
}
body.light .card { background:#e2e8f0; color:#1e293b; }
.card:hover { transform:scale(1.08); box-shadow:0 0 20px rgba(34,197,94,0.7); }
.card img { width:100%; border-radius:8px; margin-bottom:10px; object-fit:contain; transition: transform 0.3s; }
.card:hover img { transform:scale(1.05); }
.card p { margin:5px 0; opacity:1; transition:opacity 0.3s; }
.card button { padding:8px 10px; background:#22c55e; border:none; border-radius:5px; color:white; cursor:pointer; font-weight:bold; margin-top:5px; transition:background 0.3s, transform 0.3s; }
.card:hover button { transform:translateY(-2px); background:#16a34a; }

/* FILTER BUTTONS */
.filter-container { text-align:center; margin-bottom:20px; }
.filter-container button { margin:5px; padding:6px 12px; cursor:pointer; border-radius:5px; border:none; font-weight:bold; background:#22c55e; color:white; transition:background 0.3s; }
.filter-container button:hover { background:#16a34a; }

/* OVERLAY & MODAL */
#overlay { display:none; position:fixed; top:0; left:0; width:100%; height:100%; background: rgba(0,0,0,0.7); z-index:900; }
.modal { display:none; position:fixed; top:50%; left:50%; transform:translate(-50%,-50%); background:#1e293b; border:2px solid #22c55e; border-radius:12px; padding:20px; width:90%; max-width:350px; text-align:center; z-index:1000; overflow:hidden; }
body.light .modal { background:#e2e8f0; border-color:#16a34a; color:#1e293b; }
.modal h2 { color:#22c55e; }
.modal button.closeBtn { margin-top:10px; background:#dc2626; }
.modal button.closeBtn:hover { background:#b91c1c; }
.modal input { width:80%; padding:8px; margin-top:10px; border-radius:5px; border:none; }

/* RESPONSIVE */
@media(max-width:500px){ .header{font-size:22px;} .navbar a{margin:5px;font-size:14px;} .card{padding:10px;} button{font-size:14px;} }
</style>
</head>
<body>

<div class="header">
    Standoff 2 Skin Shop
    <div id="balance">💰 100,000 so'm <button onclick="openAddBalance()">+</button></div>
</div>

<div class="navbar">
    <a href="#">Bosh sahifa</a><a href="#">Skinlar</a><a href="#">Kontakt</a>
</div>

<div class="filter-container">
    <button onclick="filterSkins('all')">Barchasi</button>
    <button onclick="filterSkins('rifle')">Rifled Guns</button>
    <button onclick="filterSkins('pistol')">Pistols</button>
    <button onclick="filterSkins('knife')">Knives</button>
    <button onclick="toggleMode()">🌓 Light/Dark</button>
</div>

<div class="container">
    <div class="skins" id="skinContainer"></div>
</div>

<div id="overlay"></div>

<!-- ID Modal -->
<div class="modal" id="idModal">
    <h2>ID kiriting</h2>
    <input id="playerId" placeholder="oʻyinchi ID">
    <br><br>
    <button onclick="confirmSkin()">Tasdiqlash</button>
    <button class="closeBtn" onclick="closeModal()">❌ Bekor</button>
</div>

<!-- Skin tushdi modal -->
<div class="modal" id="okModal">
    <h2>🎉 Skin tushdi! 🎉</h2>
    <p id="okText"></p>
    <button class="closeBtn" onclick="closeOk()">OK</button>
</div>

<!-- Balans qo'shish modal -->
<div class="modal" id="addBalanceModal">
    <h2>Balansni oshirish</h2>
    <input id="addBalanceInput" type="number" placeholder="Qo'shmoqchi bo'lgan summa">
    <br><br>
    <button onclick="confirmAddBalance()">Qo'shish</button>
    <button class="closeBtn" onclick="closeAddBalance()">❌ Bekor</button>
</div>

<script>
let balance = 100000;
let selectedSkin="", selectedPrice=0;
let isLight = false;

const sounds = {
    click: "https://cdn.pixabay.com/download/audio/2023/06/16/audio_9f0c8d7d6b.mp3",
    coin: "https://cdn.pixabay.com/download/audio/2022/03/15/audio_3d9f8a.mp3",
    popup: "https://cdn.pixabay.com/download/audio/2022/03/22/audio_5fcb8e.mp3",
    spark: "https://cdn.pixabay.com/download/audio/2023/07/01/audio_4d6c7a.mp3"
};

function playSound(type){
    const audio = new Audio(sounds[type]);
    audio.volume = 0.6;
    audio.play();
}

const skins = [
    {name:"AKR Necromancer", price:75000, type:"rifle", img:"https://www.kindpng.com/picc/m/564-5649096_standoff2-2-karambit-standoff-2-karambit-hd-png.png"},
    {name:"M4A1‑S", price:60000, type:"rifle", img:"https://www.kindpng.com/picc/m/564-5649096_standoff2-2-karambit-standoff-2-karambit-hd-png.png"},
    {name:"AWP Standoff", price:82000, type:"rifle", img:"https://www.kindpng.com/picc/m/564-5649096_standoff2-2-karambit-standoff-2-karambit-hd-png.png"},
    {name:"Karambit Standoff", price:90000, type:"knife", img:"https://www.kindpng.com/picc/m/564-5649096_standoff2-2-karambit-standoff-2-karambit-hd-png.png"},
    {name:"Desert Eagle Blaze", price:65000, type:"pistol", img:"https://www.kindpng.com/picc/m/564-5649096_standoff2-2-karambit-standoff-2-karambit-hd-png.png"},
    {name:"P90 Neon Rider", price:70000, type:"rifle", img:"https://www.kindpng.com/picc/m/564-5649096_standoff2-2-karambit-standoff-2-karambit-hd-png.png"},
    {name:"USP Shadow", price:58000, type:"pistol", img:"https://www.kindpng.com/picc/m/564-5649096_standoff2-2-karambit-standoff-2-karambit-hd-png.png"},
    {name:"M4A4 Dragon", price:85000, type:"rifle", img:"https://www.kindpng.com/picc/m/564-5649096_standoff2-2-karambit-standoff-2-karambit-hd-png.png"},
    {name:"Knife Phantom", price:95000, type:"knife", img:"https://www.kindpng.com/picc/m/564-5649096_standoff2-2-karambit-standoff-2-karambit-hd-png.png"},
    {name:"AK-47 Inferno", price:88000, type:"rifle", img:"https://www.kindpng.com/picc/m/564-5649096_standoff2-2-karambit-standoff-2-karambit-hd-png.png"},
];

const container = document.getElementById("skinContainer");

function showSkins(filter="all"){
    container.innerHTML = "";
    skins.forEach(s => {
        if(filter==="all" || s.type===filter){
            const card = document.createElement("div");
            card.className="card";
            card.innerHTML = `
                <img src="${s.img}" alt="${s.name}">
                <p>${s.name}</p><p>${s.price.toLocaleString()} so'm</p>
                <button onclick="buySkin('${s.name}',${s.price})">Sotib olish</button>`;
            container.appendChild(card);
        }
    });
}
showSkins();
function filterSkins(type){ showSkins(type); }

function toggleMode(){
    document.body.classList.toggle("light");
    isLight = !isLight;
}

function buySkin(name,price){
    if(balance < price){ alert("Balans yetarli emas!"); return; }
    selectedSkin=name; selectedPrice=price;
    playSound("click");
    document.getElementById("overlay").style.display="block";
    document.getElementById("idModal").style.display="block";
    playSound("popup");
}

function animateBalance(change){
    let start = balance;
    let end = balance + change;
    const duration = 500;
    const startTime = performance.now();
    function animate(time){
        let progress = (time - startTime)/duration;
        if(progress>1) progress=1;
        let current = Math.floor(start + (end-start)*progress);
        document.getElementById("balance").innerHTML = "💰 "+current.toLocaleString()+" so'm <button onclick='openAddBalance()'>+</button>";
        if(progress<1) requestAnimationFrame(animate);
        else balance = end;
    }
    requestAnimationFrame(animate);
}

function closeModal(){ document.getElementById("overlay").style.display="none"; document.getElementById("idModal").style.display="none"; }
function closeOk(){ document.getElementById("overlay").style.display="none"; document.getElementById("okModal").style.display="none"; }
function openAddBalance(){ 
    document.getElementById("overlay").style.display="block"; 
    document.getElementById("addBalanceModal").style.display="block"; 
    playSound("popup");
}
function closeAddBalance(){ document.getElementById("overlay").style.display="none"; document.getElementById("addBalanceModal").style.display="none"; }

function confirmAddBalance(){
    const amt = parseInt(document.getElementById("addBalanceInput").value);
    if(isNaN(amt)||amt<=0){ alert("Iltimos, to'g'ri summa kiriting!"); return; }
    animateBalance(amt);
    closeAddBalance();
    document.getElementById("okText").innerText = "💰 Balansga "+amt.toLocaleString()+" so'm qo'shildi!";
    document.getElementById("overlay").style.display="block";
    document.getElementById("okModal").style.display="block";
    playSound("coin");
}

// Particle effekt
function createParticles(type){
    const modal = document.getElementById("okModal");
    let color;
    if(type==="rifle") color="#3b82f6";
    else if(type==="pistol") color="#facc15";
    else if(type==="knife") color="#ef4444";
    else color="#22c55e";

    for(let i=0;i<25;i++){
        const particle = document.createElement("div");
        particle.style.position="absolute";
        particle.style.width="6px";
        particle.style.height="6px";
        particle.style.backgroundColor=color;
        particle.style.borderRadius="50%";
        particle.style.top="50%";
        particle.style.left="50%";
        particle.style.pointerEvents="none";
        particle.style.opacity=1;
        particle.style.transform=`translate(${Math.random()*80-40}px, ${Math.random()*80-40}px)`;
        particle.style.transition="all 0.8s ease-out";
        modal.appendChild(particle);
        setTimeout(()=>{
            particle.style.transform=`translate(${Math.random()*200-100}px, ${-Math.random()*200}px)`;
            particle.style.opacity=0;
        },50);
        setTimeout(()=>{ particle.remove(); },800);
    }
}

function confirmSkin(){
    const id = document.getElementById("playerId").value.trim();
    if(!id){ alert("Iltimos ID kiriting!"); return; }
    animateBalance(-selectedPrice);
    closeModal();
    document.getElementById("okText").innerText = selectedSkin + " sizga berildi!";
    document.getElementById("overlay").style.display="block";
    document.getElementById("okModal").style.display="block";
    playSound("spark");
    const skinType = skins.find(s=>s.name===selectedSkin).type;
    createParticles(skinType);
}
</script>

</body>
</html>