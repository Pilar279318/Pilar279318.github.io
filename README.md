<!DOCTYPE html>
<html lang="pl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Zakłady Bukmacherskie</title>
<style>
body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; margin:0; padding:0; background:#f2f2f7; }
.container { padding:16px; max-width:600px; margin:auto; }
h2, h3 { text-align:center; }
button { width:100%; padding:16px; font-size:18px; border-radius:12px; border:none; margin-top:10px; background:#007aff; color:white; }
button:hover { opacity:0.9; }
input, select { width:100%; padding:14px; font-size:17px; border-radius:10px; border:1px solid #ccc; margin-top:10px; }
.event, .bet { background:white; padding:16px; border-radius:14px; margin-top:14px; box-shadow:0 2px 6px rgba(0,0,0,0.1); }
.admin-panel, #userPanel { display:none; }
</style>
</head>
<body>

<div class="container">

<h2>Zakłady Bukmacherskie</h2>

<!-- Wybór roli -->
<div id="roleSelect">
    <h3>Wybierz rolę:</h3>
    <button onclick="chooseRole('guest')">Gość – tylko obstawianie</button>
    <button onclick="chooseRole('admin')">Admin – dodawanie wydarzeń</button>
</div>

<!-- Logowanie Admina -->
<div id="adminLogin" class="admin-panel">
    <h3>Logowanie administratora</h3>
    <input id="adminPass" type="password" placeholder="Hasło administratora">
    <button onclick="loginAdmin()">Zaloguj</button>
</div>

<!-- Panel Admina -->
<div id="adminPanel" class="admin-panel">
    <h3>Dodaj wydarzenie</h3>
    <input id="eventName" placeholder="Nazwa wydarzenia">
    <input id="eventOdds" placeholder="Kurs (np. 1.85)">
    <button onclick="addEvent()">Dodaj wydarzenie</button>
</div>

<!-- Panel użytkownika -->
<div id="userPanel">
    <h3>Dostępne wydarzenia</h3>
    <div id="events"></div>

    <h3>Twoje zakłady</h3>
    <div id="bets"></div>
</div>

</div>

<script>
let events = JSON.parse(localStorage.getItem('events')) || [];
let bets = JSON.parse(localStorage.getItem('bets')) || [];

// Wybór roli
function chooseRole(role){
    document.getElementById('roleSelect').style.display='none';
    document.getElementById('userPanel').style.display='block';
    if(role==='admin'){
        document.getElementById('adminLogin').style.display='block';
    }
    renderEvents();
    renderBets();
}

// Logowanie Admina
function loginAdmin(){
    const pass = document.getElementById('adminPass').value;
    if(pass==='fusy123'){
        document.getElementById('adminLogin').style.display='none';
        document.getElementById('adminPanel').style.display='block';
        alert('Zalogowano jako admin');
    } else alert('Błędne hasło!');
}

// Dodawanie wydarzeń
function addEvent(){
    const name = document.getElementById('eventName').value;
    const odds = parseFloat(document.getElementById('eventOdds').value);
    if(!name || !odds){ alert('Wypełnij wszystkie pola poprawnie!'); return; }
    events.push({name, odds});
    localStorage.setItem('events', JSON.stringify(events));
    document.getElementById('eventName').value='';
    document.getElementById('eventOdds').value='';
    renderEvents();
}

// Renderowanie wydarzeń
function renderEvents(){
    const container = document.getElementById('events');
    container.innerHTML='';
    events.forEach((e,index)=>{
        container.innerHTML+=`
        <div class="event">
            <strong>${e.name}</strong><br>
            Kurs: <b>${e.odds}</b><br>
            <input id="bet${index}" placeholder="Kwota w zł">
            <button onclick="placeBet(${index})">Obstaw</button>
        </div>
        `;
    });
}

// Obstawianie
function placeBet(index){
    const value = parseFloat(document.getElementById('bet'+index).value);
    if(!value || value<=0){ alert('Wpisz poprawną kwotę!'); return; }
    const win = (value*events[index].odds).toFixed(2);
    bets.push({event:events[index].name, odds:events[index].odds, stake:value, potential:win});
    localStorage.setItem('bets', JSON.stringify(bets));
    renderBets();
    document.getElementById('bet'+index).value='';
}

// Renderowanie zakładów
function renderBets(){
    const container = document.getElementById('bets');
    container.innerHTML='';
    bets.forEach(b=>{
        container.innerHTML+=`
        <div class="bet">
            <strong>${b.event}</strong><br>
            Kurs: ${b.odds} | Kwota: ${b.stake} zł<br>
            Potencjalna wygrana: ${b.potential} zł
        </div>
        `;
    });
}

// Automatyczne wyświetlenie wydarzeń dla gościa przy starcie
if(events.length>0){
    document.getElementById('userPanel').style.display='block';
    renderEvents();
    renderBets();
}

// ---- Aktualizacja panelu gościa co 1 sekundę ----
setInterval(()=>{
    let storedEvents = JSON.parse(localStorage.getItem('events')) || [];
    if(JSON.stringify(storedEvents) !== JSON.stringify(events)){
        events = storedEvents;
        renderEvents();
    }
}, 1000);
</script>

</body>
</html>
