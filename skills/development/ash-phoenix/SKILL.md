---
name: ash-phoenix
description: AshPhoenix integration guidelines for using Ash Framework with Phoenix. Use when working with AshPhoenix.Form, creating forms backed by Ash resources, handling nested forms, union types in forms, or integrating Ash actions with Phoenix LiveViews. Covers form creation, validation, submission, and error handling patterns.
---

# AshPhoenix Guidelines

AshPhoenix integrates Ash Framework with Phoenix, providing `AshPhoenix.Form` for forms backed by Ash resources.

## Creating Forms

```elixir
# For creating a new resource
form = AshPhoenix.Form.for_create(MyApp.Blog.Post, :create) |> to_form()

# For updating an existing resource
post = MyApp.Blog.get_post!(post_id)
form = AshPhoenix.Form.for_update(post, :update) |> to_form()

# With initial values
form = AshPhoenix.Form.for_create(MyApp.Blog.Post, :create,
  params: %{title: "Draft Title"}
) |> to_form()
```

## Code Interface Forms

Add the `AshPhoenix` extension to domains for `form_to_*` functions:

```elixir
# In domain
use Ash.Domain,
  extensions: [AshPhoenix]

resources do
  resource MyApp.Accounts.User do
    define :register_with_password, args: [:email, :password]
  end
end

# Usage - generates form_to_register_with_password
MyApp.Accounts.form_to_register_with_password(...opts)
```

### Positional Arguments in Forms

By default, `args` from `define` are ignored for forms. Configure in `forms` section:

```elixir
forms do
  form :register_with_password, args: [:email]
end

# Usage
MyApp.Accounts.form_to_register_with_password(email, ...)
```

Use positional arguments for values that shouldn't be editable in the form (e.g., `user_id` on a user-specific page).

## Form Validation & Submission

```elixir
def handle_event("validate", %{"form" => params}, socket) do
  form = AshPhoenix.Form.validate(socket.assigns.form, params)
  {:noreply, assign(socket, :form, form)}
end

def handle_event("submit", %{"form" => params}, socket) do
  case AshPhoenix.Form.submit(socket.assigns.form, params: params) do
    {:ok, post} ->
      socket =
        socket
        |> put_flash(:info, "Post created successfully")
        |> push_navigate(to: ~p"/posts/#{post.id}")
      {:noreply, socket}

    {:error, form} ->
      {:noreply, assign(socket, :form, form)}
  end
end
```

## Nested Forms

If your action has `manage_relationship`, AshPhoenix automatically infers nested forms:

```elixir
# In resource
create :create do
  accept [:name]
  argument :locations, {:array, :map}
  change manage_relationship(:locations, type: :create)
end
```

```heex
<.simple_form for={@form} phx-change="validate" phx-submit="submit">
  <.input field={@form[:name]} />

  <.inputs_for :let={location} field={@form[:locations]}>
    <.input field={location[:name]} />
  </.inputs_for>
</.simple_form>
```

### Adding Nested Forms

```heex
<.button type="button" phx-click="add-form" phx-value-path={@form.name <> "[locations]"}>
  <.icon name="hero-plus" />
</.button>
```

```elixir
def handle_event("add-form", %{"path" => path}, socket) do
  form = AshPhoenix.Form.add_form(socket.assigns.form, path)
  {:noreply, assign(socket, :form, form)}
end
```

### Removing Nested Forms

```heex
<.button type="button" phx-click="remove-form" phx-value-path={location.name}>
  <.icon name="hero-x-mark" />
</.button>
```

```elixir
def handle_event("remove-form", %{"path" => path}, socket) do
  form = AshPhoenix.Form.remove_form(socket.assigns.form, path)
  {:noreply, assign(socket, :form, form)}
end
```

## Union Forms

For union types with different inputs per type:

```heex
<.inputs_for :let={fc} field={@form[:content]}>
  <.input
    field={fc[:_union_type]}
    phx-change="type-changed"
    type="select"
    options={[Normal: "normal", Special: "special"]}
  />

  <%= case fc.params["_union_type"] do %>
    <% "normal" -> %>
      <.input type="text" field={fc[:body]} />
    <% "special" -> %>
      <.input type="text" field={fc[:text]} />
  <% end %>
</.inputs_for>
```

```elixir
def handle_event("type-changed", %{"_target" => path} = params, socket) do
  new_type = get_in(params, path)
  path = :lists.droplast(path)

  form =
    socket.assigns.form
    |> AshPhoenix.Form.remove_form(path)
    |> AshPhoenix.Form.add_form(path, params: %{"_union_type" => new_type})

  {:noreply, assign(socket, :form, form)}
end
```

## Debugging Form Errors

Errors only display when they implement `AshPhoenix.FormData.Error` protocol and have `field`/`fields` set.

```elixir
# See ALL errors (including ones not shown in UI)
AshPhoenix.Form.raw_errors(form, for_path: :all)

# See errors that will be displayed (implement protocol + have fields)
AshPhoenix.Form.errors(form, for_path: :all)
```

For action errors not tied to fields, display with flash messages or notices at form top/bottom.

## Best Practices

1. **Let the Resource guide the UI** - Well-defined resources with validations make AshPhoenix more effective
2. **Use code interfaces** - Define on domains for clean, consistent API
3. **Load before editing** - Use `Ash.load!/2` to load all required relationships before creating update forms
