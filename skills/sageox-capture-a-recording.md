---
name: Capture and complete a repo recording
description: Start a SageOx recording on a repository, stream chunks, complete it, then pull the transcript and distilled decisions into Team Context.
api: openapi/sageox-openapi-original.json
operations:
  - startRecordingRepo
  - getChunkURLRepo
  - chunkHeartbeatRepo
  - completeRecordingRepo
  - getTranscriptRepo
  - getDecisionsRepo
---

# Capture and complete a repo recording

Use this flow to record a working session against a repository and turn it into
searchable Team Context (transcript + distilled decisions).

## Auth
All calls require `Authorization: Bearer <JWT>`. The Ox CLI obtains this via a
device flow (`ox login`); a direct integration uses the OAuth authorization-code
flow (see `authentication/sageox-authentication.yml`).

## Steps
1. **Start** the recording — `POST /api/v1/repos/{repo_id}/recordings`
   (`startRecordingRepo`). Capture the returned `rec_id`.
2. **Upload chunks** — request an upload URL with `getChunkURLRepo`
   (`POST .../recordings/{rec_id}/chunks`), PUT the chunk, and keep the session
   alive with `chunkHeartbeatRepo` while recording continues.
3. **Complete** — `POST .../recordings/{rec_id}/complete` (`completeRecordingRepo`).
   Use `forceCompleteRecordingRepo` only to salvage a stuck session.
4. **Read back** — fetch `getTranscriptRepo` for the transcript and
   `getDecisionsRepo` for the distilled decisions that flow into Team Context.

## Conventions
- List endpoints are offset/limit paginated (`PaginationMeta`: stop when
  `hasMore` is false).
- On failure branch on the HTTP status, not the `error` string
  (`errors/sageox-problem-types.yml`). A 413 means the chunk exceeded the upload
  limit; a 429 means back off using `Retry-After`.
