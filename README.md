# Autotranscribe

You upload an audio file and get a cached transcription.

This is a microservice approach built on [the Cloudflare Stack](https://developers.cloudflare.com) that shares uploaded audio and cached transcriptions. It uses a [WorkerEntryPoint for RPC](https://developers.cloudflare.com/workers/runtime-apis/bindings/service-bindings/rpc/) to connected service bindings.

This service exposes the following methods:

- `listUploads()` - List available transcribed uploads
- `get(key)` - Retrieve a specific upload URL and metadata
- `delete(key)` - Delete an existing transcription

After it is deployed, you can use it in an existing Worker like so:

```toml
[[services]]
binding = "UPLOADER"
service = "autotranscriber"
entrypoint = "Uploader"
```

```typescript
const uploads = await env.UPLOADER.listUploads();
```

Transcripts are auto generated by adding to the `uploads` bucket created below.

## Setup

We are going to provision an [R2](https://developers.cloudflare.com/r2) bucket, a [Queue](https://developers.cloudflare.com/queues), and a [KV Namespace](https://developers.cloudflare.com/kv) to cache our results from the [Whisper model](https://developers.cloudflare.com/workers-ai/models/whisper-large-v3-turbo/) hosted on [Workers AI](https://developers.cloudflare.com/workers-ai).

```bash
# Create a new object storage R2 bucket named "uploads"
npx wrangler r2 bucket create uploads
```

```bash
# This will enable the bucket to serve the file
npx wrangler r2 bucket dev-url enable uploads
# ✨ Public access enabled at 'https://pub-your-url-here.r2.dev'.
```

Modify your [wrangler.json](./wrangler.json) with your bucket id.

**NOTE**: You can edit your [CORS Headers](https://developers.cloudflare.com/r2/buckets/cors/) if you want to access this from another application

```bash
# Create a new Queue named "uploaded"
npx wrangler queues create uploaded
```

```bash
# Get notified of creations and deletions to "uploads" bucket via the "uploaded" Queue
npx wrangler r2 bucket notification create uploads --event-type object-create --event-type object-delete --queue uploaded
```

```bash
# Create a KV store where you can cache your results
npx wrangler kv namespace create uploaded-metadata
```
## Develop

```bash
npm run dev
```

If you want Type completion in your worker:

```typescript
interface Env {
	UPLOADER: Fetcher<import("../autotranscriber/src/").Uploader>;
}
```

## Deploy

```bash
npm run deploy
```
