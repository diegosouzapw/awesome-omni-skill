---
name: threat-modeling
description: Threat modeling methodologies (STRIDE, DREAD), attack trees, threat modeling as code, and integration with SDLC for proactive security design
argument-hint: "<component-or-feature-description>"
allowed-tools: Read, Grep, Glob, Write, Edit, Task, mcp__perplexity__*, mcp__context7__*, mcp__microsoft-learn__*
---

# Threat Modeling

Systematic approach to identifying, quantifying, and addressing security threats in software systems.

## When to Use This Skill

**Keywords:** threat modeling, STRIDE, DREAD, attack trees, security design, risk assessment, threat analysis, data flow diagram, trust boundary, attack surface, threat enumeration

**Use this skill when:**

- Designing new systems or features
- Conducting security architecture reviews
- Identifying potential attack vectors
- Prioritizing security investments
- Documenting security assumptions
- Integrating security into SDLC
- Creating threat models as code

## Quick Decision Tree

1. **Starting a threat model?** → Begin with [Threat Modeling Process](#threat-modeling-process)
2. **Identifying threats?** → Use [STRIDE methodology](#stride-methodology)
3. **Prioritizing threats?** → Apply [DREAD scoring](#dread-risk-scoring) or [Attack Trees](#attack-trees)
4. **Automating threat models?** → See [references/threat-modeling-tools.md](references/threat-modeling-tools.md)
5. **Specific architecture patterns?** → See [Architecture-Specific Threats](#architecture-specific-threats)

## Threat Modeling Process

```text
┌─────────────────────────────────────────────────────────────────┐
│                    THREAT MODELING WORKFLOW                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. DECOMPOSE        2. IDENTIFY         3. PRIORITIZE          │
│  ┌──────────┐       ┌──────────┐        ┌──────────┐           │
│  │ System   │──────▶│ Threats  │───────▶│ Risks    │           │
│  │ Model    │       │ (STRIDE) │        │ (DREAD)  │           │
│  └──────────┘       └──────────┘        └──────────┘           │
│       │                   │                   │                  │
│       ▼                   ▼                   ▼                  │
│  ┌──────────┐       ┌──────────┐        ┌──────────┐           │
│  │ DFD      │       │ Attack   │        │ Counter- │           │
│  │ Trust    │       │ Trees    │        │ measures │           │
│  │ Boundary │       │ Patterns │        │ Backlog  │           │
│  └──────────┘       └──────────┘        └──────────┘           │
│                                                                  │
│  4. DOCUMENT ──────▶ 5. VALIDATE ──────▶ 6. ITERATE            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Step 1: System Decomposition

Create a Data Flow Diagram (DFD) with these elements:

| Element | Symbol | Description |
|---------|--------|-------------|
| External Entity | Rectangle | Users, external systems |
| Process | Circle | Code that transforms data |
| Data Store | Parallel lines | Databases, files, caches |
| Data Flow | Arrow | Data movement |
| Trust Boundary | Dashed line | Security perimeter |

```csharp
// Example: E-commerce system decomposition
public enum ElementType
{
    ExternalEntity, Process, DataStore, DataFlow
}

/// <summary>Defines a security perimeter</summary>
public sealed record TrustBoundary(
    string Id,
    string Name,
    string Description,
    IReadOnlyList<string> Elements);  // IDs of contained elements

/// <summary>Data Flow Diagram element</summary>
public sealed record DfdElement
{
    public required string Id { get; init; }
    public required string Name { get; init; }
    public required ElementType ElementType { get; init; }
    public string? TrustBoundary { get; init; }
    public string Description { get; init; } = "";
}

/// <summary>Connection between elements</summary>
public sealed record DataFlow
{
    public required string Id { get; init; }
    public required string Source { get; init; }
    public required string Destination { get; init; }
    public required string DataType { get; init; }
    public required string Protocol { get; init; }
    public bool Encrypted { get; init; }
    public bool Authenticated { get; init; }
}

/// <summary>Complete system model for threat analysis</summary>
public sealed class SystemModel(string name)
{
    public string Name => name;
    private readonly Dictionary<string, DfdElement> _elements = new();
    private readonly List<DataFlow> _flows = [];
    private readonly List<TrustBoundary> _trustBoundaries = [];

    public IReadOnlyDictionary<string, DfdElement> Elements => _elements;
    public IReadOnlyList<DataFlow> Flows => _flows;
    public IReadOnlyList<TrustBoundary> TrustBoundaries => _trustBoundaries;

    public void AddElement(DfdElement element) => _elements[element.Id] = element;
    public void AddFlow(DataFlow flow) => _flows.Add(flow);
    public void AddTrustBoundary(TrustBoundary boundary) => _trustBoundaries.Add(boundary);

    /// <summary>Identify flows that cross trust boundaries - high-risk areas</summary>
    public IReadOnlyList<DataFlow> GetCrossBoundaryFlows()
    {
        var crossBoundary = new List<DataFlow>();
        foreach (var flow in _flows)
        {
            var sourceBoundary = _elements[flow.Source].TrustBoundary;
            var destBoundary = _elements[flow.Destination].TrustBoundary;
            if (sourceBoundary != destBoundary)
                crossBoundary.Add(flow);
        }
        return crossBoundary;
    }
}

// Example usage
var model = new SystemModel("E-Commerce Platform");

// Define elements
model.AddElement(new DfdElement
{
    Id = "user",
    Name = "Customer",
    ElementType = ElementType.ExternalEntity,
    TrustBoundary = "internet",
    Description = "End user accessing via browser"
});

model.AddElement(new DfdElement
{
    Id = "web_app",
    Name = "Web Application",
    ElementType = ElementType.Process,
    TrustBoundary = "dmz",
    Description = "Frontend web server"
});

model.AddElement(new DfdElement
{
    Id = "api",
    Name = "API Gateway",
    ElementType = ElementType.Process,
    TrustBoundary = "internal",
    Description = "Backend API services"
});

model.AddElement(new DfdElement
{
    Id = "db",
    Name = "Database",
    ElementType = ElementType.DataStore,
    TrustBoundary = "internal",
    Description = "Customer and order data"
});

// Define flows
model.AddFlow(new DataFlow
{
    Id = "f1",
    Source = "user",
    Destination = "web_app",
    DataType = "HTTP Request",
    Protocol = "HTTPS",
    Encrypted = true,
    Authenticated = false
});

// Find high-risk flows
var riskyFlows = model.GetCrossBoundaryFlows();
```

## STRIDE Methodology

STRIDE is a threat classification framework for systematic threat identification:

| Category | Threat | Security Property | Example |
|----------|--------|-------------------|---------|
| **S**poofing | Impersonating someone/something | Authentication | Stolen credentials, session hijacking |
| **T**ampering | Modifying data or code | Integrity | SQL injection, file modification |
| **R**epudiation | Denying actions | Non-repudiation | Missing audit logs |
| **I**nformation Disclosure | Exposing information | Confidentiality | Data breach, verbose errors |
| **D**enial of Service | Disrupting availability | Availability | Resource exhaustion, DDoS |
| **E**levation of Privilege | Gaining unauthorized access | Authorization | Privilege escalation, IDOR |

### STRIDE-per-Element Analysis

Apply STRIDE to each DFD element:

```csharp
using System.Collections.Frozen;

public enum StrideCategory
{
    Spoofing, Tampering, Repudiation, InformationDisclosure, DenialOfService, ElevationOfPrivilege
}

/// <summary>Which STRIDE categories apply to which element types</summary>
public static class StrideApplicability
{
    public static readonly FrozenDictionary<ElementType, StrideCategory[]> Map =
        new Dictionary<ElementType, StrideCategory[]>
        {
            [ElementType.ExternalEntity] =
                [StrideCategory.Spoofing, StrideCategory.Repudiation],
            [ElementType.Process] =
                [StrideCategory.Spoofing, StrideCategory.Tampering, StrideCategory.Repudiation,
                 StrideCategory.InformationDisclosure, StrideCategory.DenialOfService,
                 StrideCategory.ElevationOfPrivilege],
            [ElementType.DataStore] =
                [StrideCategory.Tampering, StrideCategory.Repudiation,
                 StrideCategory.InformationDisclosure, StrideCategory.DenialOfService],
            [ElementType.DataFlow] =
                [StrideCategory.Tampering, StrideCategory.InformationDisclosure,
                 StrideCategory.DenialOfService]
        }.ToFrozenDictionary();
}

/// <summary>Identified threat</summary>
public sealed record Threat
{
    public required string Id { get; init; }
    public required StrideCategory Category { get; init; }
    public required string ElementId { get; init; }
    public required string Title { get; init; }
    public required string Description { get; init; }
    public required string AttackVector { get; init; }
    public string Impact { get; init; } = "";
    public string Likelihood { get; init; } = "Medium";
    public IReadOnlyList<string> Mitigations { get; init; } = [];
}

/// <summary>Threat template for generation</summary>
public sealed record ThreatTemplate(
    string TitleFormat, string DescriptionFormat, string AttackVector, string[] Mitigations);

/// <summary>Systematic STRIDE threat identification</summary>
public sealed class StrideAnalyzer(SystemModel model)
{
    private readonly List<Threat> _threats = [];
    private int _threatCounter;

    private static readonly Dictionary<(ElementType, StrideCategory), ThreatTemplate> Templates = new()
    {
        [(ElementType.Process, StrideCategory.Spoofing)] = new(
            "Spoofing of {0}",
            "Attacker impersonates {0} to gain unauthorized access",
            "Credential theft, session hijacking, certificate forgery",
            ["Strong authentication", "Certificate pinning", "Session management"]),
        [(ElementType.Process, StrideCategory.Tampering)] = new(
            "Tampering with {0}",
            "Attacker modifies data processed by {0}",
            "Input manipulation, code injection, memory corruption",
            ["Input validation", "Integrity checks", "Code signing"]),
        [(ElementType.DataStore, StrideCategory.InformationDisclosure)] = new(
            "Information disclosure from {0}",
            "Sensitive data exposed from {0}",
            "SQL injection, misconfiguration, backup exposure",
            ["Encryption at rest", "Access controls", "Data masking"])
    };

    /// <summary>Perform STRIDE analysis on all elements</summary>
    public IReadOnlyList<Threat> Analyze()
    {
        // Analyze elements
        foreach (var element in model.Elements.Values)
        {
            if (StrideApplicability.Map.TryGetValue(element.ElementType, out var categories))
            {
                foreach (var category in categories)
                    _threats.AddRange(IdentifyThreats(element, category));
            }
        }

        // Analyze data flows
        if (StrideApplicability.Map.TryGetValue(ElementType.DataFlow, out var flowCategories))
        {
            foreach (var flow in model.Flows)
            {
                foreach (var category in flowCategories)
                    _threats.AddRange(IdentifyFlowThreats(flow, category));
            }
        }

        return _threats;
    }

    private IEnumerable<Threat> IdentifyThreats(DfdElement element, StrideCategory category)
    {
        var key = (element.ElementType, category);
        if (!Templates.TryGetValue(key, out var template))
            yield break;

        _threatCounter++;
        yield return new Threat
        {
            Id = $"T{_threatCounter:D3}",
            Category = category,
            ElementId = element.Id,
            Title = string.Format(template.TitleFormat, element.Name),
            Description = string.Format(template.DescriptionFormat, element.Name),
            AttackVector = template.AttackVector,
            Mitigations = template.Mitigations
        };
    }

    private IEnumerable<Threat> IdentifyFlowThreats(DataFlow flow, StrideCategory category)
    {
        if (category == StrideCategory.InformationDisclosure && !flow.Encrypted)
        {
            _threatCounter++;
            yield return new Threat
            {
                Id = $"T{_threatCounter:D3}",
                Category = category,
                ElementId = flow.Id,
                Title = $"Unencrypted data flow: {flow.Source} -> {flow.Destination}",
                Description = $"Data ({flow.DataType}) transmitted without encryption",
                AttackVector = "Network sniffing, man-in-the-middle",
                Impact = "High - Data exposure",
                Mitigations = ["Enable TLS", "Use VPN", "Encrypt at application layer"]
            };
        }

        if (category == StrideCategory.Tampering && !flow.Authenticated)
        {
            _threatCounter++;
            yield return new Threat
            {
                Id = $"T{_threatCounter:D3}",
                Category = category,
                ElementId = flow.Id,
                Title = $"Unauthenticated data flow: {flow.Source} -> {flow.Destination}",
                Description = "Data flow lacks authentication - source cannot be verified",
                AttackVector = "Message injection, replay attacks",
                Impact = "Medium - Data integrity compromise",
                Mitigations = ["Mutual TLS", "Message signing", "API authentication"]
            };
        }
    }
}
```

**For detailed STRIDE analysis with examples**, see [references/stride-methodology.md](references/stride-methodology.md).

## DREAD Risk Scoring

DREAD provides quantitative risk assessment for prioritization:

| Factor | Description | Scale |
|--------|-------------|-------|
| **D**amage | Impact if exploited | 1-10 |
| **R**eproducibility | Ease of reproducing attack | 1-10 |
| **E**xploitability | Effort required to exploit | 1-10 |
| **A**ffected Users | Scope of impact | 1-10 |
| **D**iscoverability | Likelihood of finding vulnerability | 1-10 |

```csharp
/// <summary>DREAD risk scoring</summary>
public readonly struct DreadScore
{
    public int Damage { get; }           // 1-10
    public int Reproducibility { get; }  // 1-10
    public int Exploitability { get; }   // 1-10
    public int AffectedUsers { get; }    // 1-10
    public int Discoverability { get; }  // 1-10

    public DreadScore(int damage, int reproducibility, int exploitability,
                      int affectedUsers, int discoverability)
    {
        ValidateRange(damage, nameof(damage));
        ValidateRange(reproducibility, nameof(reproducibility));
        ValidateRange(exploitability, nameof(exploitability));
        ValidateRange(affectedUsers, nameof(affectedUsers));
        ValidateRange(discoverability, nameof(discoverability));

        Damage = damage;
        Reproducibility = reproducibility;
        Exploitability = exploitability;
        AffectedUsers = affectedUsers;
        Discoverability = discoverability;
    }

    private static void ValidateRange(int value, string name)
    {
        if (value is < 1 or > 10)
            throw new ArgumentOutOfRangeException(name, $"{name} must be between 1-10");
    }

    /// <summary>Calculate average DREAD score</summary>
    public double Total => (Damage + Reproducibility + Exploitability +
                           AffectedUsers + Discoverability) / 5.0;

    /// <summary>Categorize risk level</summary>
    public string RiskLevel => Total switch
    {
        >= 8 => "Critical",
        >= 6 => "High",
        >= 4 => "Medium",
        _ => "Low"
    };
}

/// <summary>Prioritize threats using DREAD scoring</summary>
public sealed class ThreatPrioritizer
{
    private readonly List<(Threat Threat, DreadScore Score)> _scoredThreats = [];

    public void ScoreThreat(Threat threat, DreadScore score) =>
        _scoredThreats.Add((threat, score));

    /// <summary>Return threats sorted by risk (highest first)</summary>
    public IReadOnlyList<(Threat Threat, DreadScore Score)> GetPrioritizedList() =>
        _scoredThreats.OrderByDescending(x => x.Score.Total).ToList();

    /// <summary>Generate risk matrix summary</summary>
    public Dictionary<string, List<string>> GenerateRiskMatrix()
    {
        var matrix = new Dictionary<string, List<string>>
        {
            ["Critical"] = [], ["High"] = [], ["Medium"] = [], ["Low"] = []
        };
        foreach (var (threat, score) in _scoredThreats)
            matrix[score.RiskLevel].Add(threat.Id);
        return matrix;
    }
}

// Example scoring
var sqlInjectionScore = new DreadScore(
    damage: 9,           // Full database compromise
    reproducibility: 8,  // Easily reproducible with tools
    exploitability: 7,   // Well-known techniques
    affectedUsers: 10,   // All users potentially affected
    discoverability: 6   // Requires testing to find
);

Console.WriteLine($"Risk Score: {sqlInjectionScore.Total}");    // 8.0
Console.WriteLine($"Risk Level: {sqlInjectionScore.RiskLevel}"); // Critical
```

### Alternative: CVSS-Based Scoring

For compatibility with industry standards, map to CVSS:

```csharp
/// <summary>CVSS 3.1 Base Score components</summary>
public sealed record CvssVector
{
    public required string AttackVector { get; init; }       // N, A, L, P
    public required string AttackComplexity { get; init; }   // L, H
    public required string PrivilegesRequired { get; init; } // N, L, H
    public required string UserInteraction { get; init; }    // N, R
    public required string Scope { get; init; }              // U, C
    public required string Confidentiality { get; init; }    // N, L, H
    public required string Integrity { get; init; }          // N, L, H
    public required string Availability { get; init; }       // N, L, H

    public string ToVectorString() =>
        $"CVSS:3.1/AV:{AttackVector}/AC:{AttackComplexity}/" +
        $"PR:{PrivilegesRequired}/UI:{UserInteraction}/" +
        $"S:{Scope}/C:{Confidentiality}/I:{Integrity}/A:{Availability}";
}
```

## Attack Trees

Attack trees visualize how an attacker might achieve a goal:

```text
                    ┌─────────────────────┐
                    │ Steal Customer Data │ (Goal)
                    └─────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              ▼               ▼               ▼
      ┌───────────┐   ┌───────────┐   ┌───────────┐
      │ SQL       │   │ Phishing  │   │ Insider   │
      │ Injection │   │ Attack    │   │ Threat    │
      │ [OR]      │   │ [OR]      │   │ [OR]      │
      └───────────┘   └───────────┘   └───────────┘
            │               │               │
      ┌─────┴─────┐   ┌─────┴─────┐   ┌─────┴─────┐
      │           │   │           │   │           │
      ▼           ▼   ▼           ▼   ▼           ▼
  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐
  │Find   │ │Exploit│ │Send   │ │Harvest│ │Bribe  │ │Access │
  │Vuln   │ │Input  │ │Fake   │ │Creds  │ │Staff  │ │After  │
  │Endpoint│ │Field  │ │Emails │ │Site   │ │       │ │Hours  │
  │[AND]  │ │[AND]  │ │[AND]  │ │[AND]  │ │[OR]   │ │[OR]   │
  └───────┘ └───────┘ └───────┘ └───────┘ └───────┘ └───────┘
```

```csharp
public enum NodeOperator
{
    And,  // All children must succeed
    Or    // Any child can succeed
}

/// <summary>Node in attack tree</summary>
public sealed class AttackNode
{
    public required string Id { get; init; }
    public required string Description { get; init; }
    public NodeOperator Operator { get; init; } = NodeOperator.Or;
    public double Cost { get; init; }           // Estimated attack cost
    public double Probability { get; init; } = 0.5;  // Success probability
    public string SkillRequired { get; init; } = "Medium";
    public List<AttackNode> Children { get; } = [];
    public List<string> Countermeasures { get; init; } = [];

    public void AddChild(AttackNode child) => Children.Add(child);

    /// <summary>Calculate probability based on operator and children</summary>
    public double CalculateProbability()
    {
        if (Children.Count == 0)
            return Probability;

        var childProbs = Children.Select(c => c.CalculateProbability()).ToList();

        if (Operator == NodeOperator.And)
        {
            // All must succeed - multiply probabilities
            return childProbs.Aggregate(1.0, (acc, p) => acc * p);
        }
        else // OR
        {
            // Any can succeed - 1 - (all fail)
            return 1.0 - childProbs.Aggregate(1.0, (acc, p) => acc * (1 - p));
        }
    }

    /// <summary>Calculate minimum attack cost</summary>
    public double CalculateMinCost()
    {
        if (Children.Count == 0)
            return Cost;

        var childCosts = Children.Select(c => c.CalculateMinCost()).ToList();

        if (Operator == NodeOperator.And)
        {
            // Must complete all - sum costs
            return childCosts.Sum();
        }
        else // OR
        {
            // Choose cheapest path
            return childCosts.Min();
        }
    }
}

/// <summary>Analyze attack trees for risk assessment</summary>
public sealed class AttackTreeAnalyzer(AttackNode root)
{
    /// <summary>Find the least expensive attack path</summary>
    public IReadOnlyList<AttackNode> FindCheapestPath() => FindPath(root, minimizeCost: true);

    /// <summary>Find the most probable attack path</summary>
    public IReadOnlyList<AttackNode> FindMostLikelyPath() => FindPath(root, minimizeCost: false);

    private static List<AttackNode> FindPath(AttackNode node, bool minimizeCost)
    {
        var path = new List<AttackNode> { node };

        if (node.Children.Count == 0)
            return path;

        if (node.Operator == NodeOperator.And)
        {
            // Must traverse all children
            foreach (var child in node.Children)
                path.AddRange(FindPath(child, minimizeCost));
        }
        else // OR
        {
            // Choose best child
            var bestChild = minimizeCost
                ? node.Children.MinBy(c => c.CalculateMinCost())!
                : node.Children.MaxBy(c => c.CalculateProbability())!;
            path.AddRange(FindPath(bestChild, minimizeCost));
        }

        return path;
    }

    /// <summary>Collect all countermeasures from tree</summary>
    public HashSet<string> GetAllCountermeasures() => CollectCountermeasures(root);

    private static HashSet<string> CollectCountermeasures(AttackNode node)
    {
        var measures = new HashSet<string>(node.Countermeasures);
        foreach (var child in node.Children)
            measures.UnionWith(CollectCountermeasures(child));
        return measures;
    }
}

// Example: Build attack tree
var rootNode = new AttackNode
{
    Id = "goal",
    Description = "Steal Customer Data",
    Operator = NodeOperator.Or
};

var sqlInjection = new AttackNode
{
    Id = "sqli",
    Description = "SQL Injection Attack",
    Operator = NodeOperator.And,
    Countermeasures = ["Parameterized queries", "WAF", "Input validation"]
};

sqlInjection.AddChild(new AttackNode
{
    Id = "find_vuln",
    Description = "Find vulnerable endpoint",
    Cost = 100,
    Probability = 0.7,
    SkillRequired = "Medium"
});

sqlInjection.AddChild(new AttackNode
{
    Id = "exploit",
    Description = "Exploit injection point",
    Cost = 200,
    Probability = 0.8,
    SkillRequired = "High"
});

rootNode.AddChild(sqlInjection);

// Analyze
var analyzer = new AttackTreeAnalyzer(rootNode);
Console.WriteLine($"Overall attack probability: {rootNode.CalculateProbability():P2}");
Console.WriteLine($"Minimum attack cost: ${rootNode.CalculateMinCost()}");
Console.WriteLine($"Countermeasures needed: {string.Join(", ", analyzer.GetAllCountermeasures())}");
```

## Architecture-Specific Threats

### Microservices Architecture

| Component | Key Threats | Mitigations |
|-----------|-------------|-------------|
| API Gateway | DDoS, injection, auth bypass | Rate limiting, WAF, OAuth |
| Service Mesh | mTLS bypass, sidecar compromise | Certificate rotation, network policies |
| Message Queue | Message tampering, replay | Message signing, idempotency |
| Service Discovery | Registry poisoning | Secure registration, health checks |

### Serverless Architecture

| Component | Key Threats | Mitigations |
|-----------|-------------|-------------|
| Functions | Cold start attacks, injection | Input validation, minimal permissions |
| Event Sources | Event injection, DoS | Source validation, rate limiting |
| Shared Tenancy | Noisy neighbor, data leakage | Isolation, encryption |

### Container/Kubernetes Architecture

| Component | Key Threats | Mitigations |
|-----------|-------------|-------------|
| Container Runtime | Escape, privilege escalation | Rootless, seccomp, AppArmor |
| Orchestrator | API server compromise | RBAC, audit logging, network policies |
| Registry | Image tampering, supply chain | Image signing, vulnerability scanning |

## SDLC Integration

### When to Threat Model

| Phase | Activity | Output |
|-------|----------|--------|
| Design | Initial threat model | Threat register, DFD |
| Development | Update for changes | Updated threats, security tests |
| Code Review | Verify mitigations | Security checklist |
| Testing | Validate mitigations | Penetration test plan |
| Release | Final review | Risk acceptance, residual risks |
| Operations | Incident analysis | Updated threat model |

### Lightweight Threat Modeling

For agile environments, use rapid threat modeling:

```yaml
# threat-model.yaml - Minimal threat model per feature
feature: User Authentication
date: 2024-01-15
author: security-team

assets:
  - name: User credentials
    classification: Confidential
  - name: Session tokens
    classification: Internal

threats:
  - id: AUTH-001
    category: Spoofing
    description: Credential stuffing attack
    risk: High
    mitigation: Rate limiting, MFA, breach detection

  - id: AUTH-002
    category: Information Disclosure
    description: Token exposure in logs
    risk: Medium
    mitigation: Token redaction, structured logging

mitigations_implemented:
  - Argon2id password hashing
  - JWT with short expiration
  - Refresh token rotation

open_risks:
  - Legacy systems using MD5 (migration planned Q2)
```

## Security Checklist

Before finalizing threat model:

- [ ] All trust boundaries identified
- [ ] STRIDE applied to each element
- [ ] Data flows across boundaries encrypted
- [ ] Authentication at trust boundary crossings
- [ ] Risks prioritized (DREAD/CVSS)
- [ ] Mitigations mapped to threats
- [ ] Residual risks documented
- [ ] Model reviewed by stakeholders
- [ ] Integration with issue tracking

## References

- [STRIDE Methodology Deep Dive](references/stride-methodology.md) - Detailed analysis with examples
- [Threat Modeling Tools](references/threat-modeling-tools.md) - pytm, threagile, automation

## User-Facing Interface

When invoked directly by the user, this skill generates a structured threat model using STRIDE methodology.

### Execution Workflow

1. **Parse Arguments** - Extract the component or feature description from `$ARGUMENTS`. If no arguments provided, ask the user what to threat model.
2. **Understand the System** - Identify assets, data flows, trust boundaries, and entry points.
3. **Create Data Flow Diagram** - Generate a DFD in Mermaid notation showing components, data flows, and trust boundaries.
4. **Apply STRIDE Analysis** - Analyze each component for Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, and Elevation of Privilege threats.
5. **Build Attack Trees** - Create goal-oriented attack trees for high-value assets.
6. **Score Risks** - Apply DREAD methodology (Damage, Reproducibility, Exploitability, Affected Users, Discoverability) to prioritize threats.
7. **Recommend Controls** - Provide prioritized security control recommendations (Must Have / Should Have / Nice to Have).

## Version History

- **v1.0.0** (2025-12-26): Initial release with STRIDE, DREAD, attack trees, architecture patterns

---

**Last Updated:** 2025-12-26
