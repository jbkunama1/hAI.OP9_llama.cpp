# 🤖 Llama.cpp als LAN‑Server auf dem OnePlus 9 (Android + Termux)

Dieses Projekt zeigt, wie du **Llama.cpp** direkt auf einem **OnePlus 9** mit **Android + Termux** als **lokalen LLM‑Server im LAN** betreibst – **ohne Ollama** und ohne Cloud.

Getestet mit:

- Gerät: OnePlus 9 (Snapdragon 888, Adreno 660)
- OS: Android (Termux aus F‑Droid)
- LLM: `Llama-3.2-1B-Instruct-Q4_K_M.gguf` (GGUF, Hugging Face)[web:187][web:189]
- Server: `llama-server` (OpenAI‑kompatible API)

---

## ✨ Features

- ✅ Llama.cpp **nativ** auf dem Smartphone
- ✅ **OpenAI‑kompatible API** (`/v1/chat/completions`)
- ✅ **LAN‑Zugriff** für PCs, Laptops, ESP32, andere Phones
- ✅ **Start/Stop‑Skripte** (inkl. GPU‑Offload)
- ✅ Läuft im **Hintergrund** mit `nohup` + `termux-wake-lock`[web:31][web:33][web:209]

---

## ⚙️ Voraussetzungen

- OnePlus 9 mit Android
- **Termux** (empfohlen: F‑Droid‑Version)
- WLAN/LAN, in dem andere Geräte das Handy erreichen können
- Internetzugang, um das Modell einmalig von Hugging Face zu laden

---

## 📥 1. Termux vorbereiten

In Termux:

```bash
pkg update && pkg upgrade -y
pkg install git cmake clang make python lld termux-api -y
```

Storage (optional, aber praktisch):

```bash
termux-setup-storage
ls ~/storage/
# sollte z.B. "downloads", "shared", ... anzeigen
```

[web:106][web:132]

---

## 📦 2. Llama.cpp klonen & bauen

### 2.1 Repository klonen

```bash
cd ~
rm -rf ~/llama.cpp
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
```

*(Optional kannst du auf einen bekannten stabilen Commit pinnen.)*[web:171]

### 2.2 Mit CMake bauen (CPU + Server)

```bash
cmake -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DLLAMA_BUILD_TESTS=OFF \
  -DLLAMA_BUILD_EXAMPLES=OFF \
  -DLLAMA_BUILD_SERVER=ON

cmake --build build --config Release -j4
```

### 2.3 Build prüfen

```bash
cd ~/llama.cpp/build/bin
ls -la
```

Erwartete Binaries u. a.:

```text
llama-cli
llama-server
llama-quantize
libllama.so
...
```

Kurztest:

```bash
./llama-cli --help
./llama-server --help
```

[web:31][web:33]

---

## 🧠 3. Modell: Llama‑3.2‑1B‑Instruct‑Q4_K_M

Wir nutzen Hugging Face direkt im `llama-server`:

- Repo: `bartowski/Llama-3.2-1B-Instruct-GGUF`[web:189]
- Datei: `Llama-3.2-1B-Instruct-Q4_K_M.gguf`[web:187][web:195]

Vorteil: **kein manueller Download nötig** – der Server lädt das Modell beim ersten Start automatisch und cached es lokal.

---

## 🚀 4. Llama‑Server starten (Testlauf)

In Termux:

```bash
cd ~/llama.cpp/build/bin

./llama-server \
  --hf-repo bartowski/Llama-3.2-1B-Instruct-GGUF \
  --hf-file Llama-3.2-1B-Instruct-Q4_K_M.gguf \
  --host 0.0.0.0 \
  --port 8080 \
  -c 4096 \
  --ctx-size 4096 \
  -ngl 0
```

- `--hf-repo` / `--hf-file`: automatisch von Hugging Face laden[web:187][web:189]
- `--host 0.0.0.0`: macht den Server im LAN erreichbar[web:152][web:157]
- `-c 4096` / `--ctx-size 4096`: Kontextfenster 4k Tokens
- `-ngl 0`: CPU‑only (GPU‑Offload später)

Beim **ersten Start** wird die GGUF‑Datei (~0,8 GB) heruntergeladen, danach kommt direkt die Server‑Ausgabe.

---

## 🌐 5. Zugriff im LAN

### 5.1 IP des OnePlus 9 herausfinden

In Termux:

```bash
ip addr show wlan0 | grep inet
```

Beispiel:

```text
inet 192.168.178.18/24 brd 192.168.178.255 scope global wlan0
```

→ IP im LAN: `192.168.178.18`.

### 5.2 Browser-Zugriff

Auf einem anderen Gerät im selben WLAN:

- Browser öffnen:  
  `http://192.168.178.18:8080`

Du solltest eine einfache Web‑UI von `llama-server` sehen.[web:31][web:159]

### 5.3 OpenAI‑API per `curl`

```bash
curl -X POST http://192.168.178.18:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama",
    "max_tokens": 256,
    "messages": [
      {"role": "user", "content": "Erkläre kurz den Unterschied zwischen RAM und ROM."}
    ]
  }'
```


<a href="https://buymeacoffee.com/highfish"><img src="https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png"></a>
