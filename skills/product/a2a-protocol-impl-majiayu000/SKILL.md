---
name: a2a-protocol-impl
description: Implement Agent-to-Agent (A2A) protocol for inter-agent communication. Use for agent discovery, agent cards, task delegation, and multi-agent orchestration. Triggers on "A2A protocol", "agent-to-agent", "agent discovery", "agent card", "multi-agent", "agent delegation", "agent communication", or when implementing spec/api/017-a2a-protocol.md.
---

# A2A Protocol Implementation

## Overview

Implement the Agent-to-Agent (A2A) protocol for standardized inter-agent communication, enabling agents to discover, delegate, and collaborate with other agents.

## A2A Architecture

```
┌───────────────────────────────────────────────────────────────────┐
│                       A2A Protocol Flow                           │
├───────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌─────────────┐                              ┌─────────────┐    │
│  │  Agent A    │───── 1. Discover ──────────▶│  Registry   │    │
│  │  (Client)   │◀───── Agent Cards ──────────│             │    │
│  └─────────────┘                              └─────────────┘    │
│        │                                                          │
│        │ 2. Fetch /.well-known/agent.json                        │
│        ▼                                                          │
│  ┌─────────────┐                                                  │
│  │  Agent B    │                                                  │
│  │  (Server)   │                                                  │
│  └─────────────┘                                                  │
│        │                                                          │
│        │ 3. POST /tasks/send                                      │
│        ▼                                                          │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │ Task Execution: Streaming Updates via SSE                   │ │
│  │ event: task_progress | task_artifact | task_complete        │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

## Agent Card

### Agent Card Endpoint

Every A2A-compatible agent must expose:

```
GET /.well-known/agent.json
```

### Agent Card Schema

```json
{
  "name": "customer-support-agent",
  "description": "Handles customer inquiries, support tickets, and account management",
  "version": "1.0.0",
  "url": "https://customer-support.agents.agentstack.io",
  "provider": {
    "name": "AgentStack",
    "url": "https://agentstack.io"
  },
  "capabilities": [
    {
      "name": "answer_question",
      "description": "Answer customer questions about products and services",
      "inputSchema": {
        "type": "object",
        "properties": {
          "question": {"type": "string"},
          "context": {"type": "object"}
        },
        "required": ["question"]
      },
      "outputSchema": {
        "type": "object",
        "properties": {
          "answer": {"type": "string"},
          "confidence": {"type": "number"}
        }
      }
    },
    {
      "name": "create_ticket",
      "description": "Create a support ticket",
      "inputSchema": {
        "type": "object",
        "properties": {
          "subject": {"type": "string"},
          "description": {"type": "string"},
          "priority": {"enum": ["low", "medium", "high"]}
        },
        "required": ["subject", "description"]
      }
    }
  ],
  "authentication": {
    "schemes": ["bearer", "api_key"]
  },
  "rateLimit": {
    "requestsPerMinute": 60
  }
}
```

### Go Implementation

```go
// internal/api/handlers/a2a.go
package handlers

import (
    "github.com/gofiber/fiber/v2"
)

type AgentCard struct {
    Name           string         `json:"name"`
    Description    string         `json:"description"`
    Version        string         `json:"version"`
    URL            string         `json:"url"`
    Provider       Provider       `json:"provider,omitempty"`
    Capabilities   []Capability   `json:"capabilities"`
    Authentication AuthConfig     `json:"authentication,omitempty"`
    RateLimit      *RateLimitInfo `json:"rateLimit,omitempty"`
}

type Provider struct {
    Name string `json:"name"`
    URL  string `json:"url,omitempty"`
}

type Capability struct {
    Name         string      `json:"name"`
    Description  string      `json:"description"`
    InputSchema  interface{} `json:"inputSchema,omitempty"`
    OutputSchema interface{} `json:"outputSchema,omitempty"`
}

type AuthConfig struct {
    Schemes []string `json:"schemes"`
}

type RateLimitInfo struct {
    RequestsPerMinute int `json:"requestsPerMinute"`
}

