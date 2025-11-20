# UN-AI
// ==========================
// Multi-LLM Orchestrator (Single File)
// ==========================

require("dotenv").config();
const express = require("express");
const fetch = require("node-fetch");
const app = express();
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// ----------------------------------------
// PROVIDERS
// ----------------------------------------

async function openaiProvider(input) {
  const response = await fetch("https://api.openai.com/v1/chat/completions", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.OPENAI_API_KEY}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      model: "gpt-5.1",
      messages: [{ role: "user", content: input }]
    })
  });
  const json = await response.json();
  return json.choices?.[0]?.message?.content || "OpenAI error";
}

async function geminiProvider(input) {
  const response = await fetch(
    `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${process.env.GOOGLE_GEMINI_KEY}`,
    {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ contents: [{ parts: [{ text: input }] }] })
    }
  );
  const json = await response.json();
  return json?.candidates?.[0]?.content?.parts?.[0]?.text || "Gemini error";
}

async function grokProvider(input) {
  const response = await fetch("https://api.x.ai/v1/chat/completions", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.XAI_API_KEY}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      model: "grok-beta",
      messages: [{ role: "user", content: input }]
    })
  });
  const json = await response.json();
  return json.choices?.[0]?.message?.content || "Grok error";
}

async function perplexityProvider(input) {
  const response = await fetch("https://api.perplexity.ai/chat/completions", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.PERPLEXITY_API_KEY}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      model: "sonar-pro",
      messages: [{ role: "user", content: input }]
    })
  });
  const json = await response.json();
  return json.choices?.[0]?.message?.content || "Perplexity error";
}

async function metaProvider(input) {
  const response = await fetch("https://api.meta.ai/v1/chat/completions", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.META_LLAMA_KEY}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      model: "llama-3.1-70b",
      messages: [{ role: "user", content: input }]
    })
  });
  const json = await response.json();
  return json.choices?.[0]?.message?.content || "Meta error";
}

async function copilotProvider(input) {
  return "Copilot API not available — using stub response: " + input;
}

// Provider map
const PROVIDERS = {
  openai: openaiProvider,
  gemini: geminiProvider,
  grok: grokProvider,
  perplexity: perplexityProvider,
  meta: metaProvider,
  copilot: copilotProvider
};

// ----------------------------------------
// API ROUTE
// ----------------------------------------

app.post("/api/chat", async (req, res) => {
  try {
    const { provider, input } = req.body;

    if (!provider || !input)
      return res.status(400).json({ error: "Missing provider or input." });

    const selected = PROVIDERS[provider];
    if (!selected)
      return res.status(400).json({ error: "Invalid provider." });

    const output = await selected(input);
    res.json({ provider, response: output });
  } catch (err) {
    res.status(500).json({ error: "Server error", details: err });
  }
});

// ----------------------------------------
// FRONTEND (HTML + JS)
// ----------------------------------------

app.get("/", (req, res) => {
  res.send(`
<html>
<head>
<title>Multi LLM Orchestrator</title>
<style>
body { font-family: Arial; background: #111; color: #eee; padding: 20px; }
.card { background:#222; padding:20px; border-radius:10px; margin-top:20px; }
input, select, button { padding:10px; border-radius:5px; margin:5px; }
textarea { width:100%; height:150px; padding:10px; border-radius:5px; }
</style>
</head>
<body>

<h1>⚡ Multi-LLM Orchestrator</h1>

<select id="provider">
  <option value="openai">OpenAI GPT-5</option>
  <option value="gemini">Google Gemini 3</option>
  <option value="grok">xAI Grok</option>
  <option value="perplexity">Perplexity</option>
  <option value="meta">Meta Llama</option>
  <option value="copilot">GitHub Copilot</option>
</select>

<br>

<input id="input" placeholder="Ask anything..." style="width:80%;">

<button onclick="sendMsg()">Send</button>

<div class="card">
<h3>Response:</h3>
<pre id="response"></pre>
</div>

<script>
async function sendMsg() {
  const provider = document.getElementById("provider").value;
  const input = document.getElementById("input").value;

  const res = await fetch("/api/chat", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ provider, input })
  });

  const data = await res.json();
  document.getElementById("response").innerText =
    JSON.stringify(data, null, 2);
}
</script>

</body>
</html>
  `);
});

// ----------------------------------------
// START SERVER
// ----------------------------------------

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log("Server running on port " + PORT));


