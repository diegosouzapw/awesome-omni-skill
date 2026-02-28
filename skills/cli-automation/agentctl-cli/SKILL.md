---
name: agentctl-cli
description: Build CLI tools using Go with Cobra and Viper. Use for implementing agentctl commands, interactive prompts, configuration management, and output formatting. Triggers on "CLI", "agentctl", "command line", "cobra", "terminal application", "interactive prompt", or when implementing spec/009-developer-experience.md CLI section.
---

# agentctl CLI Development

## Overview

Build the `agentctl` CLI tool for AgentStack platform interaction. Implements authentication, project management, agent operations, development workflows, and evaluation commands.

## CLI Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     agentctl Architecture                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    cmd/agentctl/                          │  │
│  │  main.go → root.go → [auth|project|agent|dev|eval].go   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                           │                                     │
│                           ▼                                     │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                  internal/cli/                            │  │
│  │  config/ │ client/ │ output/ │ prompt/ │ spinner/        │  │
│  └──────────────────────────────────────────────────────────┘  │
│                           │                                     │
│                           ▼                                     │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                 pkg/agentstack/                           │  │
│  │  SDK client for API communication                         │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Project Structure

```
agentctl/
├── cmd/agentctl/
│   ├── main.go
│   ├── root.go           # Root command, global flags
│   ├── auth.go           # auth login, logout, whoami
│   ├── project.go        # project create, list, switch
│   ├── agent.go          # agent init, deploy, list, logs, delete
│   ├── dev.go            # dev (local development)
│   ├── eval.go           # eval run, dataset, report, compare
│   └── version.go        # version command
├── internal/cli/
│   ├── config/           # Configuration management
│   ├── client/           # API client wrapper
│   ├── output/           # Table, JSON, YAML output
│   ├── prompt/           # Interactive prompts
│   └── spinner/          # Progress indicators
├── pkg/agentstack/       # SDK (can be external package)
├── templates/            # Agent scaffolding templates
└── Makefile
```

## Root Command Setup

```go
// cmd/agentctl/root.go
package main

import (
    "os"
    
    "github.com/spf13/cobra"
    "github.com/spf13/viper"
)

var (
    cfgFile string
    output  string
)

var rootCmd = &cobra.Command{
    Use:   "agentctl",
    Short: "AgentStack CLI - Deploy and manage AI agents",
    Long: `agentctl is the command-line interface for the AgentStack platform.
    
It allows you to create, deploy, and manage AI agents with full 
observability and evaluation capabilities.`,
    PersistentPreRunE: func(cmd *cobra.Command, args []string) error {
        return initConfig()
    },
}

func init() {
    // Global flags
    rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", 
        "config file (default is $HOME/.agentctl/config.yaml)")
    rootCmd.PersistentFlags().StringVarP(&output, "output", "o", "table", 
        "output format: table, json, yaml")
    rootCmd.PersistentFlags().Bool("no-color", false, 
        "disable colored output")
    
    // Bind to viper
    viper.BindPFlag("output", rootCmd.PersistentFlags().Lookup("output"))
    
    // Add subcommands
    rootCmd.AddCommand(authCmd)
    rootCmd.AddCommand(projectCmd)
    rootCmd.AddCommand(agentCmd)
    rootCmd.AddCommand(devCmd)
    rootCmd.AddCommand(evalCmd)
    rootCmd.AddCommand(versionCmd)
}

func initConfig() error {
    if cfgFile != "" {
        viper.SetConfigFile(cfgFile)
    } else {
        home, err := os.UserHomeDir()
        if err != nil {
            return err
        }
        viper.AddConfigPath(home + "/.agentctl")
        viper.SetConfigName("config")
        viper.SetConfigType("yaml")
    }
    
    viper.SetEnvPrefix("AGENTCTL")
    viper.AutomaticEnv()
    
    viper.ReadInConfig() // Ignore error if not exists
    return nil
}

func main() {
    if err := rootCmd.Execute(); err != nil {
        os.Exit(1)
    }
}
```

## Authentication Commands

