# Layers for progressive signal discovery

Start at layer 1. Go one layer down only when every signal you have is weak
(most hypotheses fit it) or no hypothesis fits all facts. Stop as soon as a
fact fits exactly one hypothesis. Cost and specialist knowledge rise as you
go down; an agent can afford to go further than a person usually does.

Ask for read-only access, scoped to the investigation. Do not change a live
system to gather a fact without the user's approval.

## Server: Linux host, container, Kubernetes or a container platform

| Layer | Example sources | Typical signals |
|---|---|---|
| 1. App and platform | app logs, error tracker, metrics, traces; `kubectl get events`, `kubectl describe pod`, `kubectl logs --previous`; platform revision/replica lists; deploy and config history; feature flags | error spikes, restarts, failed probes, a deploy or flag flip that lines up in time, a replica that differs |
| 2. Live config | `nginx -T`, `env` in the container, the live object (`kubectl get … -o yaml`) vs. the repo, image digests per replica, pod start times vs. config change times | config the process never loaded, replicas on different config or image |
| 3. Network | `dig`, `ss -tanp`, `curl -v` (status, headers, `Server`, body), `tcpdump`, proxy/ingress/sidecar logs | which layer answered (the error body and `Server` header), DNS answers, resets, slow handshakes |
| 4. Filesystem and storage | `lsof`, `strace -e trace=file`, `df -i`, `iostat`, mounts | files re-read per request, full disk or inodes, wrong mount |
| 5. Process and runtime | stack dumps (`py-spy dump`, `gdb`, `jstack`), CPU and heap profiles, thread and fd counts, GC logs | blocked threads, a hot function, heap growth, fd leaks |
| 6. Kernel and OS | `strace`, `dmesg`, cgroup stats (`/sys/fs/cgroup`), eBPF tools | OOM kills, CPU throttling, slow syscalls, stuck IPC |

## Web front end (browser)

| Layer | Example sources | Typical signals |
|---|---|---|
| 1. App | console errors, error tracker, server action / route handler logs, source maps, release and flag history | uncaught errors, a regression tied to a release |
| 2. Network | DevTools Network panel, HAR export, response headers and bodies | which server answered, redirects (3xx loops), CORS, cache misses, payload size |
| 3. Storage | DevTools Application panel: cookies, localStorage, IndexedDB, service workers | stale or missing cookies, an old service worker, quota errors |
| 4. Runtime | Performance and Memory panels, heap snapshots | long tasks, leaks, slow renders |
| 5. Device | CPU and network throttling, a real low-end device | failures only on slow devices or networks |

## Python / Django backend and its database

| Layer | Example sources | Typical signals |
|---|---|---|
| 1. App | request logs (is there a request line at all?), Django error log, middleware and throttle settings, the effective settings (`manage.py diffsettings`) | the request never reached Django, a setting that differs from the repo |
| 2. Database | `pg_stat_activity`, `pg_locks`, `EXPLAIN (ANALYZE, BUFFERS)`, slow query log, connection counts | lock waits, a missing index, pool exhaustion, a plan that changed |
| 3. Process | `py-spy dump`/`top`, worker counts and timeouts (gunicorn/uvicorn), memory per worker | a stuck worker, a timeout that kills workers, memory growth |

Add a table here for each new stack (mobile, macOS, queue workers) when a real
investigation needs one.
