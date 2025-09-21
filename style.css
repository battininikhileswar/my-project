// ---------- CONFIG ----------
const LOCAL_USERS_KEY = "moodapp_users_v1";
const LOCAL_CURRENT_KEY = "moodapp_current_user_v1";

// Mood mapping with songs, quotes, and YouTube videos
const moodContent = {
  happy: {
    playlist: ["happy_song.mp3"],
    quotes: ["Keep smiling!", "Joy is contagious!"],
    videos: ["https://www.youtube.com/embed/z0GKGpObgPY"]
  },
  sad: {
    playlist: ["calm_song.mp3"],
    quotes: ["This too shall pass.", "Every day is a new beginning."],
    videos: ["https://www.youtube.com/embed/EXfFBEQpSM4"]
  },
  angry: {
    playlist: ["relaxing_song.mp3"],
    quotes: ["Take a deep breath.", "Calmness is power."],
    videos: ["https://www.youtube.com/embed/r8rw5FPMIys"]
  },
  neutral: {
    playlist: ["neutral_song.mp3"],
    quotes: ["Stay balanced.", "Keep going!"],
    videos: ["https://www.youtube.com/embed/DyDfgMOUjCI"]
  },
  surprised: {
    playlist: ["energetic_song.mp3"],
    quotes: ["Expect the unexpected!", "Life is full of surprises!"],
    videos: ["https://www.youtube.com/embed/ZXsQAXx_ao0"]
  }
};

let currentMood = "";
let currentUsername = null;

// ---------- HELPERS: localStorage user management ----------
function loadUsers() {
  const raw = localStorage.getItem(LOCAL_USERS_KEY);
  if (!raw) return [];
  try { return JSON.parse(raw); } catch { return []; }
}
function saveUsers(users) {
  localStorage.setItem(LOCAL_USERS_KEY, JSON.stringify(users));
}
function findUser(username) {
  const users = loadUsers();
  return users.find(u => u.username === username);
}
function saveCurrent(username) {
  localStorage.setItem(LOCAL_CURRENT_KEY, username);
}
function clearCurrent() {
  localStorage.removeItem(LOCAL_CURRENT_KEY);
}
function getCurrent() {
  return localStorage.getItem(LOCAL_CURRENT_KEY);
}

// ---------- CRYPTO: salted SHA-256 ----------
function randomSaltHex(bytes = 16) {
  const arr = new Uint8Array(bytes);
  crypto.getRandomValues(arr);
  return Array.from(arr).map(b => b.toString(16).padStart(2, "0")).join("");
}

async function hashWithSaltHex(password, saltHex) {
  const enc = new TextEncoder();
  // combine salt + password
  const data = enc.encode(saltHex + password);
  const hashBuffer = await crypto.subtle.digest("SHA-256", data);
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  const hashHex = hashArray.map(b => b.toString(16).padStart(2, "0")).join("");
  return hashHex;
}

// ---------- AUTH: signup / login / logout ----------
function showSignup() {
  document.getElementById("loginForm").style.display = "none";
  document.getElementById("signupForm").style.display = "block";
}
function showLogin() {
  document.getElementById("loginForm").style.display = "block";
  document.getElementById("signupForm").style.display = "none";
}

async function signup() {
  const username = document.getElementById("signupUsername").value.trim();
  const password = document.getElementById("signupPassword").value;

  if (!username || !password) {
    alert("Enter both username and password.");
    return;
  }
  if (findUser(username)) {
    alert("Username already exists. Choose another.");
    return;
  }

  const salt = randomSaltHex(16);
  const passHash = await hashWithSaltHex(password, salt);

  const users = loadUsers();
  users.push({
    username,
    salt,
    passHash,
    history: [] // array of {timestamp, type, value}
  });
  saveUsers(users);
  alert("Account created. Please log in.");
  showLogin();
  document.getElementById("signupUsername").value = "";
  document.getElementById("signupPassword").value = "";
}

async function login() {
  const username = document.getElementById("loginUsername").value.trim();
  const password = document.getElementById("loginPassword").value;

  if (!username || !password) {
    alert("Enter both username and password.");
    return;
  }
  const user = findUser(username);
  if (!user) {
    alert("No such user. Please sign up.");
    return;
  }
  const candidateHash = await hashWithSaltHex(password, user.salt);
  if (candidateHash === user.passHash) {
    currentUsername = username;
    saveCurrent(username);
    document.getElementById("authBox").style.display = "none";
    document.getElementById("app").style.display = "block";
    document.getElementById("currentUserLabel").innerText = username;
    loadUserHistoryToUI();
    // start video & detection after login
    startVideo();
    detectMood();
  } else {
    alert("Invalid credentials.");
  }
}

function logout() {
  clearCurrent();
  currentUsername = null;
  // stop video feed if needed (best-effort)
  const v = document.getElementById("video");
  if (v && v.srcObject) {
    v.srcObject.getTracks().forEach(t => t.stop());
  }
  document.getElementById("app").style.display = "none";
  document.getElementById("authBox").style.display = "block";
  document.getElementById("currentUserLabel").innerText = "";
}