```go
// cmd/agentctl/auth.go
package main

import (
    "fmt"
    
    "github.com/spf13/cobra"
)

var authCmd = &cobra.Command{
    Use:   "auth",
    Short: "Manage authentication",
}

var authLoginCmd = &cobra.Command{
    Use:   "login",
    Short: "Login to AgentStack",
    RunE: func(cmd *cobra.Command, args []string) error {
        token, _ := cmd.Flags().GetString("token")
        
        if token != "" {
            return loginWithToken(token)
        }
        return loginBrowser()
    },
}

var authWhoamiCmd = &cobra.Command{
    Use:   "whoami",
    Short: "Display current identity",
    RunE: func(cmd *cobra.Command, args []string) error {
        cfg := config.Load()
        if cfg.Token == "" {
            return fmt.Errorf("not logged in, run: agentctl auth login")
        }
        
        client := client.New(cfg)
        user, err := client.Auth.WhoAmI(cmd.Context())
        if err != nil {
            return err
        }
        
        output.Print(user, output.Format(viper.GetString("output")))
        return nil
    },
}

func init() {
    authLoginCmd.Flags().String("token", "", "API token for non-interactive login")
    
    authCmd.AddCommand(authLoginCmd)
    authCmd.AddCommand(authWhoamiCmd)
    authCmd.AddCommand(&cobra.Command{
        Use:   "logout",
        Short: "Logout from AgentStack",
        RunE: func(cmd *cobra.Command, args []string) error {
            return config.ClearCredentials()
        },
    })
}

func loginBrowser() error {
    // Open browser for OAuth flow
    fmt.Println("Opening browser for login...")
    
    // Start local server for callback
    server := oauth.NewCallbackServer(8765)
    go server.Start()
    
    // Open browser
    url := fmt.Sprintf("%s/auth/cli?port=8765", config.Load().Endpoint)
    browser.Open(url)
    
    // Wait for token
    token := <-server.TokenChan
    
    // Save token
    return config.SaveCredentials(token)
}
```

## Agent Commands

```go
// cmd/agentctl/agent.go
package main

import (
    "fmt"
    "os"
    "text/template"
    
    "github.com/spf13/cobra"
)

var agentCmd = &cobra.Command{
    Use:   "agent",
    Short: "Manage agents",
}

var agentInitCmd = &cobra.Command{
    Use:   "init <name>",
    Short: "Scaffold a new agent",
    Args:  cobra.ExactArgs(1),
    RunE: func(cmd *cobra.Command, args []string) error {
        name := args[0]
        tmpl, _ := cmd.Flags().GetString("template")
        
        return scaffoldAgent(name, tmpl)
    },
}

var agentDeployCmd = &cobra.Command{
    Use:   "deploy [path]",
    Short: "Deploy an agent to the platform",
    Args:  cobra.MaximumNArgs(1),
    RunE: func(cmd *cobra.Command, args []string) error {
        path := "."
        if len(args) > 0 {
            path = args[0]
        }
        
        wait, _ := cmd.Flags().GetBool("wait")
        return deployAgent(cmd.Context(), path, wait)
    },
}

var agentLogsCmd = &cobra.Command{
    Use:   "logs <name>",
    Short: "Stream agent logs",
    Args:  cobra.ExactArgs(1),
    RunE: func(cmd *cobra.Command, args []string) error {
        follow, _ := cmd.Flags().GetBool("follow")
        tail, _ := cmd.Flags().GetInt("tail")
        
        return streamLogs(cmd.Context(), args[0], follow, tail)
    },
}

func init() {
    agentInitCmd.Flags().StringP("template", "t", "google-adk", 
        "agent template: google-adk, langchain, custom")
    
    agentDeployCmd.Flags().BoolP("wait", "w", false, 
        "wait for deployment to complete")
    
    agentLogsCmd.Flags().BoolP("follow", "f", false, "follow log output")
    agentLogsCmd.Flags().IntP("tail", "n", 100, "number of lines to show")
    
    agentCmd.AddCommand(agentInitCmd)
    agentCmd.AddCommand(agentDeployCmd)
    agentCmd.AddCommand(agentLogsCmd)
    agentCmd.AddCommand(&cobra.Command{
        Use:   "list",
        Short: "List agents",
        RunE:  listAgents,
    })
    agentCmd.AddCommand(&cobra.Command{
        Use:   "delete <name>",
        Short: "Delete an agent",
        Args:  cobra.ExactArgs(1),
        RunE:  deleteAgent,
    })
}

func scaffoldAgent(name, tmpl string) error {
    spinner := spinner.New("Creating agent: " + name)
    spinner.Start()
    
    // Create directory
    if err := os.MkdirAll(name, 0755); err != nil {
        spinner.Fail("Failed to create directory")
        return err
    }
    spinner.Success("Created directory structure")
    
    // Generate files from template
    files := templates.Get(tmpl)
    for _, f := range files {
        spinner.Update("Generating " + f.Name)
        if err := generateFile(name, f); err != nil {
            spinner.Fail("Failed to generate " + f.Name)
            return err
        }
        spinner.Success("Generated " + f.Name)
    }
    
    // Print next steps
    fmt.Printf("\n✓ Agent scaffolded: %s\n\n", name)
    fmt.Println("Next steps:")
    fmt.Printf("  cd %s\n", name)
    fmt.Println("  agentctl dev          # Start local development")
    fmt.Println("  agentctl deploy       # Deploy to platform")
    
    return nil
}
```

