---
name: github.com/n-r-w/ctxlog guidelines
description: Guidelines and examples for using the ctxlog logging package.
---

<ctxlog name="github.com/n-r-w/ctxlog guidelines">
    <instructions>
        - Import path: `github.com/n-r-w/ctxlog`.
        - Primary workflow:
            - Parse log level from environment variable or config: `ctxlog.ParseLogLevel()`
            - Create a logger once (app startup, request entrypoint, job runner) and store it in a context
            - Log everywhere via context-based helpers: `ctxlog.Debug/Info/Warn/Error(ctx, msg, ...)`
            - Enrich the context logger when you need extra fields: `ctx = ctxlog.With(ctx, ...)`
            - Group related fields: `ctx = ctxlog.WithGroup(ctx, "group")`
        - Passing structured fields:
            - You can pass key/value pairs: `"user_id", "12345"`
            - You can pass `slog.Attr` values: `slog.String("host", "localhost")`
            - When passing key/value pairs, the number of arguments must be even (key-value-key-value...)
        - Optional: extract the logger from context for direct usage: `logger := ctxlog.FromContext(ctx)`
        - For external dependencies that should not depend on the concrete logger type, use `ctxlog.ILogger` via `ctxlog.NewWrapper()` (real) or `ctxlog.NewStubWrapper()` (no-op)
    </instructions>
    <example>     
        ```go
        package main

        import (
            "context"
            "log/slog"

            "github.com/n-r-w/ctxlog"
        )

        func main() {
            // Parse level from environment variable
            // (here we hardcode it for demonstration)
            level, err := ctxlog.ParseLogLevel("DEBUG")
            if err != nil {
                panic(err)
            }
            
            // create context with logger
            ctx := ctxlog.MustContext(
                context.Background(),
                ctxlog.WithName("myapp"),
                ctxlog.WithLevel(level),
            )
            
            // simple format (key/value pairs)
            ctxlog.Info(ctx, "starting application", "version", "1.0.0")

            // slog attr fields
            ctxlog.Debug(ctx, "connecting to database",
                slog.String("host", "localhost"),
                slog.Int("port", 5432),
            )

            // enrich context with more fields
            ctx = ctxlog.WithGroup(ctx, "database")
            ctx = ctxlog.With(ctx, "user_id", "12345", "ip", "192.168.1.1")
            ctxlog.Error(ctx, "some error occurred")

            // extract logger from context for direct usage
            logger := ctxlog.FromContext(ctx)
            logger.Warn(ctx, "direct usage of logger")
        }
        ```
    </example>
</ctxlog>
