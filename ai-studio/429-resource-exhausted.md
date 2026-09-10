# 429 RESOURCE_EXHAUSTED: Resource has been exhausted (e.g. check quota)

> Reduce request volume, confirm your Google AI Studio or Gemini API quota, add retry backoff, and verify you are using a supported model in the correct project. Most 429 RESOURCE_EXHAUSTED errors clear after quota resets, rate smoothing, billing activation, or moving production workloads onto a higher approved usage tier safely.

## Why This Happens

This error usually appears when your project sends too many requests too quickly, exceeds token or request quotas, or targets a model that is temporarily capacity constrained. It can also happen when billing is not fully configured, the wrong project is selected, or retries are too aggressive.

## Step-by-Step Resolution

### 1. Check current quota and billing status

Open Google AI Studio or the linked Google Cloud project and confirm that:

- Billing is enabled and active
- The correct project owns the API key
- Daily and per-minute quotas are not exhausted
- The model you are calling is available for your account tier

### 2. Slow down request bursts with exponential backoff

If you are sending many parallel prompts, spread them out and retry gradually.

```js
async function callWithBackoff(makeRequest, retries = 5) {
  for (let attempt = 0; attempt <= retries; attempt += 1) {
    try {
      return await makeRequest();
    } catch (error) {
      if (attempt === retries || !String(error.message || error).includes('RESOURCE_EXHAUSTED')) {
        throw error;
      }

      const jitter = Math.floor(Math.random() * 300);
      const delay = 1000 * 2 ** attempt + jitter;
      await new Promise((resolve) => setTimeout(resolve, delay));
    }
  }
}
```

### 3. Reduce token-heavy prompts and large batch sizes

Large prompts, large candidate counts, and wide concurrency settings consume quota faster. Trim unnecessary context, lower output size, and process jobs in smaller batches.

```bash
# Example strategy
# - cut prompt context
# - lower parallel workers
# - retry after quota window resets
```

### 4. Verify the request is using the intended model and key

A stale key, wrong project, or unsupported model name can trigger quota-like failures. Double-check environment variables, deployment secrets, and model IDs before retrying.

```env
GEMINI_API_KEY=your_active_project_key
GEMINI_MODEL=gemini-1.5-flash
```

### 5. Wait for quota reset or request a higher limit

If usage is legitimate and sustained, wait for the next quota window or request a higher quota/tier through the relevant Google controls. For production traffic, combine higher quota with queueing and retry controls.

## FAQ / Related Errors

### Is this always a billing problem?

No. Billing can cause it, but sudden traffic spikes, bursty retries, oversized prompts, and regional capacity constraints can also trigger the same 429 response.

### How long should I wait before retrying?

Start with exponential backoff and jitter. Many transient spikes clear in seconds to minutes, but hard quota exhaustion may require waiting for the quota window reset.

### Related guides

- [Return to the master troubleshooting index](../README.md)
- [Back to the Google AI Studio section](../README.md?id=google-ai-studio)
- [Jump to Developer API & SDKs overview](../README.md?id=developer-api--sdks)
