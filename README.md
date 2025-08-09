# Spike-IA

Automação completa da Spike IA

1. Backend (Node.js + Express + Firebase Functions)

Funcionalidades backend:

Recebe pedidos de criação de vídeo com tema e parâmetros

Solicita roteiro via OpenAI GPT

Gera voz via TTS (Google Cloud TTS)

Gera thumbnail via DALL·E

Monta vídeo com ffmpeg (inserindo legenda, áudio e imagens)

Salva resultados no storage (Firebase Storage)

Notifica frontend via WebSocket ou polling status do job

Faz publicação automática em redes sociais (API YouTube, Instagram etc)



---

Código inicial backend (Node.js + Express + Firebase Functions)

// backend/index.js
const functions = require("firebase-functions");
const admin = require("firebase-admin");
const express = require("express");
const cors = require("cors");
const { Configuration, OpenAIApi } = require("openai");
const textToSpeech = require("@google-cloud/text-to-speech");
const { spawn } = require("child_process");
const fs = require("fs");
const path = require("path");

admin.initializeApp();
const db = admin.firestore();
const storage = admin.storage().bucket();

const app = express();
app.use(cors({ origin: true }));
app.use(express.json());

// Configuração OpenAI
const openai = new OpenAIApi(new Configuration({
  apiKey: process.env.OPENAI_API_KEY,
}));

// Configuração Google TTS
const clientTTS = new textToSpeech.TextToSpeechClient();

app.post("/create-video", async (req, res) => {
  try {
    const { topic, style, length } = req.body;

    // 1. Geração do roteiro
    const prompt = `Crie um roteiro educativo, estilo ${style}, sobre o tema: ${topic}, com duração de ${length} minutos.`;
    const completion = await openai.createChatCompletion({
      model: "gpt-4o-mini",
      messages: [{ role: "user", content: prompt }],
    });
    const script = completion.data.choices[0].message.content;

    // 2. Geração da voz (TTS)
    const requestTTS = {
      input: { text: script },
      voice: { languageCode: "pt-BR", ssmlGender: "FEMALE" },
      audioConfig: { audioEncoding: "MP3" },
    };
    const [responseTTS] = await clientTTS.synthesizeSpeech(requestTTS);
    const audioBuffer = responseTTS.audioContent;

    const audioPath = path.join("/tmp", `audio-${Date.now()}.mp3`);
    fs.writeFileSync(audioPath, audioBuffer);

    // TODO: 3. Gerar thumbnail via DALL·E (API chamada separada)
    // TODO: 4. Montar vídeo com ffmpeg (inserir áudio, texto e thumbnail)

    // Placeholder resposta
    res.json({
      status: "process started",
      script,
      audioFile: audioPath,
    });

  } catch (error) {
    console.error(error);
    res.status(500).json({ error: error.message });
  }
});

exports.api = functions.https.onRequest(app);


---

2. Frontend (React)

Funcionalidades:

Formulário para enviar tema, estilo e duração

Mostra status da geração do vídeo (loading, error, complete)

Visualiza roteiro gerado e player do áudio

Lista vídeos gerados



---

Código inicial frontend (React + Vite)

// frontend/src/App.jsx
import { useState } from "react";

