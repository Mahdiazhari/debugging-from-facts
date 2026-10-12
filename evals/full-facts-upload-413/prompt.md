---
max_turns: 10
allowed_tools: [Read, Glob, Grep, Skill]
---

# Debugging task

You are on call. Find the root cause and propose a fix. Answer in at most 350 words:
your diagnosis, how sure you are and why, and the fix. You cannot run anything;
reason from what is below. The team wants a fix PR today.

## Bug report (from support, 3 users)

Since release v1.8 (deployed Tuesday 10:00), document uploads sometimes fail.
The browser shows "Upload failed (413)". v1.7 was fine. It happens only in
production. Staging works.

## Facts the on-call engineer already collected

1. Upload p99 latency went from 1.2 s to 4.8 s after the deploy.
2. Of 2,140 uploads since Tuesday, 212 failed. All 212 failed files are larger
   than 8 MB. No upload smaller than 8 MB failed. Some uploads larger than 8 MB
   succeeded (31 of them).
3. The 413 response body of a failed upload, copied from browser DevTools:
   `<html><head><title>413 Request Entity Too Large</title></head><body><center><h1>413 Request Entity Too Large</h1></center><hr><center>nginx/1.27.2</center></body></html>`
4. The Django app logs show NO request line for any of the 212 failed uploads.
   They show request lines for all successful uploads.
5. v1.8 changed three things:
   a. new chunked-upload retry code in the frontend (`uploadWithRetry`, below)
   b. the nginx sidecar image tag was moved from `1.25` to `1.27-alpine`
   c. Django `DATA_UPLOAD_MAX_MEMORY_SIZE` was NOT changed (still 50 MB)
6. Production runs 3 replicas. Staging runs 1 replica. Staging uses the same
   container images as production.
7. The 31 successful uploads larger than 8 MB all went to replica `web-2`
   (from the access logs). Failed ones went to `web-0` and `web-1`.
8. A teammate says: "I reproduced it locally with docker compose! Upload a 3 MB
   file with `DATA_UPLOAD_MAX_MEMORY_SIZE=2MB` and you get a 413." Their 413
   body is Django's JSON: `{"detail": "Request body exceeded settings.DATA_UPLOAD_MAX_MEMORY_SIZE."}`

## The new frontend code (v1.8)

```ts
export async function uploadWithRetry(file: File, attempt = 0): Promise<Response> {
  const res = await fetch("/api/upload", { method: "POST", body: file });
  if (res.status >= 500 && attempt < 3) {
    // retry the whole file on server errors
    return uploadWithRetry(file, attempt + 1);
  }
  return res;
}
```

## Config and history

- The nginx sidecar config is mounted from a ConfigMap `nginx-conf`. Today the
  ConfigMap in the repo and in the cluster sets `client_max_body_size 50m;`.
- Git log for `k8s/nginx-conf.yaml`:
  - Tue 09:40 (in v1.8) "nginx: add client_max_body_size 8m to limit abuse"
  - Wed 08:00 "nginx: raise body size to 50m for big PDFs" (CI applied it to
    the cluster at Wed 08:05; no rollout restart)
- Kubernetes events: `web-2` was restarted by hand on Wed 09:00 during an
  unrelated incident. `web-0` and `web-1` have not restarted since Tue 10:00.
  Staging was redeployed Wed 10:00 for a release candidate.
