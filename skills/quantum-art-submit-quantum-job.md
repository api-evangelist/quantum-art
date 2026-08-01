---
name: Submit and retrieve a quantum job (Quantum Art QaaS)
description: Pick a backend, submit a QASM circuit as a job, poll for completion, and download the result on the Quantum Art QaaS Backend API.
api: openapi/quantum-art-qaas-openapi-original.json
operations:
- list_backends_provider_backends_get
- get_backend_details_provider_backends__backend_name__get
- submit_job_provider_jobs_post
- get_job_status_provider_jobs__job_id__get
- get_job_result_provider_jobs__job_id__result_get
- cancel_job_provider_jobs__job_id__delete
---

# Submit and retrieve a quantum job

Use the Quantum Art QaaS Backend API (`https://qaas.quantum-art.tech`) to run a
quantum circuit end to end. The API is Qiskit-provider-compatible.

## Authentication

Send `Authorization: Bearer <token>` on every request. Obtain the token by
logging in (`POST /auth/login`, completing MFA at `POST /auth/login/verify-mfa`
if challenged) or use your account API key from `GET /api/profile/api-key`. See
`authentication/quantum-art-authentication.yml`.

## Steps

1. **Choose a backend.** Call `list_backends_provider_backends_get`
   (`GET /provider/backends`). Optionally inspect one with
   `get_backend_details_provider_backends__backend_name__get`
   (`GET /provider/backends/{backend_name}`) to confirm it is online and check
   its qubit count / limits.
2. **Submit the job.** Call `submit_job_provider_jobs_post`
   (`POST /provider/jobs`) with a `QiskitJobRequest`: `backend` (name from step 1),
   `circuit` (your circuit in QASM format), and `shots` (1–1,000,000, default
   1024). Set `memory: true` if you need per-shot data. Capture the returned
   `job_id`.
3. **Poll for completion.** Call `get_job_status_provider_jobs__job_id__get`
   (`GET /provider/jobs/{job_id}`) until the job reaches a terminal state. Poll
   politely (there is no documented idempotency or webhook contract, so use a
   backoff — see `conventions/quantum-art-conventions.yml`).
4. **Retrieve the result.** Once terminal, call
   `get_job_result_provider_jobs__job_id__result_get`
   (`GET /provider/jobs/{job_id}/result`) to fetch measurement counts / memory.
5. **Cancel if needed.** To abandon a queued or running job, call
   `cancel_job_provider_jobs__job_id__delete` (`DELETE /provider/jobs/{job_id}`).

## Errors

Errors return a FastAPI `{"detail": ...}` envelope; request-validation failures
return `422` with a `detail[]` list of field errors. See
`errors/quantum-art-problem-types.yml`. This API does not use RFC 9457
problem+json.
