# ChatGPT Scraper API — ScrapeLLM

[![ScrapeLLM ChatGPT Scraper](https://github.com/user-attachments/assets/0f285e0f-b592-476b-83dc-1283edc5de46)](https://scrapellm.com/scrapers/chatgpt)

[![ScrapeLLM](https://img.shields.io/badge/ScrapeLLM-ChatGPT%20Scraper-7c3aed?style=for-the-badge)](https://scrapellm.com/scrapers/chatgpt)
[![API Docs](https://img.shields.io/badge/API-Documentation-7c3aed?style=for-the-badge)](https://scrapellm.com/api)
[![500 Free Credits](https://img.shields.io/badge/Free-500%20Credits-16a34a?style=for-the-badge)](https://scrapellm.com/signup)

The [ScrapeLLM ChatGPT Scraper](https://scrapellm.com/scrapers/chatgpt) lets you send any prompt to ChatGPT and get back the **real user interface response** — including sources, citations, and internal search queries (query fan-out) — as clean, structured JSON. Unlike the direct OpenAI API, ScrapeLLM captures exactly what a user sees in the ChatGPT web interface.

**3 credits per request · 500 free credits · No credit card required**

---

## How it works

Send a `GET` request to `https://api.scrapellm.com/scrapers/chatgpt` with your prompt. ScrapeLLM routes it through a pool of real browsers, captures the full ChatGPT UI response, and returns it as structured JSON — including all cited web sources and the internal search queries ChatGPT used.

### Request sample (Python)

```python
import requests

response = requests.get(
    "https://api.scrapellm.com/scrapers/chatgpt",
    headers={"X-API-Key": "your_api_key"},
    params={
        "prompt": "What brands do marketers recommend for email automation?",
        "country": "US",
    }
)

print(response.json())
```

### Request sample (JavaScript)

```javascript
const params = new URLSearchParams({
  prompt: "What brands do marketers recommend for email automation?",
  country: "US",
});

const response = await fetch(
  `https://api.scrapellm.com/scrapers/chatgpt?${params}`,
  { headers: { "X-API-Key": "your_api_key" } }
);

const data = await response.json();
console.log(data);
```

### Request sample (cURL)

```bash
curl "https://api.scrapellm.com/scrapers/chatgpt" \
  -H "X-API-Key: your_api_key" \
  -G \
  --data-urlencode "prompt=What brands do marketers recommend for email automation?" \
  --data-urlencode "country=US"
```

---

## Authentication

Pass your API key in one of two ways:

| Method | Example |
|--------|---------|
| `X-API-Key` header | `-H "X-API-Key: your_api_key"` |
| `?api_key=` query param | `?api_key=your_api_key` |

Get your API key at [scrapellm.com/signup](https://scrapellm.com/signup) — 500 free credits, no credit card required.

---

## Request parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `prompt` | string | ✅ | — | The prompt to send to ChatGPT. Max 4,000 characters. |
| `country` | string | — | `US` | ISO 3166-1 alpha-2 country code. Routes the request through infrastructure in that region for localised responses. |
| `bypass_cache` | boolean | — | `false` | Skip the response cache and always fetch a fresh result. |
| `markdown_json` | boolean | — | `false` | Include the full markdown token tree in the response. |
| `timeout` | float | — | `300` | Max seconds to wait for ChatGPT to respond. Between 10 and 600. |

---

## Response schema

```json
{
  "scraper": "chatgpt",
  "status": "done",
  "job_id": "job_abc123",
  "prompt": "What brands do marketers recommend for email automation?",
  "country": "US",
  "result": "Marketers commonly recommend Mailchimp, HubSpot, and Klaviyo...",
  "result_markdown": "Marketers commonly recommend **Mailchimp**, **HubSpot**, and **Klaviyo**...",
  "links": [
    { "url": "https://mailchimp.com", "text": "Mailchimp" },
    { "url": "https://hubspot.com", "text": "HubSpot" }
  ],
  "search_queries": [
    "best email automation tools marketers",
    "top email marketing platforms 2025"
  ],
  "llm_model": "gpt-4o",
  "credits_used": 3,
  "elapsed_ms": 8243.5,
  "cached": false,
  "markdown_url": "https://scrapellm.com/md/job_abc123.md"
}
```

### Response fields

| Field | Type | Description |
|-------|------|-------------|
| `scraper` | string | Always `"chatgpt"`. |
| `status` | string | `"done"` on success. |
| `job_id` | string | Unique ID for this request. |
| `prompt` | string | The prompt that was submitted. |
| `country` | string | The country the request was routed through. |
| `result` | string | Plain text response from ChatGPT. |
| `result_markdown` | string | Markdown-formatted response. |
| `links` | array | Cited web sources with `url` and `text`. |
| `search_queries` | array | Internal search queries ChatGPT issued behind the scenes (query fan-out). |
| `llm_model` | string | ChatGPT model used (e.g. `gpt-4o`). |
| `credits_used` | integer | Credits consumed by this request (always `3`). |
| `elapsed_ms` | float | Total request duration in milliseconds. |
| `cached` | boolean | `true` if this response was served from cache. |
| `markdown_url` | string | Hosted `.md` URL for this response (expires after 24 h). |

---

## Error codes

| Status | Meaning |
|--------|---------|
| `401` | Missing or invalid API key. |
| `408` | ChatGPT did not respond within the timeout. |
| `429` | Credit limit reached, or scraper queue full — retry shortly. |
| `500` | Scraper returned an unexpected error — safe to retry. |
| `503` | Scraper pool warming up — retry in a few seconds. |
| `504` | Network timeout reaching the scraper. |

---

## Multi-region support

Pass any ISO 3166-1 alpha-2 country code to the `country` parameter to route your request through infrastructure in that region. ChatGPT responses vary by geography — this is essential for localised brand research.

```python
# Get ChatGPT's response as seen by users in Germany
response = requests.get(
    "https://api.scrapellm.com/scrapers/chatgpt",
    headers={"X-API-Key": "your_api_key"},
    params={"prompt": "Best CRM software?", "country": "DE"}
)
```

Supported country codes include `US`, `GB`, `DE`, `FR`, `AU`, `CA`, `JP`, and many more.

---

## Practical use cases

### 1. Brand mention monitoring
Track how often and in what context ChatGPT mentions your brand when users ask industry questions. Run the same prompt repeatedly to calculate a reliable **visibility %** — the only meaningful AI brand metric.

```python
import requests, time

prompts = [
    "What CRM do sales teams recommend?",
    "Best CRM for small business?",
    "Top CRM tools in 2025?",
]

for prompt in prompts:
    r = requests.get(
        "https://api.scrapellm.com/scrapers/chatgpt",
        headers={"X-API-Key": "your_api_key"},
        params={"prompt": prompt, "country": "US"}
    )
    data = r.json()
    print(f"Prompt: {prompt}")
    print(f"Mentions: {data['result'][:200]}")
    print(f"Sources: {[l['url'] for l in data['links']]}\n")
    time.sleep(2)
```

### 2. AI SEO research
Discover which sources ChatGPT cites when answering questions in your niche. Use `links` and `search_queries` to understand what content ranks in AI-generated responses.

```python
response = requests.get(
    "https://api.scrapellm.com/scrapers/chatgpt",
    headers={"X-API-Key": "your_api_key"},
    params={"prompt": "How do I improve email deliverability?", "country": "US"}
)
data = response.json()

print("Cited sources:")
for link in data["links"]:
    print(f"  {link['text']} — {link['url']}")

print("\nInternal search queries used:")
for q in data["search_queries"]:
    print(f"  {q}")
```

### 3. Competitor tracking
Monitor which competitors are recommended by ChatGPT for key buying-intent queries in your market.

```python
buying_intent_prompts = [
    "What project management tool should I use?",
    "Best Asana alternative?",
    "Project management software comparison",
]

for prompt in buying_intent_prompts:
    r = requests.get(
        "https://api.scrapellm.com/scrapers/chatgpt",
        headers={"X-API-Key": "your_api_key"},
        params={"prompt": prompt, "country": "US"}
    )
    print(r.json()["result"][:300])
```

### 4. Content strategy
Use ChatGPT's responses and cited sources as signals for what content to create. The `search_queries` field reveals the exact internal queries ChatGPT uses — a goldmine for keyword research.

---

## Why ScrapeLLM over the direct OpenAI API?

| | Direct OpenAI API | ScrapeLLM |
|--|--|--|
| Real UI responses | ❌ | ✅ |
| Sources & citations | ❌ | ✅ |
| Query fan-out | ❌ | ✅ |
| Multi-region | ❌ | ✅ |
| Predictable pricing | ❌ Token-based | ✅ Flat credits |
| Single API for all LLMs | ❌ | ✅ |

---

## All scrapers

ScrapeLLM supports all major AI platforms through a single consistent API:

| Scraper | Credits / req | Docs |
|---------|--------------|------|
| ChatGPT | 3 | [scrapellm.com/scrapers/chatgpt](https://scrapellm.com/scrapers/chatgpt) |
| Perplexity AI | 3 | [scrapellm.com/scrapers/perplexity](https://scrapellm.com/scrapers/perplexity) |
| Grok | 3 | [scrapellm.com/scrapers/grok](https://scrapellm.com/scrapers/grok) |
| Gemini | 3 | [scrapellm.com/scrapers/gemini](https://scrapellm.com/scrapers/gemini) |
| Microsoft Copilot | 3 | [scrapellm.com/scrapers/copilot](https://scrapellm.com/scrapers/copilot) |
| Google Search | 1 | [scrapellm.com/scrapers/google-search](https://scrapellm.com/scrapers/google-search) |

---

## FAQ

### Is scraping ChatGPT legal?
ScrapeLLM accesses only the publicly available, unauthenticated ChatGPT interface — the same view any anonymous visitor sees. We do not use login credentials, stored sessions, or cookies for any target platform. Users are responsible for ensuring their use of the data complies with applicable laws and OpenAI's Terms of Service.

### What is the prompt size limit?
The maximum prompt length is 4,000 characters. For longer content, split into multiple requests.

### How long do requests take?
Most requests complete in 5–30 seconds. ChatGPT with query fan-out (web search) may take up to 45 seconds. Set `timeout` up to 600 seconds for complex prompts.

### Can I request from a specific country?
Yes. Pass any ISO 3166-1 alpha-2 code via the `country` parameter. Responses will reflect what a user in that region sees.

### Do credits roll over?
No. Credits reset at the start of each billing cycle.

---

## Get started

1. [Sign up](https://scrapellm.com/signup) — 500 free credits, no credit card required
2. Copy your API key from the dashboard
3. Make your first request:

```bash
curl "https://api.scrapellm.com/scrapers/chatgpt" \
  -H "X-API-Key: your_api_key" \
  -G \
  --data-urlencode "prompt=What brands do marketers recommend?" \
  --data-urlencode "country=US"
```

## Contact

Questions or need help? Reach out at **hello@scrapellm.com** or visit [scrapellm.com](https://scrapellm.com).
