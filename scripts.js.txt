// scripts.js

// Paste your Firebase config here 👇
const firebaseConfig = {
  apiKey: "PASTE_YOURS",
  authDomain: "PASTE_YOURS",
  projectId: "PASTE_YOURS",
  storageBucket: "PASTE_YOURS",
  messagingSenderId: "PASTE_YOURS",
  appId: "PASTE_YOURS"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();
const auth = firebase.auth();

const loginSection = document.getElementById("loginSection");
const mainSection = document.getElementById("mainSection");

function loginUser() {
  const email = document.getElementById("loginEmail").value;
  const password = document.getElementById("loginPassword").value;
  auth.signInWithEmailAndPassword(email, password)
    .then(() => {
      loginSection.style.display = "none";
      mainSection.style.display = "block";
      loadTickets();
      loadChat();
    })
    .catch((error) => {
      alert("Login failed: " + error.message);
    });
}

document.getElementById("ticketForm").addEventListener("submit", function (e) {
  e.preventDefault();
  const ticket = {
    name: document.getElementById("name").value,
    make: document.getElementById("make").value,
    model: document.getElementById("model").value,
    serial: document.getElementById("serial").value,
    sla: document.getElementById("sla").value,
    issue: document.getElementById("issue").value,
    description: document.getElementById("description").value,
    priority: document.getElementById("priority").value,
    timestamp: new Date(),
    created_by: auth.currentUser.email
  };
  db.collection("tickets").add(ticket).then(() => {
    alert("Ticket raised successfully!");
    document.getElementById("ticketForm").reset();
    loadTickets();
  });
});

function loadTickets() {
  db.collection("tickets").orderBy("timestamp", "desc").onSnapshot((snapshot) => {
    const ticketList = document.getElementById("ticketList");
    ticketList.innerHTML = "";
    snapshot.forEach((doc) => {
      const data = doc.data();
      const li = document.createElement("li");
      li.textContent = `${data.name} | ${data.make} ${data.model} | ${data.issue} | ${data.priority} | SLA: ${data.sla}`;
      ticketList.appendChild(li);
    });
  });
}

function sendChat() {
  const message = document.getElementById("chatMessage").value;
  db.collection("chat").add({
    message,
    sender: auth.currentUser.email,
    timestamp: new Date()
  }).then(() => {
    document.getElementById("chatMessage").value = "";
  });
}

function loadChat() {
  db.collection("chat").orderBy("timestamp", "asc").onSnapshot((snapshot) => {
    const chatBox = document.getElementById("chatBox");
    chatBox.innerHTML = "";
    snapshot.forEach((doc) => {
      const data = doc.data();
      const div = document.createElement("div");
      div.textContent = `${data.sender}: ${data.message}`;
      chatBox.appendChild(div);
    });
  });
}