// ---------- HISTORY helpers ----------
function pushHistoryEntry(username, type, value) {
  const users = loadUsers();
  const user = users.find(u => u.username === username);
  if (!user) return;
  user.history = user.history || [];
  user.history.unshift({
    timestamp: new Date().toISOString(),
    type,
    value
  });
  // keep last 200 entries max
  if (user.history.length > 200) user.history.length = 200;
  saveUsers(users);
  // refresh UI
  loadUserHistoryToUI();
}

function loadUserHistoryToUI() {
  const histDiv = document.getElementById("historyList");
  histDiv.innerHTML = "";
  if (!currentUsername) {
    histDiv.innerText = "No user logged in.";
    return;
  }
  const user = findUser(currentUsername);
  if (!user || !user.history || user.history.length === 0) {
    histDiv.innerText = "No history yet.";
    return;
  }
  const ul = document.createElement("ul");
  ul.style.paddingLeft = "14px";
  user.history.forEach(entry => {
    const li = document.createElement("li");
    const t = new Date(entry.timestamp).toLocaleString();
    li.innerText = `[${t}] ${entry.type.toUpperCase()}: ${entry.value}`;
    ul.appendChild(li);
  });
  histDiv.appendChild(ul);
}

function clearHistory() {
  if (!currentUsername) return;
  if (!confirm("Clear history for current user?")) return;
  const users = loadUsers();
  const user = users.find(u => u.username === currentUsername);
  if (user) {
    user.history = [];
    saveUsers(users);
    loadUserHistoryToUI();
  }
}

// ---------- WEBCAM + MOOD DETECTION ----------
let videoStarted = false;
async function startVideo() {
  if (videoStarted) return;
  videoStarted = true;
  const video = document.getElementById("video");
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ video: true });
    video.srcObject = stream;
  } catch (err) {
    console.error("Camera error:", err);
    alert("Cannot access camera. Check permissions.");
  }
}

async function detectMood() {
  // Load models (only once)
  await faceapi.nets.tinyFaceDetector.loadFromUri("/models");
  await faceapi.nets.faceExpressionNet.loadFromUri("/models");

  const video = document.getElementById("video");
  video.addEventListener("play", () => {
    setInterval(async () => {
      const detections = await faceapi.detectAllFaces(video, new faceapi.TinyFaceDetectorOptions()).withFaceExpressions();
      if (detections && detections[0]) {
        const expressions = detections[0].expressions;
        const mood = Object.entries(expressions).reduce((a,b) => a[1] > b[1] ? a : b)[0];
        if (mood !== currentMood) {
          currentMood = mood;
          // Save history event
          if (currentUsername) pushHistoryEntry(currentUsername, "mood", mood);
          // Update UI & content
          playMoodMusic(mood);
          showQuote(mood);
          showVideo(mood);
          console.log("Detected mood:", mood);
        }
      }
    }, 2000);
  });
}

// ---------- PLAY / SHOW ----------
function playMoodMusic(mood) {
  const musicPlayer = document.getElementById("musicPlayer");
  const songs = moodContent[mood]?.playlist || [];
  if (songs.length > 0) {
    musicPlayer.src = songs[0];
    // browsers may block autoplay until user interacted
    musicPlayer.play().catch(()=> {
      // ignore -- user can click play
    });
  }
}

function showQuote(mood) {
  const quoteBox = document.getElementById("quoteBox");
  const quotes = moodContent[mood]?.quotes || [];
  if (quotes.length > 0) {
    const randomQuote = quotes[Math.floor(Math.random() * quotes.length)];
    quoteBox.innerText = randomQuote;
  } else {
    quoteBox.innerText = "";
  }
}

function showVideo(mood) {
  const videoBox = document.getElementById("videoBox");
  const videos = moodContent[mood]?.videos || [];
  if (videos.length > 0) {
    videoBox.innerHTML = `<iframe width="560" height="315" src="${videos[0]}" frameborder="0" allowfullscreen></iframe>`;
  } else {
    videoBox.innerHTML = "";
  }
}

// ---------- SEARCH ----------
function searchContent() {
  const query = document.getElementById("searchBar").value.trim().toLowerCase();
  if (!query) { alert("Type a mood/keyword like 'happy' or 'sad'"); return; }

  // Save search event
  if (currentUsername) pushHistoryEntry(currentUsername, "search", query);

  for (const mood in moodContent) {
    if (mood.includes(query)) {
      playMoodMusic(mood);
      showQuote(mood);
      showVideo(mood);
      return;
    }
  }
  alert("No content found for: " + query);
}

// ---------- AUTO-LOGIN ON PAGE LOAD ----------
window.addEventListener("load", () => {
  const saved = getCurrent();
  if (saved) {
    const user = findUser(saved);
    if (user) {
      currentUsername = saved;
      document.getElementById("authBox").style.display = "none";
      document.getElementById("app").style.display = "block";
      document.getElementById("currentUserLabel").innerText = currentUsername;
      loadUserHistoryToUI();
      // we begin camera & detection when page loads for logged-in users
      startVideo();
      detectMood();
      return;
    } else {
      clearCurrent();
    }
  }
  // else, remain on auth screen
});