func (h *A2AHandler) GetAgentCard(c *fiber.Ctx) error {
    agentID := c.Params("agentId")
    
    agent, err := h.agentService.Get(c.Context(), agentID)
    if err != nil {
        return handleError(c, err)
    }
    
    card := AgentCard{
        Name:        agent.Name,
        Description: agent.Description,
        Version:     agent.Version,
        URL:         fmt.Sprintf("https://%s.agents.agentstack.io", agent.Name),
        Provider: Provider{
            Name: "AgentStack",
            URL:  "https://agentstack.io",
        },
        Capabilities:   toCapabilities(agent.Capabilities),
        Authentication: AuthConfig{Schemes: []string{"bearer", "api_key"}},
        RateLimit:      &RateLimitInfo{RequestsPerMinute: 60},
    }
    
    return c.JSON(card)
}
```

## Task Execution

### Send Task

```
POST /tasks/send
Content-Type: application/json

{
  "task_id": "task_abc123",
  "capability": "answer_question",
  "input": {
    "question": "What is your return policy?",
    "context": {
      "user_id": "user_xyz",
      "tier": "premium"
    }
  },
  "callback_url": "https://caller-agent.agentstack.io/tasks/callback"
}
```

### Task Response

```json
{
  "task_id": "task_abc123",
  "status": "accepted",
  "stream_url": "https://agent.agentstack.io/tasks/task_abc123/stream"
}
```

### Stream Task Events

```
GET /tasks/{taskId}/stream
Accept: text/event-stream
```

Events:

```
event: task_progress
data: {"task_id": "task_abc123", "progress": 0.3, "message": "Searching knowledge base..."}

event: task_artifact
data: {"task_id": "task_abc123", "artifact_type": "text", "content": "Based on our policy..."}

event: task_complete
data: {"task_id": "task_abc123", "status": "completed", "output": {"answer": "...", "confidence": 0.95}}
```

### Go Implementation

```go
// internal/api/handlers/tasks.go
package handlers

type SendTaskRequest struct {
    TaskID      string      `json:"task_id" validate:"required"`
    Capability  string      `json:"capability" validate:"required"`
    Input       interface{} `json:"input" validate:"required"`
    CallbackURL string      `json:"callback_url,omitempty"`
}

type TaskResponse struct {
    TaskID    string `json:"task_id"`
    Status    string `json:"status"`
    StreamURL string `json:"stream_url,omitempty"`
}

func (h *A2AHandler) SendTask(c *fiber.Ctx) error {
    var req SendTaskRequest
    if err := c.BodyParser(&req); err != nil {
        return fiber.NewError(fiber.StatusBadRequest, "Invalid request")
    }
    
    agentID := c.Params("agentId")
    
    // Validate capability exists
    agent, err := h.agentService.Get(c.Context(), agentID)
    if err != nil {
        return handleError(c, err)
    }
    
    if !agent.HasCapability(req.Capability) {
        return fiber.NewError(fiber.StatusBadRequest, 
            fmt.Sprintf("Agent does not support capability: %s", req.Capability))
    }
    
    // Create task
    task, err := h.taskService.Create(c.Context(), TaskInput{
        TaskID:      req.TaskID,
        AgentID:     agentID,
        Capability:  req.Capability,
        Input:       req.Input,
        CallbackURL: req.CallbackURL,
    })
    if err != nil {
        return handleError(c, err)
    }
    
    // Start async execution
    go h.taskService.Execute(context.Background(), task)
    
    return c.Status(fiber.StatusAccepted).JSON(TaskResponse{
        TaskID:    task.ID,
        Status:    "accepted",
        StreamURL: fmt.Sprintf("/agents/%s/tasks/%s/stream", agentID, task.ID),
    })
}
```

## Agent Discovery

### Registry Service

```go
// internal/domain/registry/service.go
package registry

type AgentRegistry interface {
    Register(ctx context.Context, card AgentCard) error
    Unregister(ctx context.Context, agentID string) error
    Discover(ctx context.Context, query DiscoveryQuery) ([]AgentCard, error)
    Get(ctx context.Context, agentID string) (*AgentCard, error)
}

type DiscoveryQuery struct {
    Capability  string   `json:"capability,omitempty"`
    Tags        []string `json:"tags,omitempty"`
    MinVersion  string   `json:"min_version,omitempty"`
    ProjectID   string   `json:"project_id,omitempty"`
}
```

### Discovery Endpoint

```
GET /agents/discover?capability=answer_question&tags=support
```

Response:

```json
{
  "agents": [
    {
      "name": "customer-support-agent",
      "url": "https://customer-support.agents.agentstack.io",
      "capabilities": ["answer_question", "create_ticket"],
      "version": "1.0.0"
    },
    {
      "name": "faq-agent",
      "url": "https://faq.agents.agentstack.io",
      "capabilities": ["answer_question"],
      "version": "2.1.0"
    }
  ]
}
```

## A2A Client

### Go Client

```go
// pkg/a2a/client.go
package a2a

