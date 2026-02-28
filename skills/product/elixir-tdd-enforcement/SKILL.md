---
name: elixir-tdd-enforcement
description: MANDATORY for ANY feature or bugfix - write ExUnit test FIRST, watch it FAIL, then implement. NO exceptions. Use before writing any Elixir production code.
---

# Elixir TDD Enforcement: The Iron Law

## THE IRON LAW

**NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST**

Not sometimes. Not usually. ALWAYS.

If you write production code before a failing test, DELETE IT and start over.

## WHEN THIS SKILL APPLIES

- Implementing ANY new function
- Fixing ANY bug
- Adding ANY feature
- Modifying ANY behavior
- Refactoring ANY code

**If you're changing `.ex` files in `lib/`, this skill is MANDATORY.**

## THE RED-GREEN-REFACTOR CYCLE

### Phase 1: RED (Write Failing Test)

1. **Write ONE minimal ExUnit test**
   ```elixir
   test "creates user with valid attrs" do
     attrs = %{name: "Alice", email: "alice@example.com"}
     assert {:ok, %User{} = user} = Accounts.create_user(attrs)
     assert user.name == "Alice"
     assert user.email == "alice@example.com"
   end
   ```

2. **Run the test**
   ```bash
   mix test test/my_app/accounts_test.exs:42
   ```

3. **VERIFY IT FAILS FOR THE RIGHT REASON**
   - Read the error message
   - Confirm it's failing because functionality doesn't exist
   - NOT because of syntax errors or wrong test setup

**CHECKPOINT: If test doesn't fail, delete it and write a different test.**

### Phase 2: GREEN (Minimal Implementation)

1. **Write SIMPLEST code to pass the test**
   ```elixir
   def create_user(attrs) do
     %User{}
     |> User.changeset(attrs)
     |> Repo.insert()
   end
   ```

2. **Run the test again**
   ```bash
   mix test test/my_app/accounts_test.exs:42
   ```

3. **VERIFY IT PASSES**
   - Read the actual output
   - See the green dot or "1 test, 0 failures"
   - NOT just assume it works

**CHECKPOINT: If test doesn't pass, fix implementation (not test).**

### Phase 3: REFACTOR (Improve While Green)

1. **Improve code quality**
   - Extract functions
   - Improve names
   - Add pattern matching

2. **Run tests after EACH change**
   ```bash
   mix test
   ```

3. **Stay GREEN**
   - If tests fail during refactor, undo
   - Only refactor when all tests pass

**CHECKPOINT: Tests must stay green throughout refactoring.**

## VERIFICATION CHECKLIST

Before claiming you're done, verify:

- [ ] I wrote the test BEFORE any implementation code
- [ ] I watched the test FAIL for the right reason
- [ ] I read the actual failure message
- [ ] I implemented only enough code to pass the test
- [ ] I ran the test again and saw it PASS
- [ ] I read the actual success message
- [ ] All other tests still pass
- [ ] I refactored only while tests were green

**If you can't check ALL boxes, you didn't follow TDD.**

## COMMON VIOLATIONS AND RESPONSES

### Violation: "I'll just write the code, then write the test"
**Response:** NO. Delete the code. Write test first.

### Violation: "The function is simple, I don't need to see it fail"
**Response:** WRONG. Even simple code needs failing tests. Write test, watch fail.

### Violation: "I already know what the test will look like"
**Response:** Irrelevant. Write it first anyway.

### Violation: "I wrote the test and implementation together"
**Response:** Delete both. Write test, watch fail, then implement.

### Violation: "The test passed on first run"
**Response:** RED FLAG. Test might not be testing anything. Review test.

### Violation: "I'm just refactoring, I don't need new tests"
**Response:** Correct - but ALL existing tests must stay GREEN.

## ELIXIR-SPECIFIC TEST PATTERNS

### Testing Context Functions
```elixir
# RED: Write test first
test "list_users/0 returns all users" do
  user1 = fixture(:user)
  user2 = fixture(:user)
  users = Accounts.list_users()
  assert length(users) == 2
  assert user1 in users
  assert user2 in users
end

# Run test → watch it fail (function doesn't exist)

# GREEN: Implement
def list_users do
  Repo.all(User)
end

# Run test → watch it pass
```

### Testing Changesets
```elixir
# RED: Write test for validation
test "changeset with invalid email" do
  changeset = User.changeset(%User{}, %{email: "invalid"})
  refute changeset.valid?
  assert %{email: ["invalid format"]} = errors_on(changeset)
end

# Run test → watch it fail

# GREEN: Add validation
def changeset(user, attrs) do
  user
  |> cast(attrs, [:email])
  |> validate_format(:email, ~r/@/)
end
```

### Testing Phoenix Controllers
```elixir
# RED: Write test
test "GET /users returns 200", %{conn: conn} do
  conn = get(conn, ~p"/users")
  assert html_response(conn, 200)
end

# Run test → watch it fail (route doesn't exist)

# GREEN: Add route and controller action
```

## DIALYZER ERRORS: SPECIAL CASE

**If Dialyzer reports an error:**

1. **Write a test that exercises the problematic code**
2. **Make sure test passes** (proving code works)
3. **Add @spec to guide Dialyzer**
4. **Run `mix dialyzer` to verify**

**NEVER:**
- Add to dialyzer.ignore
- Modify dialyzer PLT to suppress
- Comment out the code

**The test proves it works. The spec helps Dialyzer understand.**

## CREDO WARNINGS: SPECIAL CASE

**If Credo reports a warning:**

1. **Understand WHY it's warning**
2. **Fix the actual issue** (complexity, style, etc.)
3. **Run `mix credo` to verify**

**NEVER:**
- Add to .credo.exs disabled list
- Use inline `# credo:disable-for-this-file`
- Ignore the warning

**Credo is helping you write better code. Listen to it.**

## THE DISCIPLINE

TDD feels slow at first. That's because you're used to:
- Writing code fast (then debugging for hours)
- Skipping tests (then breaking things in production)
- Guessing if it works (then finding out it doesn't)

TDD is actually faster because:
- Tests catch bugs immediately
- You know exactly what to implement
- Refactoring is safe
- Code works the first time

## ENFORCEMENT

**Before writing ANY Elixir production code, ask:**

1. "Have I written a failing test for this?"
2. "Have I actually RUN the test and seen it fail?"
3. "Do I know WHY it's failing?"

**If any answer is NO → write the test first.**

## REMEMBER

> "Tests that pass on the first run might not be testing anything."

> "Code without a failing test first is guess-driven development."

> "TDD is slow. Debugging untested code is slower."

## THE RULE

**RED → GREEN → REFACTOR**

**Not GREEN → RED → "oops"**

**Not WRITE → PRAY → DEBUG**

**RED → GREEN → REFACTOR**

Every. Single. Time.
