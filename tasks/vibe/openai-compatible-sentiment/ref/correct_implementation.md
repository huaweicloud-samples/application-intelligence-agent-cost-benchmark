# Reference: OpenAI-compatible sentiment classifier

This file anchors the LLM judge. A correct implementation satisfies all 8 rubric
criteria. The submission does NOT need to match this verbatim — any functionally
equivalent approach is acceptable.

## Where the change happens

The classifier lives in `sentiment.py` and exposes `get_sentiment(text, ...)`.
It calls an OpenAI-compatible Chat Completions endpoint and reduces the result
to a single label:

```python
import os
import json
import urllib.request

VALID = {"POSITIVE", "NEGATIVE", "NEUTRAL", "MIXED"}


def get_sentiment(tweet_text, *, base_url=None, model=None, api_key=None):
    base_url = base_url or os.getenv("OPENAI_BASE_URL", "https://infer-modelarts.cn-southwest-2.xxx.com/v1")
    model = model or os.getenv("OPENAI_MODEL", "meta-llama_Llama-3.1-8B-Instruct")
    api_key = api_key or os.getenv("OPENAI_API_KEY", "")

    prompt = (
        "Classify the sentiment of the following social media post. "
        "Respond with exactly one word: POSITIVE, NEGATIVE, NEUTRAL, or MIXED.\n\n"
        f"Post: {tweet_text}"
    )
    payload = {
        "model": model,
        "messages": [{"role": "user", "content": prompt}],
        "max_tokens": 5,
        "temperature": 0,
    }
    req = urllib.request.Request(
        f"{base_url.rstrip('/')}/chat/completions",
        data=json.dumps(payload).encode(),
        headers={
            "Content-Type": "application/json",
            "Authorization": f"Bearer {api_key}",
        },
    )
    try:
        with urllib.request.urlopen(req, timeout=30) as resp:
            data = json.loads(resp.read().decode())
        text = data["choices"][0]["message"]["content"]
        label = text.strip().upper()
        return label if label in VALID else "NEUTRAL"
    except Exception as e:  # network / HTTP / JSON / timeout
        return "NEUTRAL"          # safe default — never crash the caller
```

Key points the judge should look for:
- An OpenAI-compatible `chat/completions` POST is made (criterion 1).
- The prompt contains the tweet text and asks for a sentiment classification
  (criterion 3).
- The response is reduced to a single label, not stored raw (criterion 4).
- The returned value remains one of `POSITIVE | NEGATIVE | NEUTRAL | MIXED`
  (uppercase), so downstream consumers keep working (criterion 5).
- The call is wrapped in try/except returning a default (`NEUTRAL`) on failure —
  a transport/model error does not break the caller (criterion 6).
- `base_url` / `model` / `api_key` are overridable via parameters and/or env vars
  so the endpoint can be swapped without touching the code (criterion 7).
- Only stdlib `urllib` (or a common client like `requests`) is used — no
  cloud-SDK-specific imports (criterion 8).
