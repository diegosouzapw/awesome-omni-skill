---
name: iac-diagram-generator
description: Analyzes Infrastructure as Code files (Terraform, CloudFormation, Kubernetes, Docker Compose) and generates visual architecture diagrams. Use when analyzing infrastructure code, designing cloud architectures, or when the user requests architecture diagrams from IaC.
allowed-tools: Read, Bash, Glob, Grep
---

# IaC Architecture Diagram Generator

Analyzes Infrastructure as Code repositories and generates professional architecture diagrams using Nano Banana Pro. Supports Terraform, CloudFormation, Kubernetes, Docker Compose, Pulumi, and other common IaC formats.

## Core Philosophy

Infrastructure diagrams should accurately represent the logical architecture, resource relationships, and security boundaries defined in your IaC. This skill parses IaC files to extract resources, dependencies, and hierarchical structures, then generates diagrams that follow cloud architecture best practices.

## Workflow

When a user requests an architecture diagram from IaC files, follow these steps:

### Step 1: Discover IaC Files

Use Glob to identify IaC files in the target directory:

- **Terraform**: `*.tf`, `*.tfvars`
- **CloudFormation**: `*.yaml`, `*.yml`, `*.json`, `*.template`
- **Kubernetes**: `*.yaml`, `*.yml` (in manifests/, k8s/, kube/ directories)
- **Docker Compose**: `docker-compose.yaml`, `docker-compose.yml`
- **Pulumi**: `*.ts`, `*.py`, `*.go` (with Pulumi imports)
- **Azure ARM**: `*.json` (with ARM schema)
- **GCP Deployment Manager**: `*.yaml`, `*.jinja`, `*.py`

If no specific file is mentioned, search the current directory recursively.

### Step 2: Validate and Parse IaC Files

Run the appropriate parser script based on file type. The parser accepts **local paths or GitHub repository URLs**.

**Local files/directories:**
```bash
# Terraform
python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py terraform path/to/terraform/dir

# CloudFormation
python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py cloudformation path/to/template.yaml

# Kubernetes
python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py kubernetes path/to/manifests/

# Docker Compose
python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py docker-compose path/to/docker-compose.yaml
```

**GitHub repositories (cloned automatically):**
```bash
# Clone entire repo and parse
python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py terraform https://github.com/user/repo

# Clone and parse specific subdirectory
python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py terraform https://github.com/user/repo/tree/main/infrastructure

# Short format also works
python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py terraform github.com/user/repo
```

**Supported GitHub URL formats:**
- `https://github.com/user/repo`
- `https://github.com/user/repo/tree/branch/path/to/dir`
- `github.com/user/repo`
- `git@github.com:user/repo`

The parser automatically clones the repo to a temp directory, parses the files, and cleans up after.

The parser will return a JSON structure containing:
- Resources (compute, networking, storage, security)
- Dependencies and relationships
- Hierarchical organization (VPCs, subnets, namespaces)
- Connection types (public internet, private, managed services)

### Step 3: Analyze the Resource Graph

Review the parsed structure to understand:
- **Hierarchy**: VPC > Availability Zones > Subnets > Resources
- **Resource Types**: Compute (EC2, Lambda), Networking (VPC, Load Balancers), Storage (S3, RDS), Security (IAM, Security Groups)
- **Dependencies**: Which resources depend on others (explicit and implicit)
- **Connections**: How resources communicate (HTTP, database connections, message queues)
- **Security Boundaries**: VPCs, subnets, security groups, network ACLs

### Step 4: Generate Nano Banana Pro Diagram Prompt

Create a detailed, structured prompt for Nano Banana Pro that describes the architecture diagram using natural language. Follow these guidelines carefully to ensure **consistent, visually stunning results**.

#### Visual Design System

Every diagram MUST follow this standardized visual template for consistency:

**Canvas & Outer Margins:**
- "A professional 16:9 landscape architecture diagram"
- "The canvas has a clean white outer margin (at least 60 pixels on all sides) creating breathing room before the page edge"
- "This outer margin ensures no content touches or approaches the canvas boundaries"

**Border Frame (Inside the outer margin):**
- "A subtle rounded-corner border with a thin dark gray stroke frames all diagram content"
- "Everything - header, zones, connections, legend, and logos - is contained INSIDE this border"
- "An inner padding of 30 pixels separates all content from the border edge"

