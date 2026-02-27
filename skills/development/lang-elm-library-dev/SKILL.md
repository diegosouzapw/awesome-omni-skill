---
name: lang-elm-library-dev
description: Elm-specific library/package development patterns. Use when creating Elm packages, designing pure functional APIs, configuring elm.json for libraries, writing documentation comments, publishing to package.elm-lang.org, or managing semantic versioning in pure functional context. Extends meta-library-dev with Elm tooling and ecosystem patterns.
---

# Elm Library Development

Elm-specific patterns for library (package) development. This skill extends `meta-library-dev` with Elm's pure functional programming model, compiler-enforced guarantees, and package ecosystem practices.

## This Skill Extends

- `meta-library-dev` - Foundational library patterns (API design, versioning, testing strategies)

For general concepts like semantic versioning, module organization principles, and testing pyramids, see the meta-skill first.

## This Skill Adds

- **Elm tooling**: elm.json configuration, module exposure, elm-format, elm-test
- **Pure functional API design**: Immutability, type-driven development, opaque types
- **Elm ecosystem**: package.elm-lang.org publishing, documentation generation, semantic versioning enforcement
- **Elm idioms**: Custom types, pipelines, phantom types, extensible records

## This Skill Does NOT Cover

- General library patterns - see `meta-library-dev`
- Elm application development - see `lang-elm-dev`
- Frontend-specific patterns - see frontend skills
- JavaScript interop - see `lang-elm-dev` for ports

---

## Overview

Elm's package ecosystem is unique in enforcing semantic versioning and guaranteeing no runtime exceptions. The compiler and package manager work together to ensure API compatibility.

```
┌─────────────────────────────────────────────────────────────────┐
│                      Elm Package Stack                          │
├─────────────────────────────────────────────────────────────────┤
│  Source Code (src/)                                             │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │  elm.json   │───▶│     Elm     │───▶│  Compiled   │         │
│  │ (exposed-   │    │  Compiler   │    │  JavaScript │         │
│  │  modules)   │    │             │    │             │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
│       │                   │                   │                 │
│       │                   ▼                   │                 │
│       │            ┌─────────────┐            │                 │
│       │            │ Elm Package │            │                 │
│       │            │  Analyzer   │            │                 │
│       │            │  (semver)   │            │                 │
│       │            └─────────────┘            │                 │
│       ▼                   │                   ▼                 │
│  ┌─────────────────────────────────────────────────────┐       │
│  │         package.elm-lang.org Publishing             │       │
│  │  • Enforces semantic versioning                     │       │
│  │  • Generates documentation automatically            │       │
│  │  • Prevents breaking changes in patches/minors      │       │
│  └─────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────┘
```

**Key Elm Package Principles:**

| Principle | Enforcement | Benefit |
|-----------|-------------|---------|
| Semantic versioning | Compiler analyzes API changes | Guaranteed compatibility |
| No runtime exceptions | Type system + compiler | Reliability |
| Pure functions | Language design | Testability, predictability |
| Explicit exports | exposed-modules in elm.json | Clear public API |

---

## Quick Reference

| Task | Command |
|------|---------|
| Initialize package | `elm init` (then edit elm.json) |
| Install dependencies | `elm install author/package` |
| Build | `elm make src/MyModule.elm` |
| Test | `elm-test` |
| Format | `elm-format --yes src/` |
| Publish | `elm publish` |
| Bump version | `elm bump` (analyzes API changes) |
| View docs locally | `elm reactor` |
| Validate package | `elm make --docs=docs.json` |

---

## elm.json Structure

### Package Configuration

```json
{
    "type": "package",
    "name": "author/my-library",
    "summary": "A brief one-line summary (max 80 chars)",
    "license": "BSD-3-Clause",
    "version": "1.0.0",
    "exposed-modules": [
        "MyLibrary",
        "MyLibrary.Advanced"
    ],
    "elm-version": "0.19.0 <= v < 0.20.0",
    "dependencies": {
        "elm/core": "1.0.0 <= v < 2.0.0",
        "elm/json": "1.0.0 <= v < 2.0.0"
    },
    "test-dependencies": {
        "elm-explorations/test": "2.0.0 <= v < 3.0.0"
    }
}
```

### Required Fields

