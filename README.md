# Resonate Countdown on Supabase Edge Functions

A *Countdown* powered by the Resonate Typescript SDK and Supabase Edge Functions. The countdown sends periodic notifications to [ntfy.sh](https://ntfy.sh/) at configurable intervals.

## Behind the Scenes

The Countdown is implemented with Resonate's Durable Execution framework, Distributed Async Await. The Countdown is a simple loop that can sleep for hours, days, or weeks. On `yield ctx.sleep` the countdown function suspends (terminates), immediately completing the Supabase Edge Function execution. After the specified delay, Resonate will resume (restart) the countdown function by triggering a new Supabase Edge Function execution.

```typescript
export function* countdown(
	ctx: Context,
	count: number,
	delay: number,
	url: string,
) {
	for (let i = count; i > 0; i--) {
		// send notification to ntfy.sh
		yield* ctx.run(notify, url, `Countdown: ${i}`);
		// sleep creates a suspension point causing
		// the Supabase Edge Function execution to terminate
		yield* ctx.sleep(delay * 60 * 1000);
	}
	// send the last notification to ntfy.sh
	yield* ctx.run(notify, url, `Done`);
}
```

**Key Concepts:**

- **Suspension and Resumption:** Executions can be suspended for any amount of time
- **Stateful executions on stateless infrastructure:** Short-lived function instances executing one step of a long-lived execution coordinated by the Resonate Server.

![Deep Research Agent Demo](doc/mechanics.jpg)

---

# Running the Example

You can run the Countdown locally on your machine with Supabase CLI or you can deploy the Countdown to Supabase Platform.

## 1. Running Locally

### 1.1. Prerequisites

Install the Resonate Server & CLI with [Homebrew](https://docs.resonatehq.io/operate/run-server#install-with-homebrew) or download the latest release from [Github](https://github.com/resonatehq/resonate/releases).

```
brew install resonatehq/tap/resonate
```

Install the [Supabase CLI](https://supabase.com/docs/guides/local-development/cli/getting-started?queryGroups=platform&platform=macos)

### 1.2. Start Supabase locally

```
supabase start
```

### 1.3. Setup the Countdown

Clone the repository

```
git clone https://github.com/resonatehq-examples/example-countdown-supabase-ts
cd example-countdown-supabase-ts
```

Install dependencies

```
npm install
```

### 1.4. Serve the Countdown function

```
supabase functions serve countdown
```

### 1.5 Start the resonate worker behind ngrok

```
ngrok http 8001
```

```
resonate dev --system-url  <ngrok-url>
```

Example

```
resonate dev --system-url  https://583ef7749990.ngrok-free.app
```

### 1.5. Invoke the Countdown

The examples use ntfy.sh to send notifications. Create a unique channel name (to avoid receiving notifications from other users) and open the ntfy.sh channel in your browser.

```
echo https://ntfy.sh/resonatehq-$RANDOM
```

Start a countdown

```
resonate invoke <promise-id> --func countdown --arg <count> --arg <delay-in-minutes> --arg https://ntfy.sh/<channel> --target <function-url>
```

Example

```
resonate invoke countdown.1 --func countdown --arg 5 --arg 1 --arg https://ntfy.sh/resonatehq-17905 --target http://127.0.0.1:54321/functions/v1/countdown
```

### 1.6. Inspect the execution

Use the `resonate tree` command to visualize the countdown execution.

```
resonate tree countdown.1
```

Example output (while waiting on the second sleep):

```
countdown.1
├── countdown.1.0 🟢 (run)
├── countdown.1.1 🟢 (sleep)
├── countdown.1.2 🟢 (run)
└── countdown.1.3 🟡 (sleep)
```

## 2. Deploying to Supabase

This section guides you through deploying the countdown example to Supabase Platform using Workers for the countdown function.

### 2.1 Prerequisites

#### Resonate

Install the Resonate Server & CLI with [Homebrew](https://docs.resonatehq.io/operate/run-server#install-with-homebrew) or download the latest release from [Github](https://github.com/resonatehq/resonate/releases).

```
brew install resonatehq/tap/resonate
```

#### Supabase

Install the [Supabase CLI](https://supabase.com/docs/guides/local-development/cli/getting-started?queryGroups=platform&platform=macos)

### 2.1 Deploy your function

```
supabase functions deploy countdown --project-ref <PROJECT_ID>
```

### 2.2 Start the resonate worker behind ngrok

```
ngrok http 8001
```

```
resonate dev --system-url  <ngrok-url>
```

Example

```
resonate dev --system-url  https://583ef7749990.ngrok-free.app
```

### 2.4 Invoke the Countdown

The examples use ntfy.sh to send notifications. Create a unique channel name (to avoid receiving notifications from other users) and open the ntfy.sh channel in your browser.

```
echo https://ntfy.sh/resonatehq-$RANDOM
```

Start a countdown

```
resonate invoke <promise-id> --func countdown --arg <count> --arg <delay-in-minutes> --arg https://ntfy.sh/<channel> --target $FUNCTION_URL
```

Example

```
resonate invoke countdown.1 --func countdown --arg 5 --arg 1 --arg https://ntfy.sh/resonatehq-17905 --target https://<orgID>.supabase.co/functions/v1/countdown
```

### 2.5. Inspect the execution

Use the `resonate tree` command to visualize the countdown execution.

```
resonate tree countdown.1
```

Example output (while waiting on the second sleep):

```
countdown.1
├── countdown.1.0 🟢 (run)
├── countdown.1.1 🟢 (sleep)
├── countdown.1.2 🟢 (run)
└── countdown.1.3 🟡 (sleep)
```

## Troubleshooting

If everything is configured correctly, you will see notifications in your ntfy.sh workspace.

![ntfy logo](doc/ntfy.png)

If you are still having trouble please [open an issue](https://github.com/resonatehq-examples/example-countdown-supabase-ts/issues).