**Header Section (Inside the frame, top 12%):**
- "A gradient header bar spans the full width inside the frame, transitioning from deep navy blue (#1a365d) on the left to teal (#0d9488) on the right"
- "The title '[ARCHITECTURE NAME]' appears in large, bold white sans-serif text (like Inter or SF Pro) centered in the header"
- "A subtitle below reads '[Brief Description]' in smaller light gray text"

**Main Canvas (Inside the frame, middle 78%):**
- "The main area has a very light cool gray background (#f8fafc)"
- "Architectural zones are represented as softly colored rectangular regions with rounded corners and subtle drop shadows"
- "Adequate spacing between zones prevents crowding"

**Footer Section (Inside the frame, bottom 10%):**
- "A thin footer bar INSIDE the frame contains a compact legend with small icon samples and labels"
- "Cloud provider logo (AWS/Azure/GCP) appears discretely in the bottom-right corner, INSIDE the border frame"
- "The legend and logo have the same inner padding from the border as other content"

#### Icon & Visual Style

**Use Isometric 3D Style (NOT flat official icons):**
- "All resource icons are rendered in a clean isometric 3D style with subtle shadows"
- "Icons have a consistent 30-degree isometric angle and soft gradient fills"
- "Each icon type uses a distinct, harmonious color from a modern tech palette"
- "Icons are crisp, detailed, and visually appealing - like high-end infographic illustrations"