import (
    "context"
    "encoding/json"
    "net/http"
)

type Client struct {
    httpClient *http.Client
    baseURL    string
    apiKey     string
}

func NewClient(baseURL, apiKey string) *Client {
    return &Client{
        httpClient: &http.Client{Timeout: 30 * time.Second},
        baseURL:    baseURL,
        apiKey:     apiKey,
    }
}

// GetAgentCard fetches the agent card from the well-known endpoint
func (c *Client) GetAgentCard(ctx context.Context, agentURL string) (*AgentCard, error) {
    req, err := http.NewRequestWithContext(ctx, "GET", 
        agentURL+"/.well-known/agent.json", nil)
    if err != nil {
        return nil, err
    }
    
    resp, err := c.httpClient.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    var card AgentCard
    if err := json.NewDecoder(resp.Body).Decode(&card); err != nil {
        return nil, err
    }
    
    return &card, nil
}

// SendTask sends a task to an agent and returns the task response
func (c *Client) SendTask(ctx context.Context, agentURL string, task SendTaskRequest) (*TaskResponse, error) {
    body, _ := json.Marshal(task)
    
    req, err := http.NewRequestWithContext(ctx, "POST",
        agentURL+"/tasks/send", bytes.NewReader(body))
    if err != nil {
        return nil, err
    }
    
    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Authorization", "Bearer "+c.apiKey)
    
    resp, err := c.httpClient.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    var taskResp TaskResponse
    if err := json.NewDecoder(resp.Body).Decode(&taskResp); err != nil {
        return nil, err
    }
    
    return &taskResp, nil
}

// StreamTask streams task events
func (c *Client) StreamTask(ctx context.Context, streamURL string) (<-chan TaskEvent, error) {
    events := make(chan TaskEvent)
    
    req, err := http.NewRequestWithContext(ctx, "GET", streamURL, nil)
    if err != nil {
        return nil, err
    }
    
    req.Header.Set("Accept", "text/event-stream")
    req.Header.Set("Authorization", "Bearer "+c.apiKey)
    
    resp, err := c.httpClient.Do(req)
    if err != nil {
        return nil, err
    }
    
    go func() {
        defer close(events)
        defer resp.Body.Close()
        
        reader := bufio.NewReader(resp.Body)
        for {
            line, err := reader.ReadString('\n')
            if err != nil {
                return
            }
            
            if strings.HasPrefix(line, "data:") {
                var event TaskEvent
                json.Unmarshal([]byte(strings.TrimPrefix(line, "data:")), &event)
                events <- event
            }
        }
    }()
    
    return events, nil
}
```

## Multi-Agent Orchestration

### Supervisor Pattern

```go
// internal/domain/orchestration/supervisor.go
package orchestration

type Supervisor struct {
    agents    map[string]*a2a.Client
    registry  registry.AgentRegistry
}

func (s *Supervisor) DelegateTask(ctx context.Context, task Task) (*TaskResult, error) {
    // 1. Find capable agent
    agents, err := s.registry.Discover(ctx, DiscoveryQuery{
        Capability: task.RequiredCapability,
    })
    if err != nil {
        return nil, err
    }
    
    if len(agents) == 0 {
        return nil, ErrNoCapableAgent
    }
    
    // 2. Select best agent (could use load balancing, scoring, etc.)
    selected := s.selectAgent(agents)
    
    // 3. Send task
    client := s.getClient(selected.URL)
    taskResp, err := client.SendTask(ctx, selected.URL, SendTaskRequest{
        TaskID:     task.ID,
        Capability: task.RequiredCapability,
        Input:      task.Input,
    })
    if err != nil {
        return nil, err
    }
    
    // 4. Stream results
    events, err := client.StreamTask(ctx, taskResp.StreamURL)
    if err != nil {
        return nil, err
    }
    
    // 5. Collect result
    var result *TaskResult
    for event := range events {
        if event.Type == "task_complete" {
            result = &TaskResult{
                TaskID: task.ID,
                Output: event.Output,
                Status: "completed",
            }
        }
    }
    
    return result, nil
}
```

## Resources

- `references/task-states.md` - Task state machine
- `references/authentication-flows.md` - A2A authentication patterns
