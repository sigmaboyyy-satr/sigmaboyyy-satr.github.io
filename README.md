# sigmaboyyy-satr.github.io
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Discord Bot Dashboard</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <header>
    <h1>🚀 My Discord Bot Dashboard</h1>
    <button id="loginBtn">Login with Discord</button>
    <button id="logoutBtn" style="display:none;">Logout</button>
  </header>

  <main>
    <section id="userInfo" style="display:none;">
      <h2>User Info</h2>
      <img id="avatar" src="" alt="Avatar">
      <p id="username"></p>
    </section>

    <section id="guilds" style="display:none;">
      <h2>Your Servers</h2>
      <ul id="guildList"></ul>
    </section>
  </main>

  <script src="script.js"></script>
</body>
</html>
body {
  font-family: Arial, sans-serif;
  text-align: center;
  background: #2c2f33;
  color: #fff;
}

header {
  margin: 20px;
}

button {
  background: #5865f2;
  border: none;
  color: #fff;
  padding: 10px 20px;
  margin: 10px;
  cursor: pointer;
  border-radius: 5px;
}

button:hover {
  background: #4752c4;
}

img {
  border-radius: 50%;
  margin: 10px;
}
// Replace with your Discord Application CLIENT ID (NOT bot token!)
const clientId = "1399476960404443136";  

// GitHub Pages will use this as redirect URI
const redirectUri = window.location.origin + window.location.pathname;

const loginBtn = document.getElementById("loginBtn");
const logoutBtn = document.getElementById("logoutBtn");
const userInfo = document.getElementById("userInfo");
const guilds = document.getElementById("guilds");
const username = document.getElementById("username");
const avatar = document.getElementById("avatar");
const guildList = document.getElementById("guildList");

// Build login URL (OAuth2 implicit grant)
const discordAuthUrl =
  `https://discord.com/api/oauth2/authorize` +
  `?client_id=${clientId}` +
  `&redirect_uri=${encodeURIComponent(redirectUri)}` +
  `&response_type=token` +
  `&scope=identify%20guilds`;

loginBtn.onclick = () => {
  window.location.href = discordAuthUrl;
};

logoutBtn.onclick = () => {
  localStorage.removeItem("discord_token");
  window.location.href = redirectUri;
};

// Extract token from URL hash
function getTokenFromHash() {
  if (window.location.hash) {
    const params = new URLSearchParams(window.location.hash.replace("#", "?"));
    return params.get("access_token");
  }
  return null;
}

async function fetchDiscordData(token) {
  const headers = { Authorization: `Bearer ${token}` };

  // Get user info
  const userRes = await fetch("https://discord.com/api/users/@me", { headers });
  const user = await userRes.json();

  username.textContent = `${user.username}#${user.discriminator}`;
  avatar.src = `https://cdn.discordapp.com/avatars/${user.id}/${user.avatar}.png`;
  userInfo.style.display = "block";

  // Get guilds
  const guildRes = await fetch("https://discord.com/api/users/@me/guilds", { headers });
  const guildsData = await guildRes.json();

  guildList.innerHTML = "";
  guildsData.forEach(g => {
    const li = document.createElement("li");
    li.textContent = g.name;
    guildList.appendChild(li);
  });

  guilds.style.display = "block";
  loginBtn.style.display = "none";
  logoutBtn.style.display = "inline-block";
}

// Check login state
let token = getTokenFromHash();
if (token) {
  localStorage.setItem("discord_token", token);
  window.location.hash = "";
}

token = localStorage.getItem("discord_token");
if (token) {
  fetchDiscordData(token);
}