**Color Palette for Icons:**
- Compute (EC2, Lambda, Containers): Warm orange (#f97316) to coral
- Networking (VPC, Load Balancers, Gateways): Purple (#8b5cf6) to indigo
- Storage (S3, EBS, EFS): Green (#22c55e) to emerald
- Database (RDS, DynamoDB, ElastiCache): Blue (#3b82f6) to sky blue
- Security (IAM, KMS, WAF): Red (#ef4444) to rose
- Analytics (Athena, Glue, Kinesis): Teal (#14b8a6) to cyan

**Zone/Layer Backgrounds:**
- Public/Internet zone: Very light blue tint (#eff6ff) with dashed blue border
- Private/Application zone: Very light green tint (#f0fdf4) with dashed green border
- Data/Database zone: Very light amber tint (#fffbeb) with dashed amber border
- Security/Governance zone: Very light slate tint (#f1f5f9) with dashed gray border

#### Hierarchical Organization

Describe the architecture from outermost to innermost layers:

1. **Cloud Provider / Region Level**
   - "The diagram shows an AWS architecture in the us-east-1 region"

2. **VPC / Virtual Network Level**
   - "A VPC labeled 'Production VPC (10.0.0.0/16)' contains all resources"
   - Use rectangular containers with dashed borders for VPCs

3. **Availability Zone / Subnet Level**
   - "Inside the VPC, there are three subnets arranged horizontally"
   - "A public subnet (10.0.1.0/24) on the left contains..."
   - "A private subnet (10.0.2.0/24) in the center contains..."
   - "A database subnet (10.0.3.0/24) on the right contains..."

4. **Resource Level**
   - Describe each resource with its icon type and label
   - "An Application Load Balancer icon labeled 'web-alb'"
   - "Three EC2 instance icons labeled 'web-1', 'web-2', 'web-3'"

#### Resource Representation

For each resource type, use appropriate descriptions:

**Compute**:
- "EC2 instance icons" (orange/brown server icons)
- "Lambda function icons" (orange lambda symbols)
- "Container icons for ECS tasks"

**Networking**:
- "Load balancer icon" (purple/blue distribution icon)
- "VPC router icon"
- "Internet gateway icon" (world/globe icon)
- "NAT gateway icon"

**Storage**:
- "S3 bucket icon" (green/orange bucket)
- "RDS database icon" (blue cylinder)
- "ElastiCache icon" (orange cache symbol)

**Security**:
- "Security group represented as a dotted border around resources"
- "IAM role icon" (orange key/badge)
- "WAF/firewall icon"

#### Connections and Data Flow

Describe how resources connect using consistent visual styling:

**Connection Arrow Styles:**
- "All connection arrows are smooth, curved bezier paths (not straight lines) with subtle shadows"
- "Arrow heads are small, elegant triangles"
- "Connection lines have a consistent 3px stroke width"

**Connection Color Coding:**
- Internet/External traffic: Bright blue (#3b82f6) solid arrows
- Internal HTTP/REST: Purple (#8b5cf6) solid arrows
- Database connections: Amber (#f59e0b) dashed arrows
- Async/Queue messages: Green (#22c55e) dotted arrows
- Security/Auth flows: Red (#ef4444) solid arrows

**Connection Labels:**
- "Each arrow has a small pill-shaped label with white background and the protocol/port (e.g., 'HTTPS 443', 'PostgreSQL 5432')"
- "Labels are positioned along the arrow path, not overlapping other elements"

**Data Flow Direction:**
- "The primary data flow moves left-to-right, with internet entry on the left"
- "Vertical flows indicate writes going down, reads going up"
- "Return paths are shown as lighter, thinner arrows parallel to the main flow"

#### Labels and Text

**Critical - Specify Exact Text**:
- **Always include a title at the top**: "At the very top of the diagram is a prominent title reading '[Architecture Name]'"
- **Always include a subtitle/description**: "with a subtitle below it stating '[Brief Description]'"
- Enclose all labels in single quotes within the prompt
- "The VPC container is labeled 'Production VPC (10.0.0.0/16)'"
- "The Load Balancer is labeled 'web-alb'"
- "The database is labeled 'PostgreSQL RDS (db.t3.medium)'"

#### Layout and Composition

**Orientation**:
- "Left-to-right flow showing internet traffic entering from the left"
- "Top-to-bottom hierarchy with VPC at the top"

**Spacing and Clarity**:
- "Resources are evenly spaced with clear separation"
- "Connection arrows do not overlap"
- "Labels are positioned next to their resources without overlapping other elements"

### Step 5: Example Prompts for Different Architectures

**Three-Tier Web Application**:
```
A professional 16:9 landscape cloud architecture diagram in a stunning modern infographic style.

CANVAS AND MARGINS:
The image has a generous white outer margin (at least 60 pixels on all sides) so no content approaches the page edges. Inside this margin, a subtle rounded-corner border with a thin charcoal stroke frames the entire diagram. All content is contained inside this border with comfortable inner padding.

HEADER (inside the frame):
A gradient header bar spans the top inside the frame, transitioning from deep navy blue on the left to teal on the right. The title 'Three-Tier Web Application' appears in large, bold white sans-serif text centered in the header. A subtitle below reads 'AWS Production Environment • us-east-1' in smaller light blue text.

MAIN CANVAS:
The main area has a very light cool gray background. The layout flows left-to-right showing the request path from internet to database. Generous spacing between all elements.

VISUAL STYLE:
All icons are rendered in a clean isometric 3D style with subtle drop shadows and soft gradient fills. Icons are crisp, detailed, and visually appealing like high-end tech infographic illustrations. Each resource type uses harmonious colors from a modern palette.

ARCHITECTURE ZONES (arranged left to right):

ZONE 1 - Internet Entry (far left):
A small cloud icon labeled 'Internet' with a globe symbol. A bright blue curved arrow flows rightward.

ZONE 2 - Public Layer (light blue tinted rectangle with dashed blue border, labeled 'Public Subnet 10.0.1.0/24'):
- A purple isometric Internet Gateway icon with network symbol
- A purple isometric Application Load Balancer icon labeled 'web-alb' with a circular distribution symbol
- Blue curved arrows connect them showing HTTPS flow

ZONE 3 - Application Layer (light green tinted rectangle with dashed green border, labeled 'Private Subnet 10.0.2.0/24'):
- Three orange isometric EC2 server icons arranged in a clean row, each labeled 'web-1', 'web-2', 'web-3'
- The servers are grouped within a subtle dotted security boundary labeled 'web-sg'
- Purple curved arrows from the load balancer fan out to each server

ZONE 4 - Data Layer (light amber tinted rectangle with dashed amber border, labeled 'Database Subnet 10.0.3.0/24'):
- A blue isometric RDS database cylinder icon with a subtle glow, labeled 'PostgreSQL Primary'
- A smaller replica icon labeled 'Read Replica' below it
- Amber dashed curved arrows connect from the EC2 instances with pill-shaped labels reading 'PostgreSQL 5432'

FOOTER (inside the frame, at the bottom):
A thin footer bar inside the border frame contains a compact legend showing icon types with labels. The AWS logo appears discretely in the bottom-right corner, also inside the frame. Everything is well within the border with no content touching or extending beyond it.

The overall aesthetic is clean, modern, and visually stunning - suitable for executive presentations and technical documentation alike.
```

**Microservices on Kubernetes**:
```
A professional 16:9 landscape Kubernetes architecture diagram in a stunning modern infographic style.

CANVAS AND MARGINS:
The image has a generous white outer margin (at least 60 pixels on all sides) so no content approaches the page edges. Inside this margin, a subtle rounded-corner border with a thin charcoal stroke frames the entire diagram. All content is contained inside this border.

HEADER (inside the frame):
A gradient header bar spans the top inside the frame, transitioning from deep indigo on the left to violet on the right. The title 'Microservices on Kubernetes' appears in large, bold white sans-serif text. A subtitle reads 'EKS Production Cluster • Multi-Namespace Architecture' in smaller light purple text.

MAIN CANVAS:
Light cool gray background (#f8fafc). The diagram represents a Kubernetes cluster as a large rounded rectangle with a subtle shadow. Generous spacing between elements.

VISUAL STYLE:
All icons are clean isometric 3D with subtle shadows and modern gradient fills. The Kubernetes wheel logo appears subtly watermarked in the cluster background.

CLUSTER BOUNDARY:
A large rounded rectangle with a thin purple dashed border labeled 'EKS Cluster: k8s-prod' in the top-left corner.

INGRESS (top of cluster):
A purple isometric ingress controller icon labeled 'nginx-ingress' sits at the top center. A blue curved arrow enters from above, originating from a cloud/globe icon labeled 'Internet'.

NAMESPACE ZONES (arranged horizontally inside the cluster):

ZONE 1 - Frontend Namespace (light blue tinted rectangle with rounded corners):
- Header label: 'namespace: frontend'
- A purple Service icon labeled 'frontend-svc' at the top
- Three orange Pod icons in a row below, each labeled 'web-app'
- Pods connected to the service with thin lines

ZONE 2 - Backend Namespace (light green tinted rectangle):
- Header label: 'namespace: backend'
- A purple Service icon labeled 'api-svc'
- Three orange Pod icons labeled 'api-server'
- Two teal Pod icons labeled 'worker' below
- Internal green dotted arrows show async messaging

ZONE 3 - Data Namespace (light amber tinted rectangle):
- Header label: 'namespace: data'
- A blue isometric StatefulSet icon labeled 'postgres'
- A green PVC icon labeled 'db-storage' connected below

CONNECTIONS:
- Bright blue curved arrow with 'HTTPS 443' label from ingress to frontend service
- Purple curved arrows with 'HTTP 8080' labels from frontend pods to api-svc
- Amber dashed arrows with 'PostgreSQL 5432' labels from api pods to postgres

EXTERNAL SERVICES (outside cluster, right side):
- A green isometric S3 bucket icon labeled 'user-uploads'
- A blue RDS icon labeled 'analytics-db'
- Green dotted arrows connect worker pods to S3, amber dashed arrows connect api pods to RDS

FOOTER (inside the frame, at the bottom):
A compact footer bar inside the border frame shows a legend with icon samples and labels. Kubernetes logo on left, AWS logo on right - all inside the frame with no content touching the border edge.

The diagram is visually polished with consistent spacing, harmonious colors, generous margins, and professional aesthetics suitable for architecture review presentations.
```

### Step 5.5: Quick Prompt Template

Use this template as a starting point, filling in the bracketed sections:

```
A professional 16:9 landscape cloud architecture diagram in a stunning modern infographic style.

CANVAS AND MARGINS:
The image has a generous white outer margin (at least 60 pixels on all sides) so no content approaches the page edges. Inside this margin, a subtle rounded-corner border with a thin charcoal stroke frames all diagram content. Everything is contained inside this border with comfortable inner padding.

HEADER (inside the frame):
A gradient header bar spans the top inside the frame, transitioning from [PRIMARY_COLOR] on the left to [SECONDARY_COLOR] on the right. The title '[ARCHITECTURE_NAME]' appears in large, bold white sans-serif text. A subtitle reads '[DESCRIPTION] • [REGION/ENVIRONMENT]' in smaller light text.

MAIN CANVAS:
Light cool gray background (#f8fafc). Layout flows left-to-right showing the data/request path. Generous spacing between all elements.

VISUAL STYLE:
All icons are clean isometric 3D with subtle drop shadows and soft gradient fills - like high-end tech infographic illustrations. Consistent color palette: orange for compute, purple for networking, blue for databases, green for storage, teal for analytics.

ARCHITECTURE ZONES:
[Describe each zone with tinted background color, dashed border, label, and contained resources]

ZONE 1 - [ZONE_NAME] ([ZONE_COLOR] tinted rectangle with dashed border):
- [Resource descriptions with isometric style, color, and labels]

ZONE 2 - [ZONE_NAME] ([ZONE_COLOR] tinted rectangle):
- [Resource descriptions]

[Continue for additional zones...]

CONNECTIONS:
[Describe each connection with curved bezier arrows, color, and pill-shaped labels]
- [COLOR] curved arrow with '[PROTOCOL PORT]' label from [SOURCE] to [DESTINATION]

FOOTER (inside the frame, at the bottom):
A compact footer bar inside the border frame shows a legend with icon samples and labels. [PROVIDER] logo in bottom-right corner - all inside the frame with no content touching or extending beyond the border.

The diagram is visually polished with generous margins, consistent spacing, harmonious colors, and professional aesthetics.
```

**Color suggestions for headers:**
- AWS: Navy blue (#1a365d) → Teal (#0d9488)
- Azure: Dark blue (#1e3a5f) → Cyan (#06b6d4)
- GCP: Deep blue (#1e40af) → Red (#dc2626)
- Kubernetes: Indigo (#3730a3) → Violet (#7c3aed)
- Multi-cloud: Slate (#334155) → Purple (#9333ea)

### Step 6: Generate the Diagram

After creating the enhanced prompt, generate the diagram:

```bash
python ~/.claude/skills/nanobanana/scripts/generate.py "ENHANCED_PROMPT_HERE"
```

The script will save the diagram as a timestamped PNG file in the current directory.

### Step 7: Provide Context

After generating the diagram, provide the user with:
1. The diagram filename and location
2. A summary of the architecture components
3. Any notable patterns or best practices observed
4. Suggestions for improvements if applicable

## Supported IaC Formats

### Terraform
- **Extensions**: `.tf`, `.tfvars`
- **Features**: Resource extraction, module resolution, variable handling
- **Dependencies**: Explicit (`depends_on`) and implicit (resource references)

**Terraform Parsing Tiers** (automatic selection):

| Parser | Accuracy | Requirements | What It Does |
|--------|----------|--------------|--------------|
| **tfparse** | Best | `terraform init` run, Python 3.10+ | Full expression evaluation, accurate dependencies |
| **python-hcl2** | Good | None | Proper HCL2 syntax parsing, reference extraction |
| **regex** | Basic | None | Simple pattern matching, inferred dependencies |

The parser automatically selects the best available option:
1. If `.terraform/` exists and tfparse is installed → uses tfparse
2. If python-hcl2 is installed → uses hcl2
3. Falls back to regex extraction

**For best results with Terraform:**
```bash
cd /path/to/terraform
terraform init  # Downloads providers, resolves modules
# Then run the diagram generator
```

### AWS CloudFormation
- **Extensions**: `.yaml`, `.yml`, `.json`, `.template`
- **Features**: Resource extraction, parameter resolution, intrinsic function parsing
- **Dependencies**: `DependsOn`, `Ref`, `GetAtt` references

**CloudFormation Parsing Tiers** (automatic selection):

| Parser | Accuracy | Requirements | What It Does |
|--------|----------|--------------|--------------|
| **cfn-lint** | Best | `pip install cfn-lint` | Full intrinsic function parsing |
| **PyYAML** | Good | Built-in | Basic YAML with reference extraction |

The parser extracts dependencies from:
- `!Ref` / `Fn::Ref` - Direct resource references
- `!GetAtt` / `Fn::GetAtt` - Attribute lookups
- `!Sub` / `Fn::Sub` - String substitution references
- `DependsOn` - Explicit dependencies
- `Fn::If` - Conditional branch scanning

### Kubernetes
- **Extensions**: `.yaml`, `.yml` (manifests, Helm templates)
- **Features**: Resource extraction, label selectors, namespace organization
- **Dependencies**: Service selectors, ConfigMap/Secret references, ownerReferences

**Kubernetes Relationship Detection** (inspired by [KubeDiagrams](https://github.com/philippemerle/KubeDiagrams)):

| Relationship | Example | Detection Method |
|--------------|---------|------------------|
| **SELECTOR** | Service → Deployment | Label selector matching |
| **OWNER** | Deployment → Pod | Implicit ownership hierarchy |
| **REFERENCE** | Ingress → Service | Backend service references |
| **MOUNT** | Deployment → ConfigMap | Volume mount definitions |
| **COMMUNICATION** | NetworkPolicy rules | Ingress/egress pod selectors |

**Supported Kubernetes Resources** (20+ types):
- Workloads: Deployment, StatefulSet, DaemonSet, Job, CronJob, Pod
- Networking: Service, Ingress, NetworkPolicy
- Config: ConfigMap, Secret
- Storage: PersistentVolume, PersistentVolumeClaim
- RBAC: Role, ClusterRole, RoleBinding, ClusterRoleBinding, ServiceAccount

### Docker Compose
- **Extensions**: `docker-compose.yaml`, `docker-compose.yml`
- **Features**: Service extraction, network topology, volume mappings
- **Dependencies**: `depends_on`, network membership, volume sharing

### Pulumi
- **Extensions**: `.ts`, `.py`, `.go`
- **Features**: Basic resource extraction from code analysis
- **Note**: Requires language-specific AST parsing

## Best Practices

### DO:
- **Always use the standardized visual template** - 16:9 landscape, gradient header, framed border, footer legend
- **Specify isometric 3D icon style** - NOT flat official cloud icons; use "clean isometric 3D with subtle shadows and gradient fills"
- **Use the defined color palette** consistently - orange for compute, purple for networking, blue for databases, green for storage
- **Define zones with tinted backgrounds** - light blue for public, light green for private, light amber for data layers
- **Describe curved bezier arrows** for connections, not straight lines
- **Include pill-shaped labels** on connection arrows with protocol and port
- **Start prompts with canvas/frame description** before architecture content
- **Use narrative descriptions** - full sentences describing spatial relationships, not keyword lists
- **Group resources visually** by security boundaries with dotted/dashed borders
- **Specify left-to-right data flow** as the default orientation
- **Include CIDR blocks** in zone labels for networks
- **End with aesthetic summary** - reinforce "visually stunning, professional" quality

### DON'T:
- Request "official AWS/Azure/GCP icons" - they produce inconsistent, flat results
- Use white backgrounds - specify "very light cool gray (#f8fafc)" instead
- Forget the header gradient and title placement
- Use straight line arrows - always specify curved/bezier paths
- Mix icon styles within the same diagram
- Create cluttered diagrams - if >15 resources, split into multiple focused diagrams
- Use generic descriptions like "some servers" - be specific with names and counts
- Omit connection labels - every arrow needs a protocol/port label
- Forget the footer legend - it adds polish and professionalism
- Skip the zone background colors - they're essential for visual hierarchy

## Common Architecture Patterns

### Three-Tier Web Application
- Public subnet: Load balancers, NAT gateways
- Private subnet: Application servers, workers
- Database subnet: RDS, ElastiCache (no direct internet access)

### Microservices
- API Gateway / Ingress at the edge
- Service mesh for inter-service communication
- Separate namespaces or VPCs per service domain
- Shared data stores and message queues

### Serverless
- API Gateway triggering Lambda functions
- Lambda functions accessing DynamoDB, S3
- EventBridge or SQS for async processing
- CloudFront for content delivery

## Setup Requirements

### Required Python Packages

```bash
pip install pyyaml
```

### Optional (Recommended) Packages

For better Terraform parsing:
```bash
# Good: HCL2 parsing without terraform init
pip install python-hcl2

# Best: Full evaluation with terraform init (Python 3.10+ required)
pip install tfparse
```

For better CloudFormation parsing:
```bash
# Accurate intrinsic function parsing (!Ref, !GetAtt, !Sub)
pip install cfn-lint
```

### Environment Variables

```bash
# Gemini API key for Nano Banana Pro
export GEMINI_API_KEY="your-api-key-here"
```

Get your API key at: https://aistudio.google.com/apikey

### Optional Tools

For enhanced Terraform parsing:
```bash
pip install tfparse
```

For CloudFormation validation:
```bash
pip install cfn-lint
```

## Vector Output (Post-Processing)

Nano Banana Pro generates raster images (PNG), not vector files. For editable SVG/PDF output, use a vectorization tool as a post-processing step:

### Recommended Vectorization Tools

**Best Quality:**
- [Vectorizer.AI](https://vectorizer.ai/) - Deep learning-based, outputs SVG/PDF/EPS/DXF
- [Vector Magic](https://vectormagic.com/) - Full-color tracing, also has desktop app for Illustrator

**Free Options:**
- [Recraft AI Vectorizer](https://www.recraft.ai/ai-image-vectorizer) - Free, one-click PNG→SVG, also exports Lottie
- [AI Vector](https://aivector.ai/) - 100% free, no registration, claims 99.9% accuracy
- [SVGConverter.app](https://svgconverter.app/) - Free, supports multiple output formats

### Workflow for Editable Diagrams

1. Generate PNG diagram using this skill
2. Upload to vectorization tool of choice
3. Download SVG/PDF/EPS output
4. Edit in Figma, Illustrator, Inkscape, or similar

### Tips for Better Vectorization

- The isometric 3D style with solid colors vectorizes better than photorealistic images
- Clean lines and distinct color zones improve tracing accuracy
- Simpler diagrams (fewer overlapping elements) produce cleaner vectors
- Some manual cleanup in a vector editor may still be needed for complex diagrams

## Error Handling

The parser handles:
- Missing or invalid IaC files
- Unsupported IaC formats
- Syntax errors in IaC (reports to user)
- Missing environment variables
- File read permissions

All errors include clear messages for troubleshooting.

## Examples

### Example 1: Terraform Directory

**User Request**: "Generate an architecture diagram from my Terraform code"

**Steps**:
1. Use Glob to find `.tf` files in current directory
2. Run: `python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py terraform .`
3. Analyze the JSON output to understand the architecture
4. Create enhanced Nano Banana Pro prompt describing the AWS resources, VPC structure, and connections
5. Generate diagram: `python ~/.claude/skills/nanobanana/scripts/generate.py "PROMPT"`
6. Report diagram location and summary

### Example 2: CloudFormation Template

**User Request**: "Show me what this CloudFormation template deploys"

**Steps**:
1. Identify the template file
2. Run: `python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py cloudformation template.yaml`
3. Extract resources and dependencies from JSON
4. Create detailed prompt showing resource hierarchy and connections
5. Generate diagram
6. Explain the architecture and resource relationships

### Example 3: Kubernetes Manifests

**User Request**: "Diagram our Kubernetes application"

**Steps**:
1. Find YAML manifests in k8s/ or manifests/ directory
2. Run: `python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py kubernetes k8s/`
3. Identify Deployments, Services, Ingresses, and their relationships
4. Create prompt showing namespace organization and service communication
5. Generate diagram
6. Provide insights on the microservices architecture

### Example 4: GitHub Repository

**User Request**: "Generate a diagram from https://github.com/example/terraform-aws-vpc"

**Steps**:
1. Detect the GitHub URL in the user's request
2. Run: `python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py terraform https://github.com/example/terraform-aws-vpc`
3. The parser automatically clones the repo, parses, and cleans up
4. Analyze the JSON output to understand the architecture
5. Create enhanced Nano Banana Pro prompt
6. Generate diagram and report location

**User Request**: "Show me the architecture in the /infrastructure folder of https://github.com/example/myapp"

**Steps**:
1. Detect GitHub URL with subdirectory
2. Run: `python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py terraform https://github.com/example/myapp/tree/main/infrastructure`
3. Only the specified subdirectory is parsed
4. Continue with diagram generation