| Field | Purpose | Notes |
|-------|---------|-------|
| `type` | Must be `"package"` | Distinguishes from applications |
| `name` | Package identifier | Format: `author/package-name` |
| `summary` | One-line description | Max 80 characters, no period |
| `license` | OSI-approved license | Usually BSD-3-Clause for Elm |
| `version` | Current version | Enforced semantic versioning |
| `exposed-modules` | Public API | Only these modules are importable |
| `elm-version` | Elm version constraint | Usually `0.19.0 <= v < 0.20.0` |

### Exposed Modules Organization

**Flat structure (simpler packages):**
```json
{
    "exposed-modules": [
        "MyLibrary",
        "MyLibrary.Helpers"
    ]
}
```

**Categorized structure (complex packages):**
```json
{
    "exposed-modules": {
        "Core": [
            "MyLibrary",
            "MyLibrary.Types"
        ],
        "Advanced": [
            "MyLibrary.Advanced",
            "MyLibrary.Internal.Exposed"
        ],
        "Helpers": [
            "MyLibrary.String",
            "MyLibrary.List"
        ]
    }
}
```

**Best practices:**
- Keep `exposed-modules` minimal - only expose what users need
- Group related modules in categories for discoverability
- Internal modules not listed here are completely private
- Never expose modules with implementation details

---

## Module Organization

### Standard Package Structure

```
my-library/
├── elm.json                    # Package configuration
├── README.md                   # Documentation and examples
├── LICENSE                     # BSD-3-Clause (typical)
├── src/
│   ├── MyLibrary.elm          # Main public module
│   ├── MyLibrary/
│   │   ├── Types.elm          # Public types (exposed)
│   │   ├── Advanced.elm       # Advanced API (exposed)
│   │   └── Internal.elm       # Private implementation
│   └── Internal/              # Private utilities
│       └── Helpers.elm
└── tests/
    ├── Tests.elm              # Test suite
    └── Example.elm            # Usage examples as tests
```

### Module Design Principles

**Public Module (MyLibrary.elm):**
```elm
module MyLibrary exposing
    ( Config
    , parse
    , serialize
    , Error(..)
    )

{-| A library for parsing and serializing data.

# Configuration
@docs Config

# Parsing
@docs parse, Error

# Serialization
@docs serialize
-}

import MyLibrary.Internal as Internal
import MyLibrary.Types exposing (Config)


{-| Configuration for the parser.

    config =
        { strictMode = True
        , maxDepth = 100
        }
-}
type alias Config =
    MyLibrary.Types.Config


{-| Errors that can occur during parsing.
-}
type Error
    = InvalidSyntax String
    | DepthExceeded
    | UnexpectedEnd


{-| Parse input with the given configuration.

    parse config "some input"
        |> Result.map processData
-}
parse : Config -> String -> Result Error Data
parse config input =
    Internal.parseImpl config input


{-| Serialize data back to string format.

    serialize data
        |> String.length
        |> Debug.log "serialized length"
-}
serialize : Data -> String
serialize =
    Internal.serializeImpl
```

### Exposure Patterns

**Explicit exposure (preferred):**
```elm
module MyLibrary exposing
    ( Config
    , parse
    , serialize
    )
```

**Exposing custom types:**
```elm
-- Opaque type (recommended for encapsulation)
module MyLibrary exposing (Config)

-- Expose constructors (when users need pattern matching)
module MyLibrary exposing (Error(..))

-- Partially expose (allow pattern matching but not construction)
module MyLibrary exposing (Result(Ok, Err))
```

**Never use (..):**
```elm
-- Avoid: Exposes everything, loses control over API
module MyLibrary exposing (..)
```

---

## Public API Design (Elm-Specific)

### Type-Driven Design

**Opaque Types for Encapsulation:**
```elm
module MyLibrary exposing
    ( Email
    , fromString
    , toString
    )

{-| An email address that is guaranteed to be valid.
-}
type Email
    = Email String


{-| Create an email from a string.
Returns `Nothing` if the string is not a valid email.

    fromString "user@example.com"
        --> Just email

    fromString "invalid"
        --> Nothing
-}
fromString : String -> Maybe Email
fromString str =
    if isValidEmail str then
        Just (Email str)
    else
        Nothing


{-| Convert an email back to a string.

    email
        |> toString
        |> String.toLower
-}
toString : Email -> String
toString (Email str) =
    str
```

