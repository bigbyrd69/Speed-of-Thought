# Kept

Saved posts pile up on Instagram, LinkedIn, and X and never get used. Kept turns one saved link into a file, then brings that file back when you are actually doing the thing.

## What each tool does

- **Wispr Flow** is how you say why you kept it, and later what you are trying to do. Click the box and hold the Flow key. A Wispr API key is optional and only powers `/api/transcribe`.
- **Anakin** scrapes the public URL into that file: the page text and the links mentioned in it, including repositories. Your spoken reason is not sent to Anakin.
- **GenieX** reads the library locally and answers which file fits. The private “why” stays on this machine.

Each post is its own pair of files under `library/<source>/`, split by Instagram, LinkedIn, X, GitHub, YouTube, or web.

## Run

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

Put your Anakin key in `.env`, then:

```bash
uvicorn app:app --host 127.0.0.1 --port 8791
```

Open http://127.0.0.1:8791

`.env` is gitignored. Do not commit it.

GenieX serves the local model. On a machine without a Snapdragon NPU, Qualcomm’s CPU image works:

```bash
docker run -d --name kept-geniex -p 18181:18181 \
  -v "$PWD/.geniex:/data" -e GENIEX_DATADIR=/data \
  docker.io/qualcomm/geniex:latest-cpu \
  geniex serve --host 0.0.0.0:18181 --compute cpu --ngl 0 --nctx 8192 \
  --model "Qwen/Qwen3-4B-GGUF"
```

Set the chipset once before the first model pull: `geniex config set chipset cpu`, then `geniex pull Qwen/Qwen3-4B-GGUF:Q4_K_M --model-hub hf`.

## Files

- `app.py` — keep a URL, scrape it, write the file, and answer from the library
- `static/index.html` — the page you dictate into
- `library/<source>/` — one markdown file and one JSON card per saved post
- `.env.example` — the variables the app reads

## Demo

1. Paste a public link. Hold Flow and say why you are keeping it. Press Keep it.
2. Do that for two or three posts.
3. Hold Flow and say what you are building, applying for, or writing. Press What did I keep?
4. Open the file it names. The link inside the post is listed under the suggestion.
