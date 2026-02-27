---
name: health-chat
description: Unified health conversation entry point - automatically loads all health data for each conversation, supports natural language queries, and intelligently routes to appropriate health data processing
user-invocable: true
disable-model-invocation: false
context: fork
agent: general-purpose
argument-hint: <natural language health question or instruction>
allowed-tools: Read, Write
schema: health-chat/schema.md
---

# Health Chat Skill

The unified conversation entry point for WellallyHealth system. Automatically loads and considers all health data for each conversation, providing intelligent health consultation and data analysis services.

## Core Design Philosophy

This is the **unified conversation entry point** for WellallyHealth. Every conversation automatically loads and analyzes all health data, providing intelligent health consultation and data analysis services.

## Core Workflow

```
User Input -> 1. Load All Health Data (data/*.json)
         -> 2. Parse User Intent (query/analysis/advice/alert)
         -> 3. Intelligent Routing to Data Processing Module
         -> 4. Generate Personalized Response
         -> 5. Save Conversation History (ai-history.json)
```

## Step 1: Load Data (Execute Every Conversation)

### Core Data Sources (Priority Sorted)

**IMPORTANT**: Data loading uses `data/**/*.json` pattern to include all subdirectories.

| Data File | Purpose | Key Fields |
|-------------|-----------|--------------|
| `data/profile.json` | User basic info | gender, height, weight, birth_date, BMI, BSA |
| `data/user-settings.json` | User preferences | language, units, notifications |
| `data/ai-config.json` | AI features config | features, safety, data_sources |
| `data/ai-history.json` | Conversation history | recent_conversations |

### Chronic Condition Tracking Data

| Data File | Health Domain |
|-------------|----------------|
| `data/hypertension-tracker.json` | Hypertension management |
| `data/diabetes-tracker.json` | Diabetes management |
| `data/copd-tracker.json` | COPD management |
| `data/postpartum-tracker.json` | Postpartum management |
| `data/menopause-tracker.json` | Menopause management |
| `data/prostate-tracker.json` | Prostate health |
| `data/andropause-tracker.json` | Male menopause |
| `data/cycle-tracker.json` | Menstrual cycle |
| `data/pregnancy-tracker.json` | Pregnancy tracking |

### Specialist Health Data

| Data File | Health Domain |
|-------------|----------------|
| `data/cognitive-assessment.json` | Cognitive assessment |
| `data/eye-health-tracker.json` | Eye health |
| `data/fall-risk-assessment.json` | Fall risk |
| `data/growth-tracker.json` | Growth records |
| `data/fertility-tracker.json` | Fertility health |

### Medical Data

| Data File | Purpose |
|-------------|-----------|
| `data/medications/medications.json` | Medication plans |
| `data/allergies.json` | Allergy records |
| `data/vaccinations.json` | Vaccination records |
| `data/child-vaccinations.json` | Child vaccination |
| `data/radiation-records.json` | Radiation exposure |
| `data/polypharmacy-management.json` | Polypharmacy |
| `data/interactions/interaction-db.json` | Drug interactions |

### Health Management Data

| Data File | Purpose |
|-------------|-----------|
| `data/health-feeling-logs.json` | Health feeling logs |
| `data/family-health-tracker.json` | Family health |
| `data/reminders.json` | Reminders |
| `data/travel-health-tracker.json` | Travel health |

### Database Files

| Data File | Purpose |
|-------------|-----------|
| `data/index.json` | Medical records index |
| `data/food-database.json` | Food nutrition database |
| `data/vaccine-database.json` | Vaccine database |
| `data/child-vaccine-database.json` | Child vaccine database |
| `data/nutritional-reference.json` | Nutrition reference standards |
| `data/who-growth-standards.json` | WHO growth standards |

### TCM Data

| Data File | Purpose |
|-------------|-----------|
| `data/constitutions.json` | TCM constitution |
| `data/constitution-recommendations.json` | Constitution recommendations |

### Imaging Records

- `data/影像检查/YYYY-MM/YYYY-MM-DD_检查名称.json`

## Step 2: Parse User Intent

### Intent Classification

| Intent Type | Trigger Keywords | Processing |
|--------------|-------------------|--------------|
| **Data Query** | what, how much, recent, average, trend | Read corresponding data, calculate statistics |
| **Health Analysis** | analyze, assess, how is, status | Multi-dimensional data analysis |
| **Risk Alert** | risk, abnormal, warning | Apply risk models, calculate risk level |
| **Recommendation** | advice, how to, improve, should | Generate personalized recommendations |
| **Record Operation** | record, add, update | Write to tracker files |
| **Medical Consult** | doctor, test, treatment | Check data, provide medical reference |
| **Medication** | med, drug, dose, interaction | Read medications data |
| **Symptom Inquiry** | symptom, discomfort, pain | Analyze symptoms with health data |

## Step 3: Intelligent Routing

### Data Query Routing

```
User Question -> Match Keywords -> Route to Data Source
"How is my blood pressure?" -> hypertension-tracker.json -> Analyze BP trends
"How's my sleep lately?" -> Sleep-related data -> Provide assessment
"What's my BMI?" -> profile.json -> Return BMI and advice
"What meds do I take today?" -> medications/medications.json -> Return today's meds
```

### Health Analysis Routing

```
"Full analysis" -> Read all tracker data -> Generate comprehensive report
"Chronic condition analysis" -> Read chronic trackers -> Specialized analysis
"Mental health" -> Read mental health data -> Assessment and recommendations
```

