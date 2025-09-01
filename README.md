<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<title>Alper'in Yapay Zeka Asistanı</title>
<style>
  body { margin:0; font-family:'Segoe UI', sans-serif; background:#f0f0f0; height:100vh; display:flex; }
  #sidebar { width:220px; background:#ffffff; border-right:1px solid #ddd; padding:10px; overflow-y:auto; }
  #sidebar h3 { text-align:center; color:#1a73e8; margin-top:0; }
  .history-item { padding:8px; margin:5px 0; background:#e8eaf6; border-radius:6px; cursor:pointer; transition:0.2s; font-size:14px; }
  .history-item:hover { background:#c5cae9; }

  #main { flex:1; display:flex; flex-direction:column; }
  #header { background:#1a73e8; color:white; padding:15px; text-align:center; font-size:20px; font-weight:bold; }
  #messages { flex:1; padding:20px; overflow-y:auto; display:flex; flex-direction:column; gap:10px; background:#f0f0f0; }
  .msg { max-width:70%; padding:12px 15px; border-radius:18px; word-wrap:break-word; line-height:1.5; animation:fadeIn 0.3s; }
  .user { background:#1a73e8; color:white; align-self:flex-end; border-bottom-right-radius:4px; }
  .bot { background:#e0e0e0; color:black; align-self:flex-start; border-bottom-left-radius:4px; position:relative; }
  .bot.typing::after { content:'...'; display:inline-block; margin-left:5px; animation:blink 1s infinite; }
  @keyframes blink { 0%,100%{opacity:0;}50%{opacity:1;} }
  @keyframes fadeIn { from{opacity:0;} to{opacity:1;} }

  #inputBox { display:flex; padding:10px; border-top:1px solid #ddd; gap:5px; background:white; }
  #userInput { flex:1; padding:12px; border-radius:8px 0 0 8px; border:1px solid #ccc; font-size:16px; }
  #sendBtn { padding:12px 20px; border:none; border-radius:0 8px 8px 0; background:#1a73e8; color:white; cursor:pointer; font-weight:bold; transition:0.2s; }
  #sendBtn:hover { background:#0d47a1; transform:scale(1.05); }
</style>
</head>
<body>

<div id="sidebar">
  <h3>Geçmişler</h3>
  <div id="history"></div>
</div>

<div id="main">
  <div id="header">Alper'in Yapay Zeka Asistanı</div>
  <div id="messages"></div>
  <div id="inputBox">
    <input type="text" id="userInput" placeholder="Bir şeyler yaz...">
    <button id="sendBtn">Gönder</button>
  </div>
</div>

<script>
const sendBtn=document.getElementById("sendBtn");
const userInput=document.getElementById("userInput");
const messages=document.getElementById("messages");
const historyDiv=document.getElementById("history");

let chatHistory = JSON.parse(localStorage.getItem("chatHistory")) || [];

// LocalStorage'daki geçmişi yükle
chatHistory.forEach(h => addHistory(h.summary, h.user, h.bot));

function addMessage(text,sender,typing=false){
  const msg=document.createElement("div");
  msg.classList.add("msg",sender);
  if(typing) msg.classList.add("typing");
  msg.innerText=text;
  messages.appendChild(msg);
  messages.scrollTop=messages.scrollHeight;
  return msg;
}

function summarizeMessage(text){
  // Basit özetleme: ilk 1-2 kelime
  return text.split(" ").slice(0,2).join(" ") + (text.split(" ").length>2 ? "..." : "");
}

function addHistory(summary,userMsg,botMsg){
  const item=document.createElement("div");
  item.classList.add("history-item");
  item.innerText=summary;
  item.onclick=()=>{ loadHistory(userMsg,botMsg); };
  historyDiv.appendChild(item);
}

function loadHistory(userMsg,botMsg){
  messages.innerHTML="";
  addMessage(userMsg,"user");
  addMessage(botMsg,"bot");
}

async function sendMessage(){
  const text=userInput.value.trim();
  if(!text) return;
  addMessage(text,"user");
  userInput.value="";
  const typingMsg=addMessage("Yazıyor...","bot",true);

  const API_KEY="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"; // OpenAI API anahtarını buraya ekle

  try{
    const response=await fetch("https://api.openai.com/v1/chat/completions",{
      method:"POST",
      headers:{
        "Content-Type":"application/json",
        "Authorization":`Bearer ${API_KEY}`
      },
      body:JSON.stringify({
        model:"gpt-4o-mini",
        messages:[{role:"user",content:text}]
      })
    });
    const data=await response.json();
    typingMsg.remove();
    const botText=data.choices[0].message.content;
    addMessage(botText,"bot");

    const summary=summarizeMessage(botText);
    addHistory(summary,text,botText);

    chatHistory.push({summary, user:text, bot:botText});
    localStorage.setItem("chatHistory", JSON.stringify(chatHistory));
  }catch(err){
    typingMsg.remove();
    addMessage("Bir hata oluştu ❌","bot");
  }
}

sendBtn.addEventListener("click",sendMessage);
userInput.addEventListener("keypress",(e)=>{
  if(e.key==="Enter") sendMessage();
});
</script>

</body>
</html>