## Development Commands

```go
// cmd/agentctl/dev.go
package main

import (
    "context"
    "fmt"
    "os"
    "os/exec"
    "os/signal"
    
    "github.com/spf13/cobra"
)

var devCmd = &cobra.Command{
    Use:   "dev",
    Short: "Start local development server",
    RunE:  runDev,
}

func init() {
    devCmd.Flags().IntP("port", "p", 8080, "port to run on")
    devCmd.Flags().Bool("mock-llm", false, "use mock LLM provider")
}

func runDev(cmd *cobra.Command, args []string) error {
    port, _ := cmd.Flags().GetInt("port")
    mockLLM, _ := cmd.Flags().GetBool("mock-llm")
    
    fmt.Println("Starting local development server...")
    
    // Check for agent.yaml
    if _, err := os.Stat("agent.yaml"); os.IsNotExist(err) {
        return fmt.Errorf("agent.yaml not found. Run this from an agent directory")
    }
    
    spinner := spinner.New("Loading agent.yaml")
    spinner.Start()
    
    agentCfg, err := loadAgentConfig("agent.yaml")
    if err != nil {
        spinner.Fail("Failed to load agent.yaml")
        return err
    }
    spinner.Success("Loaded agent.yaml")
    
    // Build container
    spinner.Update("Building container")
    if err := buildContainer(); err != nil {
        spinner.Fail("Build failed")
        return err
    }
    spinner.Success("Built container")
    
    // Start dependencies
    spinner.Update("Starting dependencies")
    deps, err := startDependencies(mockLLM)
    if err != nil {
        spinner.Fail("Failed to start dependencies")
        return err
    }
    defer deps.Stop()
    spinner.Success("Started dependencies")
    
    // Start agent
    spinner.Update("Starting agent")
    agent, err := startAgent(port, agentCfg)
    if err != nil {
        spinner.Fail("Failed to start agent")
        return err
    }
    spinner.Success(fmt.Sprintf("Agent running at http://localhost:%d", port))
    
    // Print endpoints
    fmt.Println("\nEndpoints:")
    fmt.Printf("  Chat:    POST http://localhost:%d/chat\n", port)
    fmt.Printf("  SSE:     POST http://localhost:%d/chat/stream\n", port)
    fmt.Printf("  Health:  GET  http://localhost:%d/health\n", port)
    fmt.Println("\nWatching for changes... (Ctrl+C to stop)")
    
    // Watch for file changes
    go watchFiles(agent)
    
    // Wait for interrupt
    ctx, cancel := signal.NotifyContext(context.Background(), os.Interrupt)
    defer cancel()
    <-ctx.Done()
    
    fmt.Println("\nShutting down...")
    agent.Stop()
    
    return nil
}
```