**Custom Types for State:**
```elm
{-| Represents the loading state of data.
-}
type RemoteData e a
    = NotAsked
    | Loading
    | Failure e
    | Success a


{-| Map a function over successful data.

    Success 5
        |> map ((+) 1)
        --> Success 6

    Loading
        |> map ((+) 1)
        --> Loading
-}
map : (a -> b) -> RemoteData e a -> RemoteData e b
map fn remoteData =
    case remoteData of
        NotAsked ->
            NotAsked

        Loading ->
            Loading

        Failure e ->
            Failure e

        Success a ->
            Success (fn a)
```

### Pipeline-Friendly APIs

**Design for |> operator:**
```elm
-- Good: Data flows naturally
data
    |> parse config
    |> Result.map validate
    |> Result.andThen process
    |> Result.withDefault defaultValue

-- Support for pipelines
parse : Config -> String -> Result Error Data
parse config input =
    -- config comes first for partial application
    parseImpl config input
```

**Function Ordering:**
```elm
-- Preferred: Configuration first, data last
parse : Config -> String -> Result Error Data
filter : (a -> Bool) -> List a -> List a
map : (a -> b) -> List a -> List b

-- This enables:
data
    |> filter isValid
    |> map transform
    |> parse config
```

### Extensible Records

**Use for flexible configuration:**
```elm
{-| Create a parser with custom configuration.

You can start with `defaultConfig` and override specific fields:

    customConfig =
        { defaultConfig | maxDepth = 200 }

    parseWith customConfig input
-}
parseWith : { r | maxDepth : Int, strict : Bool } -> String -> Result Error Data
parseWith config input =
    -- Works with any record containing these fields
    parseImpl config input


{-| Default configuration values.
-}
defaultConfig : { maxDepth : Int, strict : Bool }
defaultConfig =
    { maxDepth = 100
    , strict = False
    }
```

### Phantom Types

**Type-safe states:**
```elm
{-| A request builder that ensures required fields are set.
-}
type Request state
    = Request
        { url : String
        , headers : List ( String, String )
        }


type Incomplete
    = Incomplete Never


type Complete
    = Complete Never


{-| Start building a request.
-}
new : String -> Request Incomplete
new url =
    Request
        { url = url
        , headers = []
        }


{-| Add a header to the request.
-}
withHeader : String -> String -> Request state -> Request state
withHeader key value (Request req) =
    Request { req | headers = ( key, value ) :: req.headers }


{-| Mark the request as ready to send.
-}
build : Request Incomplete -> Request Complete
build (Request req) =
    Request req


{-| Send a complete request. Won't compile with incomplete request!
-}
send : Request Complete -> Cmd msg
send (Request req) =
    -- implementation
    Cmd.none
```

---

## Documentation Standards

### Module Documentation

Every module needs comprehensive documentation:

```elm
module MyLibrary.Parser exposing
    ( Config
    , parse
    , parseWith
    , Error(..)
    )

{-| A robust parser for custom data formats.

This module provides parsing functionality with configurable options
and detailed error reporting.

# Configuration
@docs Config

# Parsing
@docs parse, parseWith

# Errors
@docs Error

# Example

    import MyLibrary.Parser as Parser

    case Parser.parse "some input" of
        Ok data ->
            -- process data

        Err (Parser.InvalidSyntax msg) ->
            -- handle syntax error
-}
```

### Function Documentation

**Complete documentation template:**
```elm
{-| Parse a string into structured data.

This function validates the input and returns structured data
or an error describing what went wrong.

The parser follows these rules:
- Whitespace is ignored
- Comments start with '#' and continue to end of line
- Maximum nesting depth is 100 levels

    parse "valid input"
        --> Ok data

    parse ""
        --> Err UnexpectedEnd

    parse "# just a comment"
        --> Ok emptyData
-}
parse : String -> Result Error Data
parse input =
    parseImpl defaultConfig input
```

**Documentation sections:**
1. **Summary** - What the function does (1 sentence)
2. **Details** - How it works, any important rules
3. **Examples** - Doctests showing usage
4. **Notes** - Edge cases, performance considerations

### Documentation Tests

Elm's documentation includes doctests:

```elm
{-| Add two numbers.

    add 2 3
        --> 5

    add -1 1
        --> 0
-}
add : Int -> Int -> Int
add x y =
    x + y
```

**Best practices:**
- Include both simple and edge-case examples
- Show typical usage patterns
- Demonstrate error cases
- Use realistic examples

---

## Semantic Versioning in Elm

### Automatic Version Bumping

Elm enforces semantic versioning by analyzing API changes:

```bash
# Let Elm determine the version bump
elm bump

# Elm will:
# - Analyze API changes since last version
# - Determine if MAJOR, MINOR, or PATCH
# - Update elm.json automatically
```

