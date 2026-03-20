# SmartKisanAI
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Smart Kisan</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body {
    margin: 0;
    font-family: Arial;
    background: #eef2f3;
}

/* AUTH */
#authPage {
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    background: linear-gradient(green, darkgreen);
}

.auth-box {
    background: white;
    padding: 35px;
    border-radius: 15px;
    text-align: center;
    width: 280px;
}

.auth-box input {
    width: 100%;
    padding: 12px;
    margin: 10px 0;
}

.auth-box button {
    width: 100%;
    padding: 12px;
    background: green;
    color: white;
    border: none;
    cursor: pointer;
    font-size: 16px;
}

.switch {
    color: blue;
    cursor: pointer;
    margin-top: 10px;
}

/* DASHBOARD */
#dashboard { display: none; }

/* HEADER */
header {
    background: #2e7d32;
    color: white;
    display: flex;
    justify-content: space-between;
    padding: 15px 25px;
}

.logo {
    display: flex;
    align-items: center;
}

.logo img {
    width: 45px;
    margin-right: 10px;
}

/* MAIN */
.main {
    padding: 30px;
    text-align: center;
}

/* SCAN BUTTON */
.scan-btn {
    background: #ff9800;
    color: white;
    border: none;
    padding: 25px 40px;
    font-size: 22px;
    border-radius: 60px;
    cursor: pointer;
    margin-bottom: 30px;
    box-shadow: 0 5px 15px rgba(0,0,0,0.2);
}

/* CARDS */
.cards-container {
    max-width: 800px;
    margin: auto;
}

.card {
    background: white;
    padding: 25px;
    margin: 15px 0;
    border-radius: 15px;
    font-size: 18px;
    box-shadow: 0 5px 12px rgba(0,0,0,0.1);
    border-left: 6px solid green;
    text-align: left;
}

/* FOOTER */
footer {
    background: #2e7d32;
    color: white;
    text-align: center;
    padding: 20px;
    margin-top: 40px;
    font-size: 16px;
}
</style>
</head>

<body>

<!-- AUTH PAGE -->
<div id="authPage">
    <div class="auth-box">
        <h2 id="formTitle">Login</h2>

        <input type="text" id="email" placeholder="Email">
        <input type="password" id="password" placeholder="Password">

        <button id="authBtn" onclick="handleAuth()">Login</button>

        <div class="switch" id="toggleText" onclick="toggleForm()">
            Don't have account? Sign Up
        </div>
    </div>
</div>

<!-- DASHBOARD -->
<div id="dashboard">

<header>
    <div class="logo">
        <img src="https://cdn-icons-png.flaticon.com/512/2909/2909767.png">
        <h2>Smart Kisan</h2>
    </div>
    <div>
        <span id="location">Loading...</span> |
        <span id="temp">--°C</span>
    </div>
</header>

<div class="main">

    <!-- SCAN BUTTON -->
    <button class="scan-btn" onclick="scanCrop()">
        📷 Scan Crop
    </button>

    <input type="file" id="fileInput" hidden onchange="processImage()">

    <!-- RESULT BOXES -->
    <div class="cards-container">

        <div class="card">
            🌱 <b>Field Health:</b>
            <p id="health">-</p>
        </div>

        <div class="card">
            📈 <b>Growth Forecast:</b>
            <p id="forecast">-</p>
        </div>

        <div class="card">
            🌍 <b>Soil & Weather:</b>
            <p id="soil">-</p>
        </div>

        <div class="card">
            ⚠ <b>Alerts & Advice:</b>
            <p id="alerts">-</p>
        </div>

        <div class="card">
            ✅ <b>Task List:</b>
            <p id="tasks">-</p>
        </div>

    </div>

</div>

<!-- FOOTER -->
<footer>
    📞 Contact: +91 9876543210 <br><br>
    📧 Email: smartkisan@email.com
</footer>

</div>

<script>

// AUTH SYSTEM
let isLogin = true;

function toggleForm() {
    isLogin = !isLogin;

    document.getElementById("formTitle").innerText = isLogin ? "Login" : "Sign Up";
    document.getElementById("authBtn").innerText = isLogin ? "Login" : "Sign Up";

    document.getElementById("toggleText").innerText =
        isLogin
        ? "Don't have account? Sign Up"
        : "Already have account? Login";
}

function handleAuth() {
    let email = document.getElementById("email").value;
    let pass = document.getElementById("password").value;

    if(!email || !pass) {
        alert("Please fill all fields");
        return;
    }

    let users = JSON.parse(localStorage.getItem("users")) || [];

    if(isLogin) {
        let user = users.find(u => u.email === email && u.password === pass);

        if(user) {
            startApp();
        } else {
            alert("Invalid login credentials");
        }

    } else {
        let exists = users.find(u => u.email === email);

        if(exists) {
            alert("User already exists");
        } else {
            users.push({email: email, password: pass});
            localStorage.setItem("users", JSON.stringify(users));

            alert("Signup successful! Please login.");
            toggleForm();
        }
    }
}

function startApp() {
    document.getElementById("authPage").style.display = "none";
    document.getElementById("dashboard").style.display = "block";
    getLocation();
}

// LOCATION + WEATHER
function getLocation() {
    if(navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(pos => {
            let lat = pos.coords.latitude;
            let lon = pos.coords.longitude;

            fetch(`https://nominatim.openstreetmap.org/reverse?format=json&lat=${lat}&lon=${lon}`)
            .then(r => r.json())
            .then(d => {
                document.getElementById("location").innerText =
                    (d.address.village || "Village") + ", " + d.address.state;
            });

            fetch(`https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current_weather=true`)
            .then(r => r.json())
            .then(d => {
                document.getElementById("temp").innerText =
                    d.current_weather.temperature + "°C";
            });
        });
    }
}

// SCAN
function scanCrop() {
    document.getElementById("fileInput").click();
}

function processImage() {
    document.getElementById("health").innerText = "Healthy (85%)";
    document.getElementById("forecast").innerText = "High yield expected";
    document.getElementById("soil").innerText = "Moisture medium | Sunny";
    document.getElementById("alerts").innerText = "Pest risk detected. Use neem spray.";
    document.getElementById("tasks").innerText = "Irrigation in 2 days";
}

</script>

</body>
</html>
