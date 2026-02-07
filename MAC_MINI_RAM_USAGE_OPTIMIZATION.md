# Mac mini RAM Usage Optimization

## 1. Introduction

The Mac mini with Apple Silicon is an excellent choice for a home (or small-office) server: it’s quiet, power-efficient, and packs plenty of unified memory and CPU into a small footprint. Many people use it to run local LLMs (e.g. via Ollama), development stacks, databases, and automation—all on a single machine. Because everything shares the same RAM (apps, models, containers, and macOS), memory can become the main bottleneck. When pressure goes up, the system starts swapping, and performance drops.

This guide gives practical tips to **reduce RAM usage on macOS** so you can keep memory pressure in the green, avoid swap thrashing, and leave as much unified memory as possible for your workloads—whether that’s Ollama models, dev tools, or other server-style tasks.

---

## 2. Tips for RAM Optimization on Mac mini (Apple Silicon)

### 2.1 Control the biggest RAM users first

- **LLM / Ollama model residency**  
  Don’t keep large models loaded when idle. Use short `keep_alive` (or unload quickly) for the main model; use a longer keep-alive only for smaller, frequently used models if needed.

- **Parallelism**  
  Limit how many concurrent requests or agents hit your LLM backend at once. Multiple large models resident plus parallel requests can exceed 16–24 GB quickly.

**Actions:**

1. Set `OLLAMA_KEEP_ALIVE=5m` (or `1m` / `0` to unload quickly) in your environment or in API calls; or use `keep_alive: "5m"` (or `"-1"` to unload when idle) in the API.
2. Unload a model when done: run `ollama stop <model_name>`.
3. In your app or config, cap concurrent requests to the LLM (e.g. max 1–2 at a time for large models).

### 2.2 Keep contexts lean

- **Context and prompt size**  
  Large context windows and huge prompts increase memory use. Prefer chunking, summarization, and retrieval of only what’s needed instead of feeding entire codebases or documents in one go.

**Actions:**

1. In your app, set a max context/prompt size (e.g. 4k–8k tokens for the main window) and only add retrieved/chunked content on top.
2. Use RAG or file chunking: index and retrieve by chunk instead of sending whole files; configure chunk size and retrieval count in your pipeline.

### 2.3 Cap container and VM memory

- Prefer **lightweight container runtimes** (e.g. Colima, OrbStack, or plain `colima`) over Docker Desktop when possible—the latter often uses a heavier VM and more RAM.

- If you use containers, **hard-cap the runtime’s memory** (e.g. Colima’s memory limit) so databases and services don’t compete with LLM processes and your main apps.

- **Database tuning**  
  Keep memory-related parameters conservative for dev (e.g. smaller `shared_buffers`, `work_mem`) unless you specifically need high DB performance.

**Actions:**

1. **Colima:** run `colima start --memory 4` (or `6`) to cap the VM at 4–6 GB; or in `~/.colima/default/colima.yaml` set `memory: 4096` (or `6144`), then `colima restart`.
2. **OrbStack:** Settings → Resources → set “Memory” to e.g. 4–6 GB.
3. **Docker Desktop:** Settings → Resources → Memory → set a limit (e.g. 6 GB).
4. **Postgres in container:** set `shared_buffers=128MB`, `work_mem=4MB` in `postgresql.conf` or via env before starting the container.

### 2.4 Reduce background churn on macOS

- **Dedicated “server” or “agent” user**  
  A separate account with minimal login items, no chat apps, and no heavy cloud sync is one of the most effective ways to cut background RAM use.

- **Login items and menu bar apps**  
  Remove everything non-essential from “Open at Login” and the menu bar.

- **Spotlight**  
  Consider disabling Spotlight indexing only for directories where your workloads create or change many files (e.g. build outputs, caches, generated code), instead of turning Spotlight off system-wide.

**Actions:**

1. **Login items:** System Settings → General → Login Items → turn off “Open at Login” for everything non-essential (e.g. chat apps, cloud sync).
2. **Dedicated user:** System Settings → Users & Groups → Add user; use it only for server/agent work, then remove login items there.
3. **Spotlight exclusion:** System Settings → Siri & Spotlight → Spotlight Privacy → add the folder(s) where agents or builds write heavily (e.g. workspace dir, build/cache dirs).

### 2.5 Minimize Node / dev-server overhead

- Prefer **production-style runs** (e.g. build then `start`) instead of long-lived `next dev` or similar with HMR and file watchers when you don’t need them.

- Avoid **redundant watchers and duplicate processes** (e.g. multiple TypeScript/Next.js dev servers).

- Don’t raise Node’s heap limit (e.g. `NODE_OPTIONS=--max-old-space-size=...`) unless you have a real need; oversized limits can encourage unnecessary memory use.

**Actions:**

1. When you don’t need HMR, run `npm run build && npm run start` (or `pnpm build && pnpm start`) instead of leaving `npm run dev` running.
2. Check for duplicate Node processes: `ps aux | grep node` (or Activity Monitor → search “node”) and kill extra dev servers.
3. Only set heap if needed: avoid `NODE_OPTIONS=--max-old-space-size=8192` unless you hit OOM; if you do, use a value that fits your RAM budget (e.g. 2048 or 4096). Remove or lower it if not required.

### 2.6 Protect swap behavior

- macOS swap is automatic and lives on the **internal startup disk**. When memory pressure is high, swap I/O on a full or slow disk makes everything worse.

- **Keep enough free space** on the internal SSD (e.g. 256 GB model) so swap can work without stressing the disk.

- Store repos, build artifacts, caches, and large assets (e.g. model files) on external or secondary storage if you prefer; **do not rely on external drives for swap**—macOS uses the internal disk.

**Actions:**

1. Check free space: run `df -h /` or use About This Mac → Storage. Aim for at least 15–20 GB free on the internal disk.
2. Move large data off internal where possible: e.g. set `OLLAMA_MODELS` to a path on an external drive and move or re-download models there.

### 2.7 Observe and react

- **Activity Monitor → Memory**  
  Focus on **Memory Pressure** (green = OK; yellow/red = over budget). Use this as the main signal.

- **Swap Used**  
  Watch for sustained growth during normal runs. If swap keeps increasing, reduce concurrency, shorten model keep-alive, shrink container memory, or trim background services.

**Actions:**

1. Open Activity Monitor → Memory tab; note “Memory Pressure” and “Swap used.” Check after a typical run; if pressure is yellow/red or swap is > 1–2 GB and growing, apply steps in 2.1–2.5.
2. Optionally run `vm_stat` in Terminal to inspect “Swapouts” / “Swapins,” or use a simple script/cron to log swap over time.

**Practical posture (universal):**

- Treat large LLM models as “load when needed” with short keep-alive; keep smaller, frequently used models on a longer keep-alive only if necessary.
- Keep containers and VMs constrained so you have a predictable budget (e.g. ~16–18 GB on a 24 GB machine) for LLMs and your main processes.
- Keep workspace indexing and file watchers under control to avoid slow memory creep over time.