### What Elm Considers Breaking

| Change | Version Bump | Example |
|--------|--------------|---------|
| Add new exposed module | MINOR | New module in `exposed-modules` |
| Add new function | MINOR | New export in existing module |
| Add new variant to type | MAJOR | `type Error = ... \| NewError` |
| Remove function | MAJOR | Delete exported function |
| Change function signature | MAJOR | Change types of parameters/return |
| Rename function | MAJOR | Old name no longer exists |
| Make type opaque | MAJOR | Remove `(..)` from exposure |

### Pre-1.0.0 Development

```json
{
    "version": "0.1.0"
}
```

**During 0.x.y:**
- Breaking changes bump MINOR (0.1.0 → 0.2.0)
- New features bump PATCH (0.1.0 → 0.1.1)
- Use for initial development

**When to publish 1.0.0:**
- API is stable and tested
- Documentation is complete
- You're ready to commit to compatibility

### Version Constraints

**In dependencies:**
```json
{
    "dependencies": {
        "elm/core": "1.0.0 <= v < 2.0.0",
        "elm/json": "1.1.0 <= v < 2.0.0"
    }
}
```

**Constraint format:**
- `1.0.0 <= v < 2.0.0` - Allow any 1.x.x version
- `1.1.0 <= v < 2.0.0` - Require at least 1.1.0
- Elm enforces these during `elm install`

---

## Testing Elm Libraries

### elm-test Configuration

**tests/Tests.elm:**
```elm
module Tests exposing (suite)

import Expect
import MyLibrary
import Test exposing (..)


suite : Test
suite =
    describe "MyLibrary"
        [ describe "parse"
            [ test "parses valid input" <|
                \_ ->
                    MyLibrary.parse "valid"
                        |> Expect.ok
            , test "rejects invalid input" <|
                \_ ->
                    MyLibrary.parse ""
                        |> Expect.err
            , test "handles edge cases" <|
                \_ ->
                    MyLibrary.parse "edge case"
                        |> Result.map MyLibrary.serialize
                        |> Expect.equal (Ok "edge case")
            ]
        ]
```

### Test Organization

**Group tests by module:**
```elm
module Tests exposing (suite)

import ParserTests
import SerializerTests
import TypesTests
import Test exposing (..)


suite : Test
suite =
    describe "MyLibrary"
        [ ParserTests.suite
        , SerializerTests.suite
        , TypesTests.suite
        ]
```

### Property-Based Testing

**Using elm-explorations/test:**
```elm
import Fuzz exposing (Fuzzer)
import Test exposing (..)


suite : Test
suite =
    describe "roundtrip property"
        [ fuzz Fuzz.string "parse then serialize returns original" <|
            \input ->
                input
                    |> parse
                    |> Result.map serialize
                    |> Expect.equal (Ok input)
        , fuzz dataFuzzer "any valid data can be serialized" <|
            \data ->
                serialize data
                    |> parse
                    |> Expect.equal (Ok data)
        ]


dataFuzzer : Fuzzer Data
dataFuzzer =
    Fuzz.map2 Data
        Fuzz.string
        (Fuzz.intRange 0 100)
```

### Example-Based Tests

```elm
{-| Tests can also serve as examples.
-}
exampleUsage : Test
exampleUsage =
    describe "Example usage"
        [ test "basic parsing workflow" <|
            \_ ->
                let
                    config =
                        { maxDepth = 50
                        , strict = True
                        }

                    input =
                        "example input"

                    expected =
                        Data "example" 42
                in
                parseWith config input
                    |> Expect.equal (Ok expected)
        ]
```

---

## Publishing to package.elm-lang.org

### Pre-publish Checklist

- [ ] `elm.json` has all required fields
- [ ] `README.md` is comprehensive with examples
- [ ] `LICENSE` file is present (BSD-3-Clause recommended)
- [ ] All exposed modules have documentation
- [ ] Documentation examples are tested
- [ ] `elm-format` applied to all files
- [ ] `elm-test` passes
- [ ] `elm make --docs=docs.json` succeeds
- [ ] Version number follows semver (use `elm bump`)

### Publishing Process

```bash
# 1. Ensure everything is clean
git status

# 2. Validate the package
elm make --docs=docs.json

# 3. Let Elm determine version
elm bump

# 4. Review the version change
git diff elm.json

# 5. Commit the version bump
git add elm.json
git commit -m "Bump to X.Y.Z"

# 6. Tag the release
git tag -a X.Y.Z -m "Release X.Y.Z"

# 7. Publish
elm publish

# 8. Push to GitHub
git push origin main --tags
```

