# Deep Research Agent on Supabase Edge Functions

A Research Agent powered by Resonate and OpenAI, running on Supabase Edge Functions. The Research Agent is a distributed, recursive agent that breaks a research topic into subtopics, researches each subtopic recursively, and synthesizes the results.

![Deep Research Agent Demo](doc/research-agent.jpeg)

## How It Works

This example demonstrates how complex, distributed agentic applications can be implemented with simple code in Resonate's Distributed Async Await: The research agent is a recursive generator function that breaks down topics into subtopics and invokes itself for each subtopic:

```typescript
function* research(ctx, topic, depth) {
  const messages = [
    { role: "system", content: "Break topics into subtopics..." },
    { role: "user", content: `Research ${topic}` }
  ];

  while (true) {
    // Ask the LLM about the topic
    const response = yield* ctx.run(prompt, messages, ...);
    messages.push(response);

    // If LLM wants to research subtopics...
    if (response.tool_calls) {
      const handles = [];

      // Spawn parallel research for each subtopic
      for (const tool_call of response.tool_calls) {
        const subtopic = ...;
        const handle = yield* ctx.beginRpc(research, subtopic, depth - 1);
        handles.push([tool_call, handle]);
      }

      // Wait for all subtopic results
      for (const [tool_call, handle] of handles) {
        const result = yield* handle;
        messages.push({ role: "tool", ..., content: result });
      }
    } else {
      // LLM provided final summary
      return response.content;
    }
  }
}
```

The following video visualizes how this recursive pattern creates a dynamic call graph, spawning parallel research branches that fan out as topics are decomposed, then fan back in as results are synthesized:

https://github.com/user-attachments/assets/cf466675-def3-4226-9233-a680cd7e9ecb

**Key concepts:**
- **Concurrent Execution**: Multiple subtopics are researched concurrently via `ctx.beginRpc`
- **Coordination**: Handles are collected first, then awaited together (fork/join, fan-out/fan-in)
- **Depth control**: Recursion stops when `depth` reaches 0

---

# Running the Example

You can run the Deep Research Agent locally on your machine with [Supabase CLI](https://supabase.com/docs/guides/local-development/cli/getting-started?queryGroups=platform&platform=macos) or you can deploy the agent to Supabase Platform.

## 1. Running Locally

### 1.1. Prerequisites

Install the Resonate Server & CLI with [Homebrew](https://docs.resonatehq.io/operate/run-server#install-with-homebrew) or download the latest release from [Github](https://github.com/resonatehq/resonate/releases).

```
brew install resonatehq/tap/resonate
```

Install the [Supabase CLI](https://supabase.com/docs/guides/local-development/cli/getting-started?queryGroups=platform&platform=macos)


To run this project you also need an [OpenAI API Key](https://platform.openai.com) and export the key as an environment variable

```
export OPENAI_API_KEY="sk-..."
```

### 1.2. Start Supabase locally

```
supabase start
```

### 1.3. Setup the Countdown

Clone the repository

```
git clone https://github.com/resonatehq-examples/example-openai-deep-research-agent-supabase-ts
cd example-openai-deep-research-agent-supabase-ts
```

Install dependencies

```
npm install
```

### 1.4. Serve the Countdown function

```
supabase functions serve deep-research-agent
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


### 1.6. Invoke the Deep Research Agent

Start a research task

```
resonate invoke <promise-id> --func research --arg <topic> --arg <depth> --target <function-url>
```

Example

```
resonate invoke research.1 --func research --arg "What are distributed systems" --arg 1 --target http://127.0.0.1:54321/functions/v1/deep-research-agent
```

### 1.7. Inspect the execution

Use the `resonate tree` command to visualize the research execution.

```
resonate tree research.1
```


```
research.1
├── research.1.0 🟢 (run)
├── research.1.1 🟡 (rpc research)
│   └── research.1.1.0 🟡 (run)
├── research.1.2 🟡 (rpc research)
│   └── research.1.2.0 🟡 (run)
└── research.1.3 🟡 (rpc research)
    └── research.1.3.0 🟡 (run)
```

#### Supabase

Install the [Supabase CLI](https://supabase.com/docs/guides/local-development/cli/getting-started?queryGroups=platform&platform=macos)

### 2.1 Deploy your function

```
supabase functions deploy deep-research-agent --project-ref <PROJECT_ID>
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

### 2.4 Invoke the Deep Research Agent

Start a research task

```
resonate invoke <promise-id> --func research --arg <topic> --arg <depth> --target <function-url>
```

Example

```
resonate invoke research.1 --func research --arg "What are distributed systems" --arg 1 --target https://wryfyvstwcwpjaulrdmx.supabase.co/functions/v1/deep-research-agent
```

### 2.5. Inspect the execution

Use the `resonate tree` command to visualize the countdown execution.

```
resonate tree research.1
```

Example output (while waiting on the second sleep):

```
research.1
├── research.1.0 🟢 (run)
├── research.1.1 🟢 (rpc research)
│   └── research.1.1.0 🟢 (run)
├── research.1.2 🟢 (rpc research)
│   └── research.1.2.0 🟢 (run)
├── research.1.3 🟡 (rpc research)
├── research.1.4 🟢 (rpc research)
│   └── research.1.4.0 🟢 (run)
└── research.1.5 🟢 (rpc research)
    └── research.1.5.0 🟢 (run)
```

## Troubleshooting
If you are still having trouble please [open an issue](https://github.com/resonatehq-examples/example-openai-deep-research-agent-supabase-ts/issues).