## Evaluation Commands

```go
// cmd/agentctl/eval.go
package main

import (
    "fmt"
    
    "github.com/spf13/cobra"
)

var evalCmd = &cobra.Command{
    Use:   "eval",
    Short: "Run agent evaluations",
}

var evalRunCmd = &cobra.Command{
    Use:   "run [agent]",
    Short: "Run evaluation suite",
    Args:  cobra.MaximumNArgs(1),
    RunE: func(cmd *cobra.Command, args []string) error {
        agent := ""
        if len(args) > 0 {
            agent = args[0]
        }
        
        dataset, _ := cmd.Flags().GetString("dataset")
        scorers, _ := cmd.Flags().GetStringSlice("scorer")
        
        return runEvaluation(cmd.Context(), agent, dataset, scorers)
    },
}

var evalCompareCmd = &cobra.Command{
    Use:   "compare <run1> <run2>",
    Short: "Compare two evaluation runs",
    Args:  cobra.ExactArgs(2),
    RunE: func(cmd *cobra.Command, args []string) error {
        return compareRuns(cmd.Context(), args[0], args[1])
    },
}

func init() {
    evalRunCmd.Flags().StringP("dataset", "d", "", "evaluation dataset")
    evalRunCmd.Flags().StringSliceP("scorer", "s", nil, 
        "scorers to use (can specify multiple)")
    
    evalCmd.AddCommand(evalRunCmd)
    evalCmd.AddCommand(evalCompareCmd)
    evalCmd.AddCommand(&cobra.Command{
        Use:   "report <run-id>",
        Short: "View evaluation report",
        Args:  cobra.ExactArgs(1),
        RunE: func(cmd *cobra.Command, args []string) error {
            return showReport(cmd.Context(), args[0])
        },
    })
}

func runEvaluation(ctx context.Context, agent, dataset string, scorers []string) error {
    spinner := spinner.New("Running evaluation")
    spinner.Start()
    
    client := client.New(config.Load())
    
    // Start evaluation
    run, err := client.Eval.Start(ctx, &EvalRequest{
        AgentID:  agent,
        Dataset:  dataset,
        Scorers:  scorers,
    })
    if err != nil {
        spinner.Fail("Failed to start evaluation")
        return err
    }
    
    // Poll for completion
    for run.Status == "running" {
        spinner.Update(fmt.Sprintf("Running... %d/%d", run.Completed, run.Total))
        time.Sleep(2 * time.Second)
        run, _ = client.Eval.Get(ctx, run.ID)
    }
    
    if run.Status == "failed" {
        spinner.Fail("Evaluation failed")
        return fmt.Errorf("evaluation failed: %s", run.Error)
    }
    
    spinner.Success("Evaluation complete")
    
    // Print results
    fmt.Println("\nResults:")
    output.PrintTable(run.Results, []string{"Scorer", "Score", "Passed"})
    
    fmt.Printf("\nFull report: agentctl eval report %s\n", run.ID)
    
    return nil
}
```

## Configuration Management

```go
// internal/cli/config/config.go
package config

import (
    "os"
    "path/filepath"
    
    "gopkg.in/yaml.v3"
)

type Config struct {
    CurrentContext string     `yaml:"current-context"`
    Contexts       []Context  `yaml:"contexts"`
}

type Context struct {
    Name     string `yaml:"name"`
    Endpoint string `yaml:"endpoint"`
    Project  string `yaml:"project"`
    Token    string `yaml:"token,omitempty"`
}

func Load() *Config {
    home, _ := os.UserHomeDir()
    path := filepath.Join(home, ".agentctl", "config.yaml")
    
    data, err := os.ReadFile(path)
    if err != nil {
        return &Config{
            CurrentContext: "default",
            Contexts: []Context{{
                Name:     "default",
                Endpoint: "https://api.agentstack.io",
            }},
        }
    }
    
    var cfg Config
    yaml.Unmarshal(data, &cfg)
    return &cfg
}

func (c *Config) Current() *Context {
    for _, ctx := range c.Contexts {
        if ctx.Name == c.CurrentContext {
            return &ctx
        }
    }
    return nil
}

func (c *Config) Save() error {
    home, _ := os.UserHomeDir()
    dir := filepath.Join(home, ".agentctl")
    os.MkdirAll(dir, 0700)
    
    path := filepath.Join(dir, "config.yaml")
    data, _ := yaml.Marshal(c)
    return os.WriteFile(path, data, 0600)
}
```