### Risk Assessment Routing

```
"Hypertension risk" -> hypertension-tracker.json + profile.json -> Apply Framingham model
"Diabetes risk" -> diabetes-tracker.json + profile.json -> Apply ADA model
"Fall risk" -> fall-risk-assessment.json -> Assessment results
```

## Step 4: Response Generation Guidelines

### Response Structure

```markdown
## 📊 Health Data Summary
[Brief overview based on current data]

## 🎯 Key Findings
[Health metrics or issues needing attention]

## 💡 Personalized Recommendations
[Personalized advice based on data]

## 📈 Trend Analysis
[If applicable, show data trends]

## ⚠️ Risk Alerts
[If applicable, alert on risk factors]

---

⚕️ Medical Disclaimer: This health information is for reference only and cannot replace professional medical advice.
Please consult a healthcare professional for health concerns.
```

### Response Style Requirements

1. **Data-Driven**: All conclusions must be based on actual data
2. **Personalized**: Adjust recommendations based on user characteristics
3. **Clear & Concise**: Avoid excessive medical jargon
4. **Positive Orientation**: Focus on encouragement and help
5. **Safety First**: Clearly recommend medical care for high-risk situations

## Step 5: Conversation History Management

### Save Conversations to ai-history.json

```json
{
  "conversations": [
    {
      "timestamp": "YYYY-MM-DDTHH:mm:ss",
      "user_input": "Original user input",
      "intent": "Identified intent type",
      "data_sources_used": ["List of data files used"],
      "response_summary": "Response summary",
      "follow_up_suggestions": ["Possible follow-up questions"]
    }
  ],
  "statistics": {
    "total_conversations": 100,
    "common_topics": ["blood pressure", "sleep", "medication"],
    "last_updated": "YYYY-MM-DD"
  }
}
```

## Intelligent Routing Examples

### Example 1: Blood Pressure Query
```
User: "How has my blood pressure been lately?"
Routing:
1. Read hypertension-tracker.json
2. Extract recent BP records
3. Calculate average and trends
4. Reference profile.json for basic info
5. Generate personalized response
```

### Example 2: Medication Consultation
```
User: "What medications should I take today?"
Routing:
1. Read medications/medications.json
2. Read interactions/interaction-db.json
3. Filter today's medication plan
4. Check for interactions
5. Generate medication reminder
```

### Example 3: Health Assessment
```
User: "Give me a health assessment"
Routing:
1. Read profile.json for basic info
2. Read all chronic condition trackers
3. Read latest test records (index.json)
4. Comprehensive health status analysis
5. Apply risk models (ai-config.json)
6. Generate comprehensive report
```

### Example 4: Symptom Consultation
```
User: "I've been feeling dizzy lately"
Routing:
1. Read hypertension-tracker.json (check BP)
2. Read diabetes-tracker.json (check blood sugar)
3. Read medications/medications.json (check side effects)
4. Analyze possible correlations
5. Provide reference recommendations
6. Recommend medical care if high risk
```

## Data Reading Priority

### Must-Read Data (Every Conversation)
1. `data/profile.json` - User basic information
2. `data/user-settings.json` - User preferences
3. `data/ai-config.json` - AI configuration

### On-Demand Data (By Question Type)
- Chronic conditions: Read corresponding tracker
- Medication: Read medications
- Tests: Read index.json and corresponding test records
- Symptoms: Read related health data and medication records

### Database Files (Reference Data)
- Read when querying nutrition/vaccine info
- Read when comparing to standard values

## Safety Boundaries

1. **Medical Disclaimer**: Required for every response
2. **No Diagnosis**: Clearly state non-doctor diagnosis
3. **No Prescription**: No dosage adjustment recommendations
4. **High-Risk Alert**: Recommend medical care when risk > 0.7
5. **Privacy Protection**: Data is local-only by default

## Execution Instructions

```
1. Read profile.json and ai-config.json (mandatory)
2. Analyze user input for intent
3. Read corresponding data files based on intent type
4. Process data and generate response
5. Add medical disclaimer
6. (Optional) Save conversation to ai-history.json
```

## Common Conversation Patterns

### Pattern 1: Daily Health Inquiry
```
User: "I've been feeling tired lately, what's the reason?"
Process:
1. Check sleep data
2. Check recent health status
3. Check medication status
4. Analyze possible causes
5. Provide recommendations
```

### Pattern 2: Data Query
```
User: "How has my weight changed recently?"
Process:
1. Read weight-related data
2. Calculate trends
3. Visualize display
```

### Pattern 3: Medication Reminder
```
User: "Did I take my meds today?"
Process:
1. Read medication plan
2. Check today's taken records
3. Remind of missed medications
```

### Pattern 4: Alert Notification
```
User: "What should I watch out for in my health?"
Process:
1. Check all abnormal indicators
2. Assess risk factors
3. Summarize alerts
4. Provide action recommendations
```

---

## Quick Start

Every conversation starts with automatic execution:

```bash
# Step 1: Load core data
Read data/profile.json
Read data/user-settings.json
Read data/ai-config.json

# Step 2: Analyze user input
# Parse intent, identify keywords

# Step 3: Read relevant data
# Based on intent type, read corresponding trackers

# Step 4: Generate response
# Data-driven + Personalized + Medical Disclaimer
```

---

**Note**: This skill is the unified conversation entry point for WellallyHealth. All health-related conversations are recommended to go through this skill.
