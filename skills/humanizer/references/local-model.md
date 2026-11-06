# Local model setup (optional)

A 12B rewriting model run locally through llama.cpp, called with `hz`, which keeps
headings, code, tables and links and rewrites prose in pieces. Everything lives in
`~/.cache/humanizer`, never in this skill folder.

Tested on Apple Silicon macOS (M2 Pro, 32 GB). Needs about 14 GB RAM and 8 GB disk.
On other platforms, take llama.cpp b11800 for that platform from its release page, and check the asset against its `digest` field from the GitHub release API.

| Item | Pin |
|---|---|
| Model | `jialinyyzz/humanizer` @ `2e57b6528a4070c33f3e09ca24e3325a84272f68`, `humanizer-12b-Q4_K_M.gguf`, SHA-256 `2229574dec5178629575ee4a153dfee7d9e924ab997d0ad9d2ac622b67e44834` (Apache-2.0) |
| Runtime | llama.cpp `b11800` macOS arm64, SHA-256 `6dd79d687754c2e6dd93fe3b921d26b05cff23e23283b7bcd6cbb79ba52e0d79` |
| hz | `sgaofen/humanizer-local-model` @ `7dd4be90cb3acb2e5d18aeea5837f888d2bb2d52` (Apache-2.0) |

## Install

Run these only at the user's request. They download third-party code and an 8 GB model.

```sh
set -eu
H=~/.cache/humanizer; mkdir -p "$H" && cd "$H"
curl -fL -C - -o humanizer-12b-Q4_K_M.gguf \
  https://huggingface.co/jialinyyzz/humanizer/resolve/2e57b6528a4070c33f3e09ca24e3325a84272f68/humanizer-12b-Q4_K_M.gguf
curl -fL -o llama-b11800.tar.gz \
  https://github.com/ggml-org/llama.cpp/releases/download/b11800/llama-b11800-bin-macos-arm64.tar.gz
shasum -a 256 -c - <<'EOF'
2229574dec5178629575ee4a153dfee7d9e924ab997d0ad9d2ac622b67e44834  humanizer-12b-Q4_K_M.gguf
6dd79d687754c2e6dd93fe3b921d26b05cff23e23283b7bcd6cbb79ba52e0d79  llama-b11800.tar.gz
EOF
tar -xzf llama-b11800.tar.gz
python3 -m venv venv
venv/bin/pip install "git+https://github.com/sgaofen/humanizer-local-model@7dd4be90cb3acb2e5d18aeea5837f888d2bb2d52"
mkdir -p ~/.local/bin && ln -sf "$H/venv/bin/hz" ~/.local/bin/hz   # ~/.local/bin must be on PATH
hz --version   # hz 0.1.0
```

## Model pass

1. **Model pass, required when installed.** Create `T=$(mktemp -d)` and write the
   pasted text to `$T/input.md`. For a file input, `hz` may read the file directly, but
   `report.json` still goes in `$T`.
   If the server is not running, start the command in Run below in the background, wait
   for its health check. Then run
   `hz "$T/input.md" -o "$T/output.md" --server http://127.0.0.1:18423 --json --quiet > "$T/report.json"`.
   If the model is installed but the pass fails, stop and report the error instead of
   skipping it.
2. **Check every model piece against its source.** Show `hz`'s flagged pieces. `hz` flags
   missing or added numbers, missing links, copying, length, truncation, repeats, stray
   markup and language switches. It does not catch changed words, names or meaning, so
   also read for changed meaning, inverted
   claims, lost or added hedges, wrong attribution (for example "our article" for
   someone else's), changed link text, changed inline code, commands or paths (`hz`
   keeps code blocks, not inline code), broken grammar, and any word or
   sentence not in the source's language (`hz` resamples only whole-piece switches
   between English and Chinese).
3. **Resample faulty pieces, never hand-edit them.** Hand edits and restored original
   sentences undo the model's effect. Put the faulty piece's source text in its own file,
   rerun `hz` on it up to 3 times, and keep the first accurate result. If none is
   accurate, show the piece and let the user decide.
4. **Remove dashes, then report remaining tells.** Replace every em dash, en dash and
   ` -- ` used as a dash in prose with a comma, colon, period or parentheses; code and
   URLs are exempt. This is the only edit to model wording. Report the other tells from the
   tell lists in `SKILL.md` without rewriting.
5. **Clean up.** After the final text is delivered, or after a failure, stop the server if you
   started it and delete `$T`.

## Run

```sh
~/.cache/humanizer/llama-b11800/llama-server -m ~/.cache/humanizer/humanizer-12b-Q4_K_M.gguf \
  -c 8192 -np 1 -ngl 99 --host 127.0.0.1 --port 18423 --cors-origins http://127.0.0.1:18423
```

`--cors-origins` stops other sites' pages from reading the server's responses. It does not
block requests, and the server has no authentication, so stop it in step 5 of the model pass if you started it. It is ready when
`curl -s http://127.0.0.1:18423/health` returns `{"status":"ok"}`.

Self-test: `printf 'We completed 88 of 100 files.\n' | hz --server http://127.0.0.1:18423`
should print a reworded sentence that keeps 88 and 100.

Always pass `--server`. Without it, `hz` prefers the Humanizer desktop app, which can be
configured to send text to an external service.

## Limits

- Output still contains AI tells (contrasts, sales words, dashes). The skill removes
  dashes and reports the rest.
- It can change a word, a number format or a claim's direction. `hz` does not catch
  changed words or meaning; read the rest.
- Generation is slow under memory pressure (about 7 tokens/s observed with heavy swap).
