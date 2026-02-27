---
name: triage-elixir-library
description: Elixir error handling refactoring using the Triage library. Use when working with Elixir code and users mention "triage" library or the `Triage` model. Useful for refactoring or evaluating code using `triage`. This skill provides correct API usage, refactoring patterns, and helps avoid common mistakes when transforming traditional Elixir error handling (with statements, case statements, try/rescue) into Triage's pipeline-based approach.
---

# Triage Refactoring Assistant

Expert guidance for refactoring Elixir code to use the Triage library - a lightweight error handling library that provides pipeline-based result handling with context wrapping and logging.

# Core Triage Concepts

## Result Types

Triage works with standard Elixir results:

- `:ok` and `{:ok, value}`
- `:error` and `{:error, reason}`

Some functions take `{:ok, ...}` / `{:error, ...}` tuples with more than two elements, but the philosophy of Triage is to reduce :ok / :error tuples to two elements as much as possible.

## Key Functions Overview

Best practice is to never `import Triage` but always to refer to `Triage.function`.

### Control Flow:

- `Triage.ok_then!/2` - Handling `:ok` results, allows exceptions to raise
- `Triage.ok_then/2` - Handling `:ok` results, catches exceptions and returns `{:error, Triage.WrappedError.t()}`
- `Triage.ok_then/1` - Executes callback function, catches exceptions and returns `{:error, Triage.WrappedError.t()}`
- `Triage.error_then/2` - error_then `:error` results

VERY IMPORTANT NOTE ABOUT THE `ok_then!/2`, `ok_then/2`, and `error_then/2` FUNCTIONS

The first argument to these functions must always be a result (`:ok`, `:error`, `{:ok, ...}`, `{:error, ...}`)

The second argument is always a function which is given the value/reason of the `:ok`/:error` tuple. If the callback to `ok_then` returns something other than a result, it automatically wraps with `{:ok, _}`. If the callback to `error_then` returns something other than a result, it automaticalyl wraps with `{:error, _}`

```elixir
|> Triage.ok_then!(& &1 + 1)
# If given `:error` result as first argument, the callback isn't executed and the argument is returned
# If given `{:ok, 2}`, returns `{:ok, 3}`

|> Triage.error_then(fn _ -> :operation_failed end)
# If given `:ok` result as first argument, the callback isn't executed and the argument is returned
# If given `{:error, :something}` returns `{:error, :operation_failed}
```

If the callback to `ok_then` functions returns an `:error` result, the `:error` result is returned without wrapping in `{:ok, _}`

If the callback to `error_then` functions returns an `:ok` result, the `:ok` result is returned without wrapping in `{:error, _}`

### Context & Debugging

`Triage.wrap_context/2,3`

Returns a `Triage.WrappedContext.t()` which stores the original error with a context string and/or metadata

```elixir
|> Triage.wrap_context("querying user service")
# or...
|> Triage.wrap_context("querying user service", %{user_id: user_id})
# or...
|> Triage.wrap_context(%{user_id: user_id})
```

`Triage.log/1,2`

Log results with file/line info. Works especially well when given error tuples with `Triage.WrappedError.t()` values.

Second argument default to `:errors`, but can be `:all` to log `:ok` results as well.

`Triage.user_message/1,2`

Takes in an error result (`{:error, _}`). Extracts user-friendly error messages for returning in HTML/JSON/etc...

If a user-friendly error message can't be made from the error result, will log technical details about the error and provide a unique code for the user to pass onto support to find that log line.

### Enumeration

`Triage.map_if/2` - Map with error short-circuiting

`Triage.find_value/2` - Find first success

`Triage.all/1` - Verify all are successful

# Important to know

## `with` clauses

It may be tempting to replace all `with` statements which have `{:ok, _}` clause patterns, but it's not always a good fit.

## Simplifying functions

Sometimes using `triage` can lead to a function being simplified to a single function call.  Often it makes code simpler to remove the function if this happens.

Example:

```elixir
  with {:ok, nouns} <- cue_transactions(transactions),

  # ... further down ...

  defp cue_transactions(transactions) do
    Enum.reduce_while(transactions, [], fn tx, acc ->
      case cue_transaction(tx) do
        {:ok, noun} ->
          {:cont, acc ++ [noun]}

        {:error, :cue_failed, err} ->
          {:halt, {:error, :cue_failed, err}}
      end
    end)
    |> case do
      {:error, {:cue_failed, err}} ->
        {:error, {:cue_failed, err}}

      txs ->
        {:ok, txs}
    end
  end
```

Could be simplified to just:

```elixir
transactions
|> Triage.map_if(&cue_transaction/1)
```

## Handling errors

```elixir
with ... do
   # ...
else
   {:error, {:cue_failed, err}} ->
      {:error, {:invalid_input, err}}

   {:error, :noun_not_a_valid_transaction} ->
      {:error, :noun_not_a_valid_transaction}

   {:error, :not_enough_transactions} ->
      {:error, :not_enough_transactions}
end
```

In order to keep the behavior the same you need to preserve that `MatchError` that might happen on any unspecified errors. So instead of this:

```elixir
|> Triage.error_then(fn
   {:cue_failed, err} -> {:invalid_input, err}
   reason -> reason
end)
```

You would do this:

```elixir
|> Triage.error_then(fn
   {:cue_failed, err} -> {:invalid_input, err}
   :noun_not_a_valid_transaction -> :noun_not_a_valid_transaction
   :not_enough_transactions -> :not_enough_transactions
end)
```

## Critical Rules to Remember

When `ok_then` / `ok_then!` receives `:ok` as an error result, it passes `nil` to the callback.

When `error_then` receives `:error` as an error result, it passes `nil` to the callback.

Don't always use `ok_then` (not `ok_then!`) when function might raise. Sometimes we want to raise an exception if the process should crash in that situation. Default to `ok_then!` functions, especially when refactoring as it's the same behavior as before.

Context is cumulative: Each `wrap_context` adds to the error context chain which adds details for `Triage.log` and `Triage.user_message`.

Be careful about using `wrap_context`: refactoring generally needs to be done at a higher level to support the `Triage.WrapError.t()` value.

## Verification Steps

After refactoring:

1. Ensure all result types are standard (`:ok`, `{:ok, ...}`, `:error`, `{:error, ...}`)
2. Verify exception handling requirements (use `ok_then` vs `ok_then!` appropriately)
3. Test error paths to ensure proper propagation
4. Check for double-wrapping anti-pattern

