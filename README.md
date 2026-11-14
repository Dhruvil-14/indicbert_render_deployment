# IndicBERT Render API — Hugging Face model storage

## Important Python version
Render uses the root-level `.python-version` file to select **Python 3.11.11**.
Do not rely on `runtime.txt`; Render's current native Python runtime uses `PYTHON_VERSION` or `.python-version`.

You can also set `PYTHON_VERSION=3.11.11` in Render Dashboard → Environment. This environment variable takes precedence over `.python-version`.

## GitHub repository contents
- `.python-version` (must be at repository root; contains `3.11.11`)
- `app.py`
- `requirements.txt`
- `render.yaml`
- `.gitignore`
- `README.md`

Do NOT commit the model weights to GitHub.

## Hugging Face model repository
The Hugging Face repository must contain:
- `model_int8.onnx`
- `spiece.model`
- `tokenizer_config.json`
- `special_tokens_map.json`
- `label_mapping.json`
- `config.json`

Use the companion Colab exporter/uploader notebook to create and upload these files.

## Render Environment Variables
- `PYTHON_VERSION` = `3.11.11` (recommended to set in the dashboard as well)
- `HF_MODEL_REPO` = `your-hf-username/your-model-repo`
- `HF_REVISION` = `main`
- `HF_TOKEN` = Hugging Face Read token (required only for a private model repo)
- `MODEL_DIR` = `/tmp/indicbert_model`
- `MAX_LENGTH` = `160`

Keep `HF_TOKEN` secret. Never commit it to GitHub.

## Render commands
Build:
`pip install --no-cache-dir -r requirements.txt`

Start:
`uvicorn app:app --host 0.0.0.0 --port $PORT --workers 1`

Health check:
`/health`

## API routes
- `GET /`
- `GET /health`
- `POST /predict`
- `GET /docs`

POST request body:
`{"text": "તમારો સંદેશ અહીં લખો"}`

## Memory note
The service uses ONNX Runtime CPU inference, one Uvicorn worker, one ORT intra-op thread and serialized inference requests. This setup reduces avoidable concurrency overhead but does not guarantee that a 512 MB Render instance will fit the model. Verify actual Render memory after model startup and inference.

Model files are downloaded from Hugging Face into Render's ephemeral `/tmp` storage at startup. Restarts/spin-downs can require re-downloading the files.
