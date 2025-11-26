# the-FOX-Ai
the fox ai the advanced Ai for the friendly users
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Mini AI Chatbot</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root {
      --bg: #0f172a;
      --panel: #111827;
      --accent: #22c55e;
      --text: #e5e7eb;
      --muted: #94a3b8;
      --user: #2563eb;
      --bot: #334155;
    }
    * { box-sizing: border-box }
    body {
      margin: 0;
      background: linear-gradient(180deg, #0b1220, var(--bg));
      color: var(--text);
      font-family: system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, sans-serif;
      display: grid;
      place-items: center;
      min-height: 100vh;
      padding: 24px;
    }
    .app {
      width: 100%;
      max-width: 800px;
      background: rgba(17, 24, 39, 0.85);
      border: 1px solid #1f2937;
      border-radius: 16px;
      overflow: hidden;
      box-shadow: 0 20px 60px rgba(0,0,0,0.4);
    }
    header {
      display: flex;
      align-items: center;
      gap: 12px;
      padding: 16px 20px;
      background: #0b1220;
      border-bottom: 1px solid #1f2937;
    }
    .logo {
      width: 34px; height: 34px; border-radius: 8px;
      display: grid; place-items: center;
      background: linear-gradient(135deg, var(--accent), #16a34a);
      color: #0b1220; font-weight: 700;
    }
    header h1 { font-size: 16px; margin: 0 }
    header p { margin: 0; font-size: 12px; color: var(--muted) }

    .chat {
      height: 60vh;
      overflow-y: auto;
      padding: 20px;
      display: flex;
      flex-direction: column;
      gap: 12px;
    }
    .msg {
      max-width: 78%;
      padding: 12px 14px;
      border-radius: 14px;
      line-height: 1.45;
      font-size: 14px;
      word-wrap: break-word;
      position: relative;
    }
    .msg.user {
      align-self: flex-end;
      background: rgba(37, 99, 235, 0.22);
      border: 1px solid rgba(37, 99, 235, 0.35);
    }
    .msg.bot {
      align-self: flex-start;
      background: rgba(51, 65, 85, 0.5);
      border: 1px solid rgba(51, 65, 85, 0.6);
    }
    .meta {
      font-size: 11px;
      color: var(--muted);
      margin-top: 4px;
    }
    .typing {
      width: 54px; height: 28px;
      background: rgba(51, 65, 85, 0.5);
      border: 1px solid rgba(51, 65, 85, 0.6);
      border-radius: 16px;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      gap: 6px;
      padding: 0 8px;
    }
    .dot {
      width: 6px; height: 6px; border-radius: 50%;
      background: #cbd5e1; opacity: 0.6;
      animation: pulse 1.2s infinite;
    }
    .dot:nth-child(2) { animation-delay: 0.2s }
    .dot:nth-child(3) { animation-delay: 0.4s }
    @keyframes pulse {
      0% { transform: scale(0.8); opacity: 0.5 }
      50% { transform: scale(1); opacity: 1 }
      100% { transform: scale(0.8); opacity: 0.5 }
    }

    .composer {
      display: grid;
      grid-template-columns: 1fr auto;
      gap: 8px;
      padding: 14px;
      border-top: 1px solid #1f2937;
      background: #0b1220;
    }
    textarea {
      resize: none;
      height: 46px;
      padding: 12px;
      border-radius: 10px;
      border: 1px solid #1f2937;
      background: #0f172a;
      color: var(--text);
      outline: none;
    }
    textarea::placeholder { color: #64748b }
    button {
      height: 46px;
      padding: 0 16px;
      border: none;
      border-radius: 10px;
      background: var(--accent);
      color: #061016;
      font-weight: 700;
      cursor: pointer;
    }
    button:disabled { opacity: 0.6; cursor: not-allowed }
    .help {
      padding: 8px 16px 16px;
      font-size: 12px;
      color: var(--muted);
    }
    .help code {
      background: #0f172a;
      border: 1px solid #1f2937;
      padding: 2px 6px;
      border-radius: 6px;
    }
  </style>
</head>
<body>
  <div class="app">
    <header>
      <div class="logo">AI</div>
      <div>
        <h1>Mini AI Chatbot</h1>
        <p>Intent matching, memory, and a clean UI — no external APIs.</p>
      </div>
    </header>

    <main class="chat" id="chat"></main>

    <div class="composer">
      <textarea id="input" placeholder="Type a message... (e.g., 'solve 2x+3=9')"></textarea>
      <button id="send">Send</button>
    </div>
    <div class="help">
      Try: <code>hi</code> <code>my name is ...</code> <code>time in Bengaluru</code> <code>solve 2x+3=9</code> <code>tips for HTML/CSS/JS</code>
    </div>
  </div>

  <script>
    // ---------- Utilities ----------
    const stopWords = new Set("a an and are as at be by for from has he in is it its of on that the to was were will with you your".split(" "));
    const tokenize = (text) =>
      text.toLowerCase()
          .replace(/[^a-z0-9\s\+\-\*\/\=\.\,\:\?]/g, " ")
          .split(/\s+/)
          .filter(t => t && !stopWords.has(t));

    const bagOfWords = (tokens) => {
      const bag = {};
      for (const t of tokens) bag[t] = (bag[t] || 0) + 1;
      return bag;
    };

    const cosine = (a, b) => {
      const keys = new Set([...Object.keys(a), ...Object.keys(b)]);
      let dot = 0, na = 0, nb = 0;
      for (const k of keys) {
        const va = a[k] || 0, vb = b[k] || 0;
        dot += va * vb;
        na += va * va;
        nb += vb * vb;
      }
      return na && nb ? dot / (Math.sqrt(na) * Math.sqrt(nb)) : 0;
    };

    const nowIST = () => {
      const date = new Date();
      const utc = date.getTime() + (date.getTimezoneOffset() * 60000);
      const ist = new Date(utc + (5.5 * 3600000));
      return ist;
    };

    // ---------- Memory ----------
    const memory = {
      get name() { return localStorage.getItem("ai_name") || null; },
      set name(n) { localStorage.setItem("ai_name", n); },
      get lastTopic() { return localStorage.getItem("ai_last_topic") || null; },
      set lastTopic(t) { localStorage.setItem("ai_last_topic", t); },
    };

    // ---------- Intents ----------
    const intents = [
      {
        id: "greeting",
        keywords: ["hi","hello","hey","yo","sup","good","morning","evening"],
        reply: ({name}) => `Hey${name ? " " + name : ""}! What are you building today?`
      },
      {
        id: "name_set",
        keywords: ["my","name","is","call","me"],
        match: (text) => {
          const m = text.match(/(?:name\s+is|call\s+me)\s+([a-z][a-z0-9_]*)/i);
          return m ? m[1] : null;
        },
        reply: ({value}) => {
          memory.name = value.charAt(0).toUpperCase() + value.slice(1);
          return `Nice to meet you, ${memory.name}. I’ll remember that.`;
        }
      },
      {
        id: "time_bengaluru",
        keywords: ["time","current","now","bengaluru","india","ist"],
        reply: () => {
          const t = nowIST();
          const day = t.toLocaleDateString("en-IN", { weekday: "long" });
          const stamp = t.toLocaleString("en-IN", { hour: "2-digit", minute: "2-digit", second: "2-digit" });
          memory.lastTopic = "time";
          return `In Bengaluru (IST), it's ${stamp} on ${day}.`;
        }
      },
      {
        id: "math_solve_linear",
        keywords: ["solve","equation","x","y","linear","algebra"],
        match: (text) => text.match(/solve\s+([0-9x\+\-\*\/\s]+)=\s*([0-9\.\-]+)/i),
        reply: ({value}) => {
          // Example: solve 2x+3=9
          const lhs = value[1].replace(/\s+/g, "");
          const rhs = parseFloat(value[2]);
          // Extract ax + b
          const m = lhs.match(/^([\-]?\d*\.?\d*)x([\+\-]\d+\.?\d*)?$/i);
          if (!m) return "I can solve forms like '2x+3=9'. Try that pattern.";
          const a = parseFloat(m[1] || "1");
          const b = parseFloat(m[2] || "0");
          const x = (rhs - b) / a;
          memory.lastTopic = "math";
          return `Solving ${lhs} = ${rhs}: x = ${x}.`;
        }
      },
      {
        id: "web_tips",
        keywords: ["html","css","javascript","front","web","design","layout","grid","flex"],
        reply: () => {
          memory.lastTopic = "web";
          return [
            "Quick dev tips:",
            "- Use semantic HTML for structure (header, main, section, footer).",
            "- Start mobile-first CSS, then layer min-width media queries.",
            "- Prefer Flexbox for simple alignment; Grid for complex layouts.",
            "- Keep JS pure: separate state, render, and events.",
            "- Reuse utility classes and clamp() for fluid typography."
          ].join("\n");
        }
      },
      {
        id: "farewell",
        keywords: ["bye","goodbye","see","later","ttyl","exit","quit"],
        reply: ({name}) => `Catch you later${name ? ", " + name : ""}. Keep building!`
      },
      {
        id: "fallback",
        keywords: [],
        reply: ({name}) => {
          return `Hmm${name ? ", " + name : ""}… I didn’t get that. Try 'time in Bengaluru', 'solve 2x+3=9', or 'tips for HTML/CSS/JS'.`;
        }
      }
    ];

    // Precompute keyword bags
    for (const intent of intents) {
      intent._bag = bagOfWords(intent.keywords);
    }

    // ---------- UI helpers ----------
    const chat = document.getElementById("chat");
    const input = document.getElementById("input");
    const sendBtn = document.getElementById("send");

    const addMsg = (text, who = "bot") => {
      const el = document.createElement("div");
      el.className = `msg ${who}`;
      el.textContent = text;
      chat.appendChild(el);
      chat.scrollTop = chat.scrollHeight;
    };

    const addTyping = () => {
      const wrap = document.createElement("div");
      wrap.className = "msg bot";
      wrap.style.background = "transparent";
      wrap.style.border = "none";
      const box = document.createElement("div");
      box.className = "typing";
      box.innerHTML = '<div class="dot"></div><div class="dot"></div><div class="dot"></div>';
      wrap.appendChild(box);
      chat.appendChild(wrap);
      chat.scrollTop = chat.scrollHeight;
      return wrap;
    };

    const removeTyping = (el) => el && el.remove();

    // ---------- Core: process message ----------
    const process = (text) => {
      const tokens = tokenize(text);
      const bag = bagOfWords(tokens);

      // Handle special pattern intents with custom match()
      for (const intent of intents) {
        if (intent.match) {
          const m = intent.match(text);
          if (m) {
            return intent.reply({ value: m, name: memory.name });
          }
        }
      }

      // Otherwise use cosine similarity over keywords
      let best = { intent: null, score: 0 };
      for (const intent of intents) {
        const score = cosine(bag, intent._bag);
        if (score > best.score) best = { intent, score };
      }
      if (!best.intent || best.score < 0.12) {
        return intents.find(i => i.id === "fallback").reply({ name: memory.name });
      }
      return best.intent.reply({ name: memory.name });
    };

    // ---------- Boot ----------
    const greet = () => {
      addMsg("Hi! I’m your mini AI. Ask me about time in Bengaluru, web dev tips, or simple equations.");
      if (memory.name) addMsg(`Welcome back, ${memory.name}.`);
    };
    greet();

    // ---------- Events ----------
    const send = () => {
      const text = input.value.trim();
      if (!text) return;
      addMsg(text, "user");
      input.value = "";
      sendBtn.disabled = true;
      const typing = addTyping();

      // Simulated thinking delay
      setTimeout(() => {
        const reply = process(text);
        removeTyping(typing);
        addMsg(reply, "bot");
        sendBtn.disabled = false;
      }, 600 + Math.random() * 400);
    };

    sendBtn.addEventListener("click", send);
    input.addEventListener("keydown", (e) => {
      if (e.key === "Enter" && !e.shiftKey) {
        e.preventDefault();
        send();
      }
    });
  </script>
</body>
</html>
