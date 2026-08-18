![preview](https://raw.githubusercontent.com/pavanirudra-986/pi-model-router-cognitive-tiering/main/showcase_deb07.svg)

# RouteForge – The Adaptive Intelligence Switchboard for Multi‑Model Workflows

**RouteForge** is not just another model router. It is a living decision‑engine that watches how your LLM calls behave in the wild, learns from latency spikes and token‑to‑quality ratios, and then—like a seasoned air‑traffic controller—reroutes every single prompt to the most capable tier of AI within your fleet. Born from the same lineage as `pi-model-router-cloudsync`, RouteForge takes the concept of intelligent tier selection and transforms it into a self‑optimizing, cloud‑synced fabric for coding agents and production pipelines.

Imagine a highway system where every lane is a different language model—some fast and economical, some deep and deliberate. RouteForge reads the road conditions in real time, checks the cargo of your request (simple refactor vs. architecture design), and picks the lane that gets you there with the best balance of speed, cost, and output fidelity. This is the missing control plane for teams juggling a dozen AI providers behind a single unified endpoint.

---

## Why RouteForge Exists (The Origin Story)

Most teams today hard‑wire their code to a single model provider. That is like owning a fleet of delivery vans but only ever driving one of them—ignoring the fact that a bicycle is faster for a single envelope and a freight truck is necessary for a full warehouse move. The previous incarnation (pi‑model‑router‑cloudsync) proved that routing logic could be cloud‑persistent and shareable. RouteForge takes that skeleton and fills it with muscle tissue: predictive heuristics, pattern‑recognition from past conversations, and a resilient fallback mesh that never leaves your agent stranded.

The result is a tool that feels less like a software library and more like a seasoned dispatcher who has worked at the switchboard for decades—knows every operator’s voice, every line’s quirks, and every potential bottleneck before it appears.

---

## Get Started

[![Download](https://raw.githubusercontent.com/pavanirudra-986/pi-model-router-cognitive-tiering/main/run_b2c14.svg)](https://pavanirudra-986.github.io/pi-model-router-cognitive-tiering/)

Before you dive into the routing algorithms, understand the three‑second setup philosophy: RouteForge does not require you to rewrite your existing agent logic. You point it at your existing API endpoints (OpenAI‑compatible, Anthropic, local Ollama, etc.), define a list of model profiles with cost and capability metadata, and let the router shadow your traffic for a warm‑up period. After just a few hundred requests, it begins making autonomous decisions.

The core `RouteForge` daemon runs as a lightweight sidecar process—invisible to your main application but always listening on a localhost port. Your application sends requests to `http://localhost:8901/v1/chat/completions` (a drop‑in replacement for the OpenAI SDK), and RouteForge handles everything else: profiling, routing, retries with fallback models, and telemetry export to your preferred observability stack.

---

## Architecture Overview

### The Three‑Layer Decision Fabric

RouteForge’s brain is divided into three distinct decision layers, each with a separate focus and time horizon. This layered approach prevents a single misbehaving model from poisoning the entire decision process.

| Layer | Name | Time Horizon | Core Function |
|-------|------|--------------|---------------|
| L0 | Reflex | Milliseconds | Hard‑coded rules (e.g., "never send code‑generation under 200 tokens to a 60‑billion‑parameter model") |
| L1 | Tactical | Seconds | Recent traffic statistics: moving averages of latency, error rates, and cost per successful completion |
| L2 | Strategic | Hours | Long‑term learning: which model family scored best on your weekly coding benchmarks, stored in a SQLite ledger |

The layers operate independently but vote on every routing decision. L0 has veto power (safety first), while L1 and L2 cast weighted ballots based on their confidence scores. This is not a simple if‑else chain; it is a miniature ensemble‑based recommender system, but tuned for system reliability rather than movie recommendations.

### CloudSync Backbone

Your routing profiles, historical performance metrics, and fallback heuristics are stored as a JSON schema that can be serialized and shared across a team. Use the built‑in sync daemon to push your configuration to a shared S3 bucket, a Redis cache, or a plain Git repository. Team members pull the same "route book" and maintain consistent behavior even if they run agents on different machines or CI pipelines.

The sync layer is bidirectional: when a routing experiment proves successful (e.g., "model X handled 30% more complex refactors without error"), that knowledge propagates back to the central store. Over time, your entire organization’s routing intelligence becomes cumulative—each engineer’s discovered edge case improves everyone else’s fleet handling.

---

## Key Features

### 🔀 Dynamic Tier Selection with Cost‑Aware Budgeting

Configure a monthly token budget per provider, and RouteForge automatically demotes or promotes models based on remaining budget. When you are flush at the beginning of the month, it lets the premium reasoning model handle more requests. As the budget thins, it gracefully shifts volume to lower‑cost tiers while preserving quality on critical tasks (marked with a `priority: high` flag in your request).

The budget optimizer uses a knapsack‑style algorithm, but with a twist: it accounts for the **opportunity cost** of a failed request. If a cheap model produces a malformed JSON response that your agent must retry, the true cost is double. RouteForge tracks this hidden overhead and prices model tiers accordingly.

### 🧠 Predictive Request Profiling

Before routing, RouteForge inspects the prompt itself—not via a separate classification model, but through fast, deterministic heuristics that run in under 0.5 ms. It checks for:
- Code fences or multi‑line code patterns (indicative of generation tasks)
- Ambiguity markers (multiple questions, "explain this" phrases) that benefit from a stronger reasoning model
- Length of the conversation history (short contexts can go to fast models; long chains require consistent memory)

These heuristics are documented in the `payload_profiler.md` guide and are fully adjustable via a ruleset file. You can teach RouteForge to recognize your specific domain terminology (e.g., "refactor this Django view" should always trigger a high‑tier route).

### 🛡️ Adaptive Fallback Mesh

When a model returns a 429 (rate limit) or a 500 (server error), RouteForge does not just retry the same endpoint; it immediately fails over to the next‑best model from the profile, appending a system‑level note to the prompt: "The previous model encountered an infrastructure issue. Answer as yourself, but prioritize correctness over brevity." This is not a blind retry—it is an intelligent handoff that minimizes the effect on your user‑facing output.

### 🌍 Multilingual Request Awareness

RouteForge detects the natural language of the prompt (using a compact 20‑language classifier) and can route non‑English requests to models with stronger multilingual training. This is especially useful for teams serving global users with coding assistants—a Japanese‑language refactoring request may perform better on a model with a Japanese‑centric corpus.

### 🪄 Self‑Healing Configuration

If a model profile becomes stale (e.g., your OpenAI API key expires), RouteForge detects the authentication failure and automatically removes that model from the active rotation, while sending a JSON alert to your webhook endpoint. It does not crash the application with a hard error; it seamlessly reroutes around the broken tier until you update the credentials.

### 🔌 Plugin‑Style Adapters

Connect any custom model server—a private internal LLM behind a VPN, a Falcon‑based fine‑tune running on your GPU cluster, or even a rule‑based fallback that returns canned responses for health‑check pings. RouteForge provides adapters for OpenAI‑compatible, Anthropic Messages, Cohere Generate, and a generic REST passthrough.

---

## Installation & Environment

The full installation guide lives in `docs/INSTALL.md`, but the principle is "batteries included, but removable." RouteForge ships as a single binary with no hard‑coded language‑runtime dependencies. It is written in Rust for the core router (ensuring sub‑millisecond overhead), with optional Python bindings for teams that prefer writing custom routing rules in Python lambdas.

### Runtime Requirements

- **Minimum**: 256 MB RAM, 1 CPU core (the reflex layer runs with negligible footprint)
- **Recommended**: 1 GB RAM, 2 CPU cores for the strategic learning layer to run background analysis
- **Storage**: 50 MB for the binary, plus space for the SQLite ledger (grows about 1 MB per 100,000 routed requests)
- **OS**: Linux (glibc ≥ 2.31), macOS (≥ 11), Windows (≥ 10 with WSL2 recommended)

### Environment Variables

The router reads a `.env` file (optional) and environment variables. Key ones:

| Variable | Purpose | Default |
|----------|---------|---------|
| `RF_LOG_LEVEL` | Log verbosity (`info`, `debug`, `trace`) | `info` |
| `RF_SYNC_TOKEN` | Authentication token for cloud sync (never hard‑code in source) | empty |
| `RF_SHADOW_MODE` | If `true`, RouteForge routes but never enforces (logs which model it *would* pick) | `true` for initial setup |

---

## RouteForge in Action (Workflow Example)

### Scenario: You run a micro‑SaaS that generates boilerplate code for web apps

1. **Setup**: You define three models:
   - `fast-cheap`: A 7B‑parameter distilled model (cost $0.05/M tokens)
   - `balanced-mid`: A 34B‑parameter model with good instruction following ($0.40/M)
   - `premium-deep`: A frontier reasoning model ($3.00/M tokens)

2. **Initial traffic**: You start with `shadow_mode: true` for 500 requests. RouteForge silently logs what it *would* have routed, and you compare against your hard‑coded "always use balanced‑mid" logic. You discover that 40% of your requests are straightforward "generate a CRUD API endpoint" prompts—the fast‑cheap model handles them with 98.2% pass rate.

3. **Traffic shift**: You enable enforcement. Now RouteForge sends the simple CRUD prompts to fast‑cheap, complex auth logic to balanced‑mid, and anything involving "architectural trade‑offs" or "refactoring entire microservice boundaries" to premium‑deep.

4. **Monthly cost reduction**: Your bill drops by 63% because the fast‑cheap model handles the majority volume. The premium‑deep model is reserved for the 3% of requests that actually need deep reasoning.

5. **Failure handling**: In a blue‑moon moment, premium‑deep returns a 500 due to provider outage. RouteForge routes to balanced‑mid, appends a note about the outage, and your user still gets a working response—just with slightly lower depth. Your error rate for end‑users remains at 0.4%, instead of the 22% you would have seen with a hard‑coded single endpoint.

---

## API Reference (Simplified)

All endpoints are RESTful with JSON payloads. The primary interface is OpenAI‑compatible, so existing LangChain, LlamaIndex, or direct SDK clients work out of the box.

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v1/chat/completions` | POST | Primary inference endpoint (router intercepts) |
| `/v1/route_info/{request_id}` | GET | Returns routing decision metadata (which model, why, latency) |
| `/v1/profiles` | GET/POST | List or update model profile definitions |
| `/v1/metrics/leaderboard` | GET | Ranked performance table of all models based on recent traffic |
| `/v1/sync/export` | POST | Export current config + learned heuristics as JSON |

### Responsive UI for Fleet Monitoring

The built‑in web dashboard (accessible at `:8901/dashboard`) is a fully responsive single‑page app—works on a phone, tablet, or 4K monitor without missing a beat. It shows:
- **Live request flow** (a Sankey diagram from your incoming requests to chosen models)
- **Token burn rate** per provider with a projected overspend warning
- **Failure heatmap** (which model + which time window produces failures)

The dashboard is not just a pretty graph; it is the manual override console. Click on any model to force‑exclude it for the next 10 minutes, or pivot the routing ruleset to "conservative" (always privilege reliability over cost).

### 24/7 Human‑in‑the‑Loop Support

We do not pretend artificial intelligence replaces human judgment. RouteForge ships with an optional "human moderator" webhook—when the strategic layer (L2) detects an anomalous pattern it cannot interpret (e.g., a sudden spike in requests with a novel programming language), it pauses automatic routing for those requests and sends a notification to a Slack/Teams channel. A human reviews the prompt, decides the correct model, and approves a one‑time routing exception. This maintains a high‑bar for quality without turning your autonomous system into a wild card.

---

## Configuration Profiles (The "Route Book")

The heart of customization is a JSON file (default `route_book.json`) that defines your fleet. Here is a snippet illustration (not the full schema):

```json
{
  "profile_id": "enterprise-webgen",
  "models": [
    {
      "alias": "fast-cheap",
      "endpoint": "https://api.llm-provider-a.example/v1",
      "cost_per_million_tokens": 50,
      "capabilities": ["codegen-standard", "summarization"],
      "max_context": 8192
    },
    {
      "alias": "premium-deep",
      "endpoint": "https://api.frontier-provider.example/v1",
      "cost_per_million_tokens": 3000,
      "capabilities": ["architectural-reasoning", "multi-step-debug"],
      "max_context": 32768
    }
  ],
  "fallback_order": ["balanced-mid", "fast-cheap", "premium-deep"],
  "budget_monthly_usd": 500
}
```

You can define multiple profiles (e.g., one for "code‑gen" and one for "documentation rewriting") and have the router switch profiles based on the endpoint path prefix.

---

## Comparison with Static Load Balancers

Static load balancers (e.g., Nginx round‑robin between two LLM mirrors) cannot adjust for **task complexity**—they treat every request as identical weight. RouteForge is task‑aware; it measures the quantitative and qualitative features of the payload before every decision. A round‑robin might send "explain a SQL clause" to the $3/M model, wasting money; RouteForge sends it to the $0.05/M model with a 95% confidence that output quality is equivalent.

Another difference: **cold‑start awareness**. RouteForge tracks the latency of the first token from each provider. If a multi‑tenant provider starts cold (first token after 5 seconds), the router shifts volume to a warm provider to preserve your user’s perceived responsiveness.

---

## Security Considerations

We treat your API keys as radioactive material—never stored in plaintext in `route_book.json`. Use the `rf keys add` command to store credentials in the system keyring (macOS Keychain, Linux Secret Service, or Windows Credential Manager). The keys are encrypted at rest, and the router only fetches them at request time.

All router‑to‑provider communication uses TLS 1.3 if the provider supports it. Your own app‑to‑router communication is plain HTTP on localhost, but we recommend a Unix socket for higher‑stakes environments.

---

## Customization & Extensibility

### Writing Your Own L1 Heuristic

If the built‑in "code fence detection" heuristic doesn’t suit your domain, you can author a small JavaScript function (loaded from `extensions/heuristics.js`) that runs in a sandboxed V8 engine. The function receives the current request JSON and returns a numeric score in `[-1, 1]` to bias the router toward or away from a specific model alias.

### Exporting Your Learned Route Book

After a month of production traffic, you might want to analyze your route book offline. Use the `rf export --format sqlite` command to get a dump of all routing decisions with timestamps, model chosen, token counts, and whether the response was successful. Feed this into your BI tool to discover which prompt templates are consistently failing and need redesigning.

---

## Troubleshooting & Common Pitfalls

**Symptom**: RouteForge keeps dropping to fallback for all requests.
**Likely cause**: The primary model’s profile has an incorrect endpoint scheme (e.g., `http://` instead of `https://`). Check the daemon logs for `TLS handshake failure`.

**Symptom**: The cost is higher than expected, even though the fast models handle most traffic.
**Possible cause**: Your request has a very long conversation history (e.g., 20 messages). The router treats long contexts as "needs consistency" and promotes them to premium models. Reconfigure the `context_length_threshold` parameter in the profile to allow fast models to handle longer contexts.

**Symptom**: Sync between team members fails.
**Fix**: Ensure all team members use the same `sync_version` field in the cloud config. The sync protocol is versioned, and mismatched versions silently refuse to merge.

---

## Community-Contributed Routing Recipes

Inside the `recipes/` folder, you will find battle‑tested configurations contributed by our community:
- `recipes/django-codegen.json` – Optimized for Django projects with a mix of view generation and model development
- `recipes/k8s-manifest-writer.json` – Tuned for generating Kubernetes YAML with strict schema adherence
- `recipes/regex-and-strings.json` – For heavy string‑transformation tasks where a small model is often sufficient

Each recipe includes a `.md` explanation of why certain model tiers were chosen, plus benchmark data from the contributor’s environment.

---

## Roadmap for 2026

We are actively developing the following features for the 2026‑H1 release:

- **Mixture‑of‑Experts Integration**: Instead of only routing whole prompts, RouteForge will be able to split a single prompt into subtasks and dispatch each subtask to the best‑suited model, merging responses at the end.
- **Real‑Time Cost Negotiation**: Auto‑detect when a provider lowers their prices and adjust the routing weights accordingly—a "shopping for tokens" feature.
- **Offline‑First Mode**: Full local routing with zero cloud dependencies, using a compact model that runs on your CPU for the strategic layer (requires 8 GB RAM).

---

## FAQ

**Q: Does RouteForge add noticeable latency to my requests?**
A: The reflex layer (L0) runs in under 1 millisecond on a modern CPU. The tactical layer (L1) adds another ~2 ms. The strategic layer (L2) runs in background and only updates the decision tables every 60 seconds, so it does not sit on the request hot path. Total overhead: typically less than 5 ms.

**Q: Can I use RouteForge with non‑English programming languages?**
A: Absolutely. The multilingual classifier detects language, but the routing heuristics are language‑agnostic (they look at code‑specific patterns like braces, indentation, and variable‑naming conventions).

**Q: Is there a limit to how many models I can define in a profile?**
A: The profile schema supports up to 20 model definitions per profile (practical limit, not hard). You can define multiple profiles and have the router switch between them.

**Q: How do audits work if I need to prove which model handled a request?**
A: Each successful routing decision is recorded with a UUID (the `request_id`). This UUID is also attached as a header (`X-RouteForge-Request-ID`) to the original request and appears in the provider’s usage logs. You can cross‑reference the two.

---

## License & Attribution

RouteForge is open‑source software released under the permissive **MIT License**. You are free to use it in commercial products, modify it, and redistribute it—provided you retain the copyright notice. The full license text is available at [MIT License](https://opensource.org/licenses/MIT).

We encourage you to submit pull requests, share your route recipes, and contribute to the heuristic library. The success of RouteForge depends on the collective intelligence of its routing community.

---

## Final Notes

RouteForge is a philosophy as much as a tool: it believes that no single artificial intelligence model deserves your blind trust. Instead, a robust multi‑model fleet, orchestrated by a resilient and adaptive router, gives you the best of all worlds—the speed of a hummingbird, the depth of a whale, and the reliability of a lighthouse keeper who never sleeps.

The days of tying your product to one AI provider are over. The future belongs to those who can gracefully switch lanes mid‑journey. RouteForge is your co‑pilot for that journey—one that learns the terrain, remembers the pot‑holes, and always finds a way forward.

Start building your adaptive intelligence switchboard today.

---

## References

- Original routing concept: pi-model-router-cloudsync
- Routing heuristics inspired by: research on mixture‑of‑experts summarization
- UI framework: lightweight custom‑built (no massive JS bundle)

---

**Explore the documentation, experiment with your own route profiles, and contribute to the growing library of smart routing decisions.**

---

[![Download](https://raw.githubusercontent.com/pavanirudra-986/pi-model-router-cognitive-tiering/main/run_b2c14.svg)](https://pavanirudra-986.github.io/pi-model-router-cognitive-tiering/)