### First Publication

**Requirements:**
- GitHub repository must exist
- `elm.json` must be valid
- At least one exposed module
- Documentation for all exposed values
- Valid license

**Common issues:**
```bash
# If elm publish fails:

# Check documentation completeness
elm make --docs=docs.json

# Ensure all tests pass
elm-test

# Verify git tag matches version
git tag -l
```

### Package Updates

**For PATCH releases (bug fixes):**
```bash
# Fix the bug
# Add tests
elm bump  # Will suggest PATCH if no API changes
elm publish
```

**For MINOR releases (new features):**
```bash
# Add new functions or modules
# Update documentation
elm bump  # Will suggest MINOR for additions
elm publish
```

**For MAJOR releases (breaking changes):**
```bash
# Make breaking changes
# Update CHANGELOG with migration guide
elm bump  # Will suggest MAJOR for breaking changes
elm publish
```

---

## Common Design Patterns

### The Builder Pattern

```elm
{-| Build a configuration incrementally.
-}
type Config
    = Config
        { maxDepth : Int
        , strict : Bool
        , timeout : Maybe Int
        }


{-| Start with default configuration.
-}
default : Config
default =
    Config
        { maxDepth = 100
        , strict = False
        , timeout = Nothing
        }


{-| Set maximum depth.
-}
withMaxDepth : Int -> Config -> Config
withMaxDepth depth (Config config) =
    Config { config | maxDepth = depth }


{-| Enable strict mode.
-}
withStrict : Bool -> Config -> Config
withStrict strict (Config config) =
    Config { config | strict = strict }


-- Usage:
myConfig =
    default
        |> withMaxDepth 200
        |> withStrict True
```

### The Newtype Pattern

```elm
{-| A validated email address.
-}
type Email
    = Email String


{-| Create an email from string.
-}
fromString : String -> Maybe Email
fromString str =
    if String.contains "@" str then
        Just (Email str)
    else
        Nothing


{-| Convert to string.
-}
toString : Email -> String
toString (Email str) =
    str
```

### The Extension Pattern

```elm
{-| Extend records without breaking users.
-}
type alias Minimal r =
    { r
        | name : String
        , age : Int
    }


{-| Works with any record containing name and age.
-}
greet : Minimal r -> String
greet person =
    "Hello, " ++ person.name


-- Users can pass records with extra fields:
fullPerson =
    { name = "Alice"
    , age = 30
    , email = "alice@example.com"
    , city = "Portland"
    }

greeting =
    greet fullPerson  -- Works fine!
```

---

## Performance Considerations

### Lazy Evaluation Patterns

```elm
{-| Use lazy lists for potentially infinite sequences.
-}
type Lazy a
    = Lazy (() -> a)


{-| Generate an infinite sequence.
-}
repeat : a -> Lazy (List a)
repeat value =
    Lazy (\_ -> value :: force (repeat value))


force : Lazy a -> a
force (Lazy thunk) =
    thunk ()
```

### Efficient Data Structures

**Prefer built-in types:**
```elm
-- Good: Use Dict for key-value pairs
import Dict exposing (Dict)

type alias Cache =
    Dict String Value


-- Good: Use Set for unique values
import Set exposing (Set)

type alias UniqueIds =
    Set String


-- Avoid: List of tuples for lookups
-- (O(n) lookup time)
type alias SlowCache =
    List ( String, Value )
```

### Tail-Call Optimization

```elm
{-| Tail-recursive sum (won't overflow stack).
-}
sum : List Int -> Int
sum list =
    sumHelper 0 list


sumHelper : Int -> List Int -> Int
sumHelper acc list =
    case list of
        [] ->
            acc

        x :: xs ->
            sumHelper (acc + x) xs
```

---

## API Evolution Strategies

### Deprecation Pattern

```elm
{-| DEPRECATED: Use `parseWith` instead.

This function will be removed in the next major version.
-}
parse : String -> Result Error Data
parse =
    parseWith defaultConfig


{-| Parse with explicit configuration (recommended).

    parseWith defaultConfig input
-}
parseWith : Config -> String -> Result Error Data
parseWith config input =
    -- implementation
    Ok (Data "example" 42)
```

### Versioned Modules

