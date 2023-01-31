## CloudFlare queues

### Prerequisites

- Install [Node.js](https://nodejs.org/en/download/)
- Install [Wrangler](https://developers.cloudflare.com/workers/wrangler/install-and-update/)
- Enable Queues in the CloudFlare dashboard.
  - Dashboard > Workers > Queues
  - Enable Queues Beta
- Use wrangler to login to CloudFlare: `wrangler login`

### Create a queue

1. Create the CloudFlare queue:

   `wrangler queues create my-messages`

### Create a producer worker

1. In the root folder, create a worker to produce messages:

    `wrangler init producer`

   1. Create package.json: `Y`
   2. Use TypeScript: `Y`
   3. Create worker: `Fetch handler`
   4. Write tests: `N`

2. Update the *producer/src/index.ts* file with:

```typescript
export interface Env {
   QUEUE_BINDING: Queue;
}

export default {
  async fetch(
      request: Request,
      env: Env,
      ctx: ExecutionContext
   ): Promise<Response> {
      let message = Math.random();
      await env.QUEUE_BINDING.send(message);
      return new Response("Success!");
   },
};
```

3. Add the following lines to the *producer/wrangler.toml* file:

```toml
[[queues.producers]]
 queue = "my-messages" 
 binding = "QUEUE_BINDING"
```

4. Publish the *producer* worker:

   `cd producer`

   `wrangler publish`

5. Trigger the *producer* endpoint:

   `curl https://producer.<your-worker>.workers.dev`

6. There should be one message in the queue backlog.


### Create a consumer worker

1. In the root folder, create a worker to produce messages:

    `wrangler init consumer`

   1. Create package.json: `Y`
   2. Use TypeScript: `Y`
   3. Create worker: `Fetch handler`
   4. Write tests: `N`

2. Update the *consumer/src/index.ts* file to:

```typescript
export default {
   async queue(
      batch: MessageBatch<Error>,
      env: Env
   ): Promise<void> {
      let messages = JSON.stringify(batch.messages);
      console.log(`${messages}`);
   },
};
```

3. Add the following lines to the *consumer/wrangler.toml* file:

```toml
[[queues.consumers]]
 queue = "my-messages"
 max_batch_size = 1
```

4. Publish the *consumer* worker:

   `cd consumer`

   `wrangler publish`

5. Start a tail to read the log of the consumer worker:

   `wrangler tail --format=json`

6. Start the *producer* worker to send messages to the queue:

   `curl https://producer.<your-worker>.workers.dev`

7. The *consumer* worker should receive the message and log it.