## Quick orientation

- This is a small Flask-based chatbot that calls Mistral AI. Key files:
  - `app.py` — main Flask app and the LLM integration. See `get_chatbot_response()` for the prompt, model selection, and JSON parsing.
  - `templates/index.html` — single-page frontend that posts to `/chat` and expects a structured response.
  - `requirements.txt` — dependencies (Flask, mistralai, python-dotenv).
  - `.env` (not checked in) — contains `MISTRAL_API_KEY` used by `app.py`.

## Big-picture architecture

- Single-process Flask server that accepts POST /chat and proxies user messages to Mistral via `mistralai.client.MistralClient`.
- The LLM is asked to return a JSON object with two keys: `sentiment` and `response`. The frontend expects the returned string to include the sentiment first and the chatbot reply second.
- Frontend parsing (in `index.html`) splits the returned text on a double newline and looks for lines starting with `**Sentiment Analysis:**` and `**Chatbot Response:**`. Keep that in mind when changing formats.

## Important patterns & conventions (do not change without updating both sides)

- Response contract: The model is instructed to return a JSON object. Example expected model output (string):

  {"sentiment": "Positive", "response": "That sounds great! Here's how to proceed..."}

- `app.py:get_chatbot_response()` uses `response_format={"type": "json_object"}` when calling `client.chat(...)`, then attempts to `json.loads()` the returned content. If parsing fails, the code returns a fallback that includes the raw model output. If you change the response format, update both the parsing logic in `app.py` and the display parsing in `templates/index.html`.

- `MODEL_NAME` is defined in `app.py` (default `mistral-tiny`). Change it there to switch models.

## Running & developer workflow

- Install deps: `pip install -r requirements.txt`.
- Activate venv on Windows: `venv\Scripts\activate` then `pip install -r requirements.txt`.
- Start dev server: `python app.py` — server binds to `0.0.0.0:5000` with Flask debug enabled by default.
- Frontend: open `http://localhost:5000`.

## Integration & secrets

- The app reads `MISTRAL_API_KEY` from environment (via `python-dotenv`). Do not commit `.env` to the repo. If the key is missing the app prints a warning and `get_chatbot_response()` returns an error message.

## Typical edits and where to make them

- To change personality/prompt: edit `system_prompt` in `get_chatbot_response()` inside `app.py`.
- To change response serialization or structure: edit `response_format` in the `client.chat(...)` call and update the `json.loads` parsing and the frontend parsing in `templates/index.html` accordingly.
- To customize the UI: edit `templates/index.html` — the chat box appends messages by splitting the bot response on `"\n\n"` and replacing `**Sentiment Analysis:**` and `**Chatbot Response:**` markers.

## Known limitations and gotchas

- There are no unit tests or CI configured in this repo. Changes to the message contract (JSON shape or markdown markers) will break the frontend parsing.
- The code currently prints warnings when `MISTRAL_API_KEY` is missing; it does not raise an exception. Be aware of silent failures in local runs.
- The app runs in debug mode when started via `python app.py`. For production deploy, run behind Gunicorn or another WSGI server and set `debug=False`.

## Quick examples for AI edits

- If you modify the system prompt, show one before/after example of the model output and a unit test that asserts `json.loads(response)` contains `sentiment` and `response` keys.
- If you change model return format, update both `app.py` and the parsing block in `templates/index.html` — include a short note in the PR describing the new contract.

If any of these assumptions look wrong or you'd like the instructions expanded (run/test commands, a sample `.env.example`, or a tiny parsing unit test), tell me which area to expand and I will update this file.
