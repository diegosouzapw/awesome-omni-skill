---
name: ash-ai
description: AshAi extension guidelines for integrating AI capabilities with Ash Framework. Use when implementing vectorization/embeddings, exposing Ash actions as LLM tools, creating prompt-backed actions, or setting up MCP servers. Covers semantic search, LangChain integration, and structured outputs.
---

# AshAi Guidelines

AshAi integrates AI capabilities with Ash resources: vectorization, LLM tools, prompt-backed actions, and MCP servers.

## Vectorization

Convert text to vector embeddings for semantic search:

```elixir
defmodule MyApp.Artist do
  use Ash.Resource, extensions: [AshAi]

  vectorize do
    full_text do
      text(fn record ->
        """
        Name: #{record.name}
        Biography: #{record.biography}
        """
      end)
      used_attributes [:name, :biography]  # Only rebuild when these change
    end

    strategy :ash_oban  # or :after_action, :manual
    embedding_model MyApp.OpenAiEmbeddingModel
  end
end
```

### Embedding Model

```elixir
defmodule MyApp.OpenAiEmbeddingModel do
  use AshAi.EmbeddingModel

  @impl true
  def dimensions(_opts), do: 3072

  @impl true
  def generate(texts, _opts) do
    response = Req.post!("https://api.openai.com/v1/embeddings",
      json: %{"input" => texts, "model" => "text-embedding-3-large"},
      headers: [{"Authorization", "Bearer #{System.fetch_env!("OPEN_AI_API_KEY")}"}]
    )

    case response.status do
      200 -> {:ok, Enum.map(response.body["data"], & &1["embedding"])}
      _ -> {:error, response.body}
    end
  end
end
```

### Vectorization Strategies

| Strategy | Behavior |
|----------|----------|
| `:after_action` | Sync after create/update (slow, not for production) |
| `:ash_oban` | Async via Ash Oban (recommended) |
| `:manual` | You control when to update |

### Semantic Search Action

```elixir
read :semantic_search do
  argument :query, :string, allow_nil?: false

  prepare before_action(fn query, _context ->
    case MyApp.OpenAiEmbeddingModel.generate([query.arguments.query], []) do
      {:ok, [search_vector]} ->
        query
        |> Ash.Query.filter(vector_cosine_distance(full_text_vector, ^search_vector) < 0.5)
        |> Ash.Query.sort(asc: vector_cosine_distance(full_text_vector, ^search_vector))
      {:error, error} -> {:error, error}
    end
  end)
end
```

### Authorization Bypass

```elixir
bypass action(:ash_ai_update_embeddings) do
  authorize_if AshAi.Checks.ActorIsAshAi
end
```

## AI Tools

Expose Ash actions as LLM tools:

```elixir
defmodule MyApp.Blog do
  use Ash.Domain, extensions: [AshAi]

  tools do
    tool :read_posts, MyApp.Blog.Post, :read do
      description "customize the tool description"
    end
    tool :create_post, MyApp.Blog.Post, :create
    tool :read_with_details, MyApp.Blog.Post, :read do
      load [:author, :internal_notes]  # Include private attrs when loaded
    end
  end
end
```

### Tool Data Access

- **Filtering/Sorting**: Only `public?: true` attributes
- **Arguments**: Only `public?: true` action arguments
- **Response**: Public attributes by default
- **`load` option**: Can include private attributes in response

### LangChain Integration

```elixir
chain =
  %{llm: LangChain.ChatModels.ChatOpenAI.new!(%{model: "gpt-4o"}), verbose: true}
  |> LangChain.Chains.LLMChain.new!()
  |> AshAi.setup_ash_ai(otp_app: :my_app, tools: [:read_posts, :create_post])
```

## Prompt-Backed Actions

Create actions where LLM provides the implementation:

```elixir
action :analyze_sentiment, :atom do
  constraints one_of: [:positive, :negative]
  description "Analyzes sentiment of text"

  argument :text, :string, allow_nil?: false

  run prompt(
    LangChain.ChatModels.ChatOpenAI.new!(%{model: "gpt-4o"}),
    tools: true  # or [:specific, :tools] or false
  )
end
```

### Structured Outputs with Custom Types

```elixir
defmodule JobListing do
  use Ash.TypedStruct

  typed_struct do
    field :title, :string, allow_nil?: false
    field :company, :string, allow_nil?: false
    field :requirements, {:array, :string}
  end
end

action :parse_job, JobListing do
  argument :raw_content, :string, allow_nil?: false

  run prompt(
    fn _input, _context ->
      LangChain.ChatModels.ChatOpenAI.new!(%{
        model: "gpt-4o-mini",
        api_key: System.get_env("OPENAI_API_KEY")
      })
    end,
    prompt: "Parse this job listing: <%= @input.arguments.raw_content %>",
    tools: false
  )
end
```

For detailed prompt format options and adapters, see [references/prompt-actions.md](references/prompt-actions.md).

## MCP Server

### Development

```elixir
# In endpoint.ex
if code_reloading? do
  plug AshAi.Mcp.Dev,
    protocol_version_statement: "2024-11-05",
    otp_app: :your_app
end
```

### Production

```elixir
pipeline :mcp do
  plug AshAuthentication.Strategy.ApiKey.Plug,
    resource: YourApp.Accounts.User,
    required?: false
end

scope "/mcp" do
  pipe_through :mcp

  forward "/", AshAi.Mcp.Router,
    tools: [:read_posts, :create_post, :analyze_sentiment],
    protocol_version_statement: "2024-11-05",
    otp_app: :my_app
end
```

## Testing

- Mock embedding model responses for consistent results
- Test vector search with known embeddings
- Use deterministic test models for prompt-backed actions
- Verify tool access and permissions
