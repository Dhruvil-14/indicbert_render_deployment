# IndicBERT Render API — model stored on Hugging Face

GitHub stores only this small API source. The large ONNX model and tokenizer live in your Hugging Face model repository and are downloaded to `/tmp/indicbert_model` at service startup.

## GitHub repository contents
- `app.py`
- `requirements.txt`
- `render.yaml`
- `runtime.txt`

Do NOT commit the ONNX model or tokenizer files to GitHub.

## Hugging Face repository contents
Upload these six files to the root of a Hugging Face **model** repository:
- `model_int8.onnx`
- `spiece.model`
- `tokenizer_config.json`
- `special_tokens_map.json`
- `label_mapping.json`
- `config.json`

Use the companion Colab notebook to export, quantize, and upload them.

## Render environment variables
- `HF_MODEL_REPO` = `your-hf-username/your-model-repo`
- `HF_REVISION` = `main` (optional)
- `HF_TOKEN` = a Hugging Face read token, required only if the model repo is private
- `MODEL_DIR` = `/tmp/indicbert_model` (optional)
- `MAX_LENGTH` = `160` (optional)

Keep the HF_TOKEN secret in Render's Environment settings. Never commit it to GitHub.

## Deploy
1. Push these source files to a GitHub repository.
2. Create a Render Web Service from that repository.
3. Build command: `pip install --no-cache-dir -r requirements.txt`
4. Start command: `uvicorn app:app --host 0.0.0.0 --port $PORT --workers 1`
5. Health check path: `/health`
6. Add the environment variables above.
7. Deploy and inspect the logs while it downloads the model.

## API
- `GET /`
- `GET /health`
- `POST /predict` with JSON: `{"text": "your message"}`
- `GET /docs`

## Important Render Free limitations
The local model download is stored on Render's ephemeral filesystem. Free services can lose local files when they spin down or restart, so the model may be downloaded again on the next start. The Hugging Face Hub is the persistent artifact store.

This design avoids putting large weights in GitHub, but it does NOT guarantee the model fits in 512 MB RAM. Check Render memory metrics and the `/health` peak RSS value after model loading and inference. If it exceeds the free limit, a larger instance or smaller model is needed.