## Output Formatting

```go
// internal/cli/output/output.go
package output

import (
    "encoding/json"
    "fmt"
    "os"
    
    "github.com/olekukonko/tablewriter"
    "gopkg.in/yaml.v3"
)

type Format string

const (
    FormatTable Format = "table"
    FormatJSON  Format = "json"
    FormatYAML  Format = "yaml"
)

func Print(v interface{}, format Format) {
    switch format {
    case FormatJSON:
        enc := json.NewEncoder(os.Stdout)
        enc.SetIndent("", "  ")
        enc.Encode(v)
    case FormatYAML:
        enc := yaml.NewEncoder(os.Stdout)
        enc.Encode(v)
    default:
        // Handle table format based on type
        printTable(v)
    }
}

func PrintTable(data [][]string, headers []string) {
    table := tablewriter.NewWriter(os.Stdout)
    table.SetHeader(headers)
    table.SetBorder(false)
    table.SetHeaderLine(true)
    table.AppendBulk(data)
    table.Render()
}

func Success(msg string) {
    fmt.Printf("✓ %s\n", msg)
}

func Error(msg string) {
    fmt.Fprintf(os.Stderr, "✗ %s\n", msg)
}

func Warning(msg string) {
    fmt.Printf("⚠ %s\n", msg)
}
```

## Interactive Prompts

```go
// internal/cli/prompt/prompt.go
package prompt

import (
    "github.com/AlecAivazis/survey/v2"
)

func Confirm(message string) (bool, error) {
    var result bool
    err := survey.AskOne(&survey.Confirm{
        Message: message,
    }, &result)
    return result, err
}

func Select(message string, options []string) (string, error) {
    var result string
    err := survey.AskOne(&survey.Select{
        Message: message,
        Options: options,
    }, &result)
    return result, err
}

func Input(message string, defaultValue string) (string, error) {
    var result string
    err := survey.AskOne(&survey.Input{
        Message: message,
        Default: defaultValue,
    }, &result)
    return result, err
}

func Password(message string) (string, error) {
    var result string
    err := survey.AskOne(&survey.Password{
        Message: message,
    }, &result)
    return result, err
}
```

## Dependencies

```
github.com/spf13/cobra v1.8.0
github.com/spf13/viper v1.18.0
github.com/AlecAivazis/survey/v2 v2.3.7
github.com/olekukonko/tablewriter v0.0.5
github.com/briandowns/spinner v1.23.0
github.com/fatih/color v1.16.0
github.com/fsnotify/fsnotify v1.7.0
gopkg.in/yaml.v3 v3.0.1
```

## Testing

```go
// cmd/agentctl/agent_test.go
package main

import (
    "bytes"
    "testing"
    
    "github.com/stretchr/testify/assert"
)

func TestAgentInit(t *testing.T) {
    // Create temp directory
    dir := t.TempDir()
    
    // Run init
    buf := new(bytes.Buffer)
    rootCmd.SetOut(buf)
    rootCmd.SetArgs([]string{"agent", "init", "test-agent", "-t", "google-adk"})
    
    err := rootCmd.Execute()
    assert.NoError(t, err)
    
    // Verify files created
    assert.FileExists(t, dir+"/test-agent/agent.yaml")
    assert.FileExists(t, dir+"/test-agent/Dockerfile")
    assert.DirExists(t, dir+"/test-agent/src")
}
```

## Resources

- `references/cobra-patterns.md` - Advanced Cobra patterns
- `assets/templates/` - Agent scaffolding templates