```elm
-- For major API changes, create new modules:

-- v1
module MyLibrary exposing (parse)

-- v2 (alongside v1)
module MyLibrary.V2 exposing (parseWithConfig)

-- Users can migrate gradually
```

### Additive Changes Only

```elm
-- Version 1.0.0
type Error
    = InvalidSyntax String
    | UnexpectedEnd


-- Version 1.1.0 (adding, not changing)
type Error
    = InvalidSyntax String
    | UnexpectedEnd
    | DepthExceeded  -- New variant (MAJOR bump required!)


-- Safe way to add: Create new type
type ErrorV2
    = V1Error Error
    | DepthExceeded
```

---

## Anti-Patterns

### 1. Exposing Implementation Details

```elm
-- Bad: Exposes internal structure
module MyLibrary exposing (Data(..))

type Data
    = Internal String Int (List String)


-- Good: Opaque type
module MyLibrary exposing (Data)

type Data
    = Data String Int (List String)
```

### 2. Overusing Extensible Records

```elm
-- Bad: Too flexible, loses type safety
process : { r | field : String } -> Result Error a

-- Good: Concrete type
type alias Input =
    { field : String
    , otherField : Int
    }

process : Input -> Result Error Data
```

### 3. Missing Documentation

```elm
-- Bad: No documentation
parse : String -> Result Error Data
parse input =
    Ok (Data "example" 42)


-- Good: Comprehensive docs
{-| Parse input string into structured data.

    parse "valid input"
        --> Ok data
-}
parse : String -> Result Error Data
parse input =
    Ok (Data "example" 42)
```

### 4. Non-Pipeline-Friendly Functions

```elm
-- Bad: Data comes first
validate : Data -> Config -> Result Error Data

-- Good: Config first, data last (enables pipelines)
validate : Config -> Data -> Result Error Data
```

---

## Dependency Management

### Choosing Dependencies

**Prefer core and official packages:**
```json
{
    "dependencies": {
        "elm/core": "1.0.0 <= v < 2.0.0",
        "elm/json": "1.1.0 <= v < 2.0.0",
        "elm/http": "2.0.0 <= v < 3.0.0"
    }
}
```

**Evaluate third-party packages:**
- Recent updates?
- Good documentation?
- Used by other packages?
- Compatible license?
- Small dependency tree?

### Minimal Dependencies

**Keep dependencies minimal:**
- Each dependency increases bundle size
- More dependencies = more potential conflicts
- Consider implementing simple utilities yourself
- Prefer pure Elm over packages requiring ports

### Version Constraints

```json
{
    "dependencies": {
        // Exact major version
        "elm/core": "1.0.0 <= v < 2.0.0",

        // Require newer minor version
        "elm/json": "1.1.0 <= v < 2.0.0",

        // Same major version as your package
        "author/package": "3.0.0 <= v < 4.0.0"
    }
}
```

---

## Troubleshooting

### Common Compilation Errors

**Exposed module not found:**
```
-- ERROR in elm.json
Module `MyLibrary.Missing` is listed in exposed-modules but not found
```
**Fix:** Ensure file exists at `src/MyLibrary/Missing.elm`

**Missing documentation:**
```
-- DOCUMENTATION ERROR
The module `MyLibrary` is missing documentation for:
    - parse
```
**Fix:** Add `{-| ... -}` documentation before `parse`

### Publishing Issues

**Version conflict:**
```
-- VERSION CONFLICT
Changes require MAJOR bump but elm.json has MINOR
```
**Fix:** Run `elm bump` to let Elm determine correct version

**Missing tag:**
```
-- GIT TAG MISSING
Version X.Y.Z in elm.json but no git tag found
```
**Fix:** `git tag -a X.Y.Z -m "Release X.Y.Z"`

### Testing Problems

**Imported module not found in tests:**
```
-- MODULE NOT FOUND
Cannot find module `MyLibrary.Internal`
```
**Fix:** Only test exposed modules, or expose for testing temporarily

---

## References

- `meta-library-dev` - Foundational library patterns
- `lang-elm-dev` - General Elm development patterns
- [Elm Package Documentation](https://package.elm-lang.org/help/documentation-format)
- [Elm Publishing Guide](https://package.elm-lang.org/help/design-guidelines)
- [Elm API Design Guidelines](https://package.elm-lang.org/help/design-guidelines)
- [Elm Package Versioning](https://github.com/elm/compiler/blob/master/docs/upgrade-instructions/0.19.md)
