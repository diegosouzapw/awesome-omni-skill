---
name: live-tests
description: Writes live integration tests that hit the real Copilot API and record responses as replayable fixtures. Use this skill when adding new agent behaviors, provider integrations, or tool interactions that need real-world API coverage.
globs: "{test/**,test/support/fixtures/**}"
---

# Live Test & Fixture Recording Skill

You write live integration tests that exercise the real GitHub Copilot API and capture responses as JSON fixtures for deterministic replay. This is the project's VCR-like system for ensuring tests stay grounded in real API behavior.

## Architecture overview

```
Live test (mix test --include live --include save_fixtures)
  │
  ├─ RecordingProvider  ← wraps real Copilot provider, captures SSE events
  │     │
  │     └─ persistent_term storage ← events buffered here during stream
  │
  └─ FixtureHelper.save_fixture/2  ← writes events to JSON file
        │
        └─ test/support/fixtures/<name>.json

Fixture-based test (mix test)
  │
  ├─ FixtureProvider  ← reads fixture JSON, replays events via Req.Response.Async
  │     │
  │     └─ FixtureHelper.build_fixture_response/1 ← spawns process to send SSE events
  │
  └─ Assertions on agent behavior, events, tool calls, etc.
```

## When to act

- Adding a new agent behavior that needs a new fixture (e.g., new tool call pattern, multi-turn conversation, error handling).
- Adding or changing provider integration logic.
- When the user asks to "record a fixture", "add a live test", or "capture API responses".
- When an existing fixture is stale and needs re-recording.

## Writing a live test

Live tests go in `test/opal/live_test.exs`. They require `@moduletag :live` (excluded by default) and use the `RecordingProvider` to capture real SSE events.

### Template

```elixir
describe "live API — <scenario description>" do
  @tag :save_fixtures
  @tag timeout: 30_000
  test "records <what this captures>" do
    RecordingProvider.start_recording()

    {:ok, pid} =
      Opal.start_session(%{
        model: {:copilot, "claude-sonnet-4"},
        system_prompt: "<constrained prompt that produces deterministic output>",
        tools: [<tool modules if needed>],
        working_dir: System.tmp_dir!(),
        provider: RecordingProvider
      })

    {:ok, response} = Opal.prompt_sync(pid, "<user message>", 25_000)
    Opal.stop_session(pid)

    events = RecordingProvider.stop_recording()
    assert length(events) > 0

    # Save as a fixture for replay
    path = FixtureHelper.save_fixture("<descriptive_name>.json", events)
    assert File.exists?(path)

    # Assertions on the live response
    assert is_binary(response)
    assert String.contains?(response, "<expected content>")
  end
end
```

### Running

```bash
# Run all live tests (requires valid Copilot auth)
mix test --include live

# Run live tests AND save recorded fixtures to disk
mix test --include live --include save_fixtures

# Run a specific live test
mix test --include live test/opal/live_test.exs:<line_number>
```

## Writing a fixture-based integration test

Once you have a fixture, write a deterministic integration test that replays it. These go in `test/opal/integration_test.exs` or a new file under `test/opal/`.

### Template

```elixir
describe "<feature under test>" do
  test "<what it verifies>" do
    # Configure the FixtureProvider with your fixture
    :persistent_term.put({FixtureProvider, :fixture}, "<your_fixture>.json")

    # For multi-turn tests (tool call → second response):
    # :persistent_term.put({FixtureProvider, :second_fixture}, "<second_turn>.json")

    session_id = "test-#{System.unique_integer([:positive])}"
    {:ok, tool_sup} = Task.Supervisor.start_link()

    agent_opts = [
      session_id: session_id,
      model: Model.new(:copilot, "test-model"),
      system_prompt: "<system prompt>",
      tools: [<tool modules>],
      working_dir: System.tmp_dir!(),
      provider: FixtureProvider,
      tool_supervisor: tool_sup
    ]

    {:ok, pid} = Agent.start_link(agent_opts)
    Events.subscribe(session_id)

    Agent.prompt(pid, "<user message>")

    # Assert on events received
    assert_receive {:event, %{type: :response_start}}, 5_000
    assert_receive {:event, %{type: :text_delta, data: %{delta: delta}}}, 5_000
    assert is_binary(delta)
    assert_receive {:event, %{type: :response_end}}, 5_000
  end
end
```

## Fixture file format

Fixtures live in `test/support/fixtures/` as JSON:

```json
{
  "description": "Recorded live fixture: responses_api_text.json",
  "recorded_at": "2025-02-13T...",
  "events": [
    {"data": "{\"type\":\"response.created\",\"response\":{...}}"},
    {"data": "{\"type\":\"response.output_item.added\",...}"},
    {"data": "{\"type\":\"response.completed\",...}"}
  ]
}
```

Each `data` field contains one SSE event payload as a JSON string.

## Rules

1. **System prompts in live tests must be highly constrained** to produce deterministic output. Use prompts like "Respond with exactly the word 'pong'" rather than open-ended prompts.
2. **Name fixtures descriptively**: `responses_api_tool_call.json`, not `test1.json`.
3. **Never hand-edit fixture JSON.** Always re-record from a live test.
4. **Tag live tests correctly**: `@moduletag :live` on the module, `@tag :save_fixtures` on recording tests, `@tag timeout: 30_000` for API calls.
5. **Clean up after recording tests** if the fixture is only meant for one-time capture — or keep it permanently if it will be used by replay tests.
6. **Fixture-based tests should be fast and async-safe** — they replay from disk, no network needed.
7. When adding a fixture for a new scenario (e.g., error response, high token usage), also add a corresponding integration test that replays it.

## Key files

- `test/opal/live_test.exs` — Live API tests with recording
- `test/opal/integration_test.exs` — Fixture-based integration tests
- `test/support/fixture_helper.ex` — Fixture load/save/replay helpers
- `test/support/fixtures/` — Recorded fixture JSON files