export default function App() {
  const [topic, setTopic] = useState("");
  const [style, setStyle] = useState("educativo");
  const [length, setLength] = useState(5);
  const [status, setStatus] = useState("");
  const [script, setScript] = useState("");
  const [audioUrl, setAudioUrl] = useState("");

  const handleSubmit = async () => {
    setStatus("Gerando roteiro...");
    const res = await fetch("https://[seu-firebase-function-url]/create-video", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ topic, style, length }),
    });
    const data = await res.json();
    if (res.ok) {
      setScript(data.script);
      setAudioUrl(URL.createObjectURL(new Blob([data.audioFile])));
      setStatus("Vídeo em processamento...");
    } else {
      setStatus("Erro: " + data.error);
    }
  };

  return (
    <div>
      <h1>Spike IA - Gerador de Vídeos Educativos</h1>
      <input
        placeholder="Tema do vídeo"
        value={topic}
        onChange={e => setTopic(e.target.value)}
      />
      <select value={style} onChange={e => setStyle(e.target.value)}>
        <option value="educativo">Educativo</option>
        <option value="informativo">Informativo</option>
        <option value="divertido">Divertido</option>
      </select>
      <input
        type="number"
        min={1}
        max={15}
        value={length}
        onChange={e => setLength(Number(e.target.value))}
      />
      <button onClick={handleSubmit}>Gerar Vídeo</button>

      <p>Status: {status}</p>
      {script && (
        <div>
          <h3>Roteiro Gerado</h3>
          <pre>{script}</pre>
          {audioUrl && <audio controls src={audioUrl} />}
        </div>
      )}
    </div>
  );
}


---

3. Próximos passos para a automação completa

Implementar geração da thumbnail via DALL·E com prompt baseado no tema

Criar função para montagem automática de vídeo usando ffmpeg (inserir legenda, áudio e imagem)

Criar sistema de fila para orquestrar etapas em segundo plano (Cloud Tasks ou RabbitMQ)

Integrar APIs das redes sociais para upload e publicação automática

Desenvolver painel para monitorar status dos jobs e histórico

Criar app Android para controle e geração móvel

// backend/index.js
const functions = require("firebase-functions");
const admin = require("firebase-admin");
const express = require("express");
const cors = require("cors");
const { Configuration, OpenAIApi } = require("openai");
const textToSpeech = require("@google-cloud/text-to-speech");
const { spawn } = require("child_process");
const fs = require("fs");
const path = require("path");

admin.initializeApp();
const db = admin.firestore();
const storage = admin.storage().bucket();

const app = express();
app.use(cors({ origin: true }));
app.use(express.json());

// Configuração OpenAI
const openai = new OpenAIApi(new Configuration({
  apiKey: process.env.OPENAI_API_KEY,
}));

// Configuração Google TTS
const clientTTS = new textToSpeech.TextToSpeechClient();

app.post("/create-video", async (req, res) => {
  try {
    const { topic, style, length } = req.body;

    // 1. Geração do roteiro
    const prompt = `Crie um roteiro educativo, estilo ${style}, sobre o tema: ${topic}, com duração de ${length} minutos.`;
    const completion = await openai.createChatCompletion({
      model: "gpt-4o-mini",
      messages: [{ role: "user", content: prompt }],
    });
    const script = completion.data.choices[0].message.content;

    // 2. Geração da voz (TTS)
    const requestTTS = {
      input: { text: script },
      voice: { languageCode: "pt-BR", ssmlGender: "FEMALE" },
      audioConfig: { audioEncoding: "MP3" },
    };
    const [responseTTS] = await clientTTS.synthesizeSpeech(requestTTS);
    const audioBuffer = responseTTS.audioContent;

    const audioPath = path.join("/tmp", `audio-${Date.now()}.mp3`);
    fs.writeFileSync(audioPath, audioBuffer);

    // TODO: 3. Gerar thumbnail via DALL·E (API chamada separada)
    // TODO: 4. Montar vídeo com ffmpeg (inserir áudio, texto e thumbnail)

    // Placeholder resposta
    res.json({
      status: "process started",
      script,
      audioFile: audioPath,
    });

  } catch (error) {
    console.error(error);
    res.status(500).json({ error: error.message });
  }
});

exports.api = functions.https.onRequest(app);// frontend/src/App.jsx
import { useState } from "react";

export default function App() {
  const [topic, setTopic] = useState("");
  const [style, setStyle] = useState("educativo");
  const [length, setLength] = useState(5);
  const [status, setStatus] = useState("");
  const [script, setScript] = useState("");
  const [audioUrl, setAudioUrl] = useState("");

  const handleSubmit = async () => {
    setStatus("Gerando roteiro...");
    const res = await fetch("https://[seu-firebase-function-url]/create-video", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ topic, style, length }),
    });
    const data = await res.json();
    if (res.ok) {
      setScript(data.script);
      setAudioUrl(URL.createObjectURL(new Blob([data.audioFile])));
      setStatus("Vídeo em processamento...");
    } else {
      setStatus("Erro: " + data.error);
    }
  };

  return (
    <div>
      <h1>Spike IA - Gerador de Vídeos Educativos</h1>
      <input
        placeholder="Tema do vídeo"
        value={topic}
        onChange={e => setTopic(e.target.value)}
      />
      <select value={style} onChange={e => setStyle(e.target.value)}>
        <option value="educativo">Educativo</option>
        <option value="informativo">Informativo</option>
        <option value="divertido">Divertido</option>
      </select>
      <input
        type="number"
        min={1}
        max={15}
        value={length}
        onChange={e => setLength(Number(e.target.value))}
      />
      <button onClick={handleSubmit}>Gerar Vídeo</button>

      <p>Status: {status}</p>
      {script && (
        <div>
          <h3>Roteiro Gerado</h3>
          <pre>{script}</pre>
          {audioUrl && <audio controls src={audioUrl} />}
        </div>
      )}
    </div>
  );
}// backend/index.js
const functions = require("firebase-functions");
const admin = require("firebase-admin");
const express = require("express");
const cors = require("cors");
const { Configuration, OpenAIApi } = require("openai");
const textToSpeech = require("@google-cloud/text-to-speech");
const fs = require("fs");
const path = require("path");

admin.initializeApp();
const app = express();
app.use(cors({ origin: true }));
app.use(express.json());

const openai = new OpenAIApi(new Configuration({ apiKey: process.env.OPENAI_API_KEY }));
const ttsClient = new textToSpeech.TextToSpeechClient();

app.post("/create-video", async (req, res) => {
  try {
    const { topic, style, length } = req.body;

    const prompt = `Crie um roteiro educativo estilo ${style} sobre o tema ${topic} com duração de ${length} minutos.`;
    const completion = await openai.createChatCompletion({
      model: "gpt-4o-mini",
      messages: [{ role: "user", content: prompt }],
    });
    const script = completion.data.choices[0].message.content;

    const ttsRequest = {
      input: { text: script },
      voice: { languageCode: "pt-BR", ssmlGender: "FEMALE" },
      audioConfig: { audioEncoding: "MP3" },
    };
    const [ttsResponse] = await ttsClient.synthesizeSpeech(ttsRequest);
    const audioBuffer = ttsResponse.audioContent;
    const audioPath = path.join("/tmp", `audio-${Date.now()}.mp3`);
    fs.writeFileSync(audioPath, audioBuffer);

    res.json({ script, audioFile: audioPath });
  } catch (e) {
    res.status(500).json({ error: e.message });
  }
});

exports.api = functions.https.onRequest(app);name: Spike IA CI/CD

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout do código
      uses: actions/checkout@v3
    - name: Configurar Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
    - name: Instalar dependências
      run: npm install
    - name: Rodar build
      run: npm run build# 👾 Spike IA

Bem-vindo ao perfil oficial da **Spike IA** — a inteligência artificial que cria automaticamente vídeos educativos, com roteiro, voz, legenda e thumbnail.

## 🚀 Recursos
- Geração de roteiros automáticos
- Voz sintética natural
- Edição de vídeo com legendas
- Criação de thumbnails
- Upload automático para redes sociais
- Publicação de APK Android na Play Store

## 📂 Projetos Atuais
- **Spike IA Core** → Backend + IA + Automação
- **Spike IA Web** → Painel de controle online
- **Spike IA Mobile** → Aplicativo Android

## 📫 Contato
- E-mail: contato@spikeia.com
- Site: [www.spikeia.com](https://www.spikeia.com)

---
✨ *Transformando ideias em vídeos, automaticamente.*

