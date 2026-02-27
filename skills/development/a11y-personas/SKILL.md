---
name: a11y-personas
description: Library of accessibility personas representing people with various disabilities, impairments, and situational limitations. Use this skill when users ask about disability types, accessibility personas, user needs for specific conditions, how people with disabilities use technology, assistive technology users, or designing for accessibility. Triggers on requests about blindness, deafness, cognitive disabilities, motor impairments, low vision, screen readers, sign language, autism, ADHD, temporary disabilities, or any question about "how would a person with X use this".
---

# A11y Personas

Library of accessibility personas for understanding diverse user needs.

## Auto-Initialize

**Before reading any persona files**, check if the `repo/` folder exists in this skill directory. If it doesn't exist, run the setup script:

```bash
cd .cursor/skills/a11y-personas && ./setup.sh
```

This clones the a11y-personas repository. Only needs to run once.

## Updating Content

To pull the latest personas:

```bash
cd .cursor/skills/a11y-personas/repo && git pull
```

## Data Source

All personas are in `repo/data/personas/` as markdown files. Each contains:
- **id**: Unique identifier
- **title**: Descriptive name
- **profile**: Key characteristics
- **interaction_style**: Input/output methods and limitations
- **key_needs**: Accessibility requirements
- **cross_functional_considerations**: Guidance for customer care, development, design, and testing
- **biography**: Narrative description with quote

## Finding Personas

Search `repo/data/personas/` for relevant personas by category:

### Vision
| File | Persona |
|------|---------|
| `blindness-screen-reader-nvda.md` | NVDA screen reader user (Windows) |
| `blindness-screen-reader-voiceover.md` | VoiceOver user (Apple) |
| `blindness-braille-user.md` | Braille display user |
| `blindness-light-perception.md` | Light perception only |
| `blindness-low-vision-progressive.md` | Progressive vision loss |
| `low-vision.md` | General low vision |
| `color-vision-deficiency.md` | Color blindness |
| `vision-contrast-sensitivity.md` | Contrast sensitivity |
| `visual-processing-disorder.md` | Visual processing differences |

### Hearing
| File | Persona |
|------|---------|
| `deafness-sign-language-user.md` | Sign language primary |
| `deafness-hard-of-hearing.md` | Hard of hearing |
| `deafness-late-deafened.md` | Late-deafened adult |
| `deafness-oral-communicator.md` | Oral/lipreading |
| `deaf-blind.md` | Deaf-blind |
| `hearing-loss-age-related.md` | Age-related hearing loss |
| `tinnitus-audio-sensitivity.md` | Tinnitus |

### Motor/Physical
| File | Persona |
|------|---------|
| `paraplegia-wheelchair.md` | Wheelchair user |
| `parkinson-tremor.md` | Parkinson's/tremor |
| `mobility-one-handed-limb-difference.md` | One-handed use |
| `mobility-chronic-pain.md` | Chronic pain |
| `mobility-repetitive-stress-injury.md` | RSI |
| `multiple-sclerosis-fluctuating.md` | MS fluctuating symptoms |
| `arthritis-rheumatoid.md` | Rheumatoid arthritis |

### Cognitive/Neurological
| File | Persona |
|------|---------|
| `cognitive-memory-loss.md` | Memory loss |
| `cognitive-aphasia-language.md` | Aphasia |
| `dyslexia-reading.md` | Dyslexia |
| `dyscalculia-number-processing.md` | Dyscalculia |
| `intellectual-disability-mild.md` | Intellectual disability |
| `adhd-attention.md` | ADHD |
| `epilepsy-seizure-risk.md` | Epilepsy/seizures |

### Autism Spectrum
| File | Persona |
|------|---------|
| `autistic.md` | General autistic needs |
| `autistic-sensory-sensitive.md` | Sensory sensitivities |
| `autistic-communication-differences.md` | Communication differences |
| `autistic-executive-function.md` | Executive function |
| `autistic-visual-thinker.md` | Visual thinking |
| `autistic-rule-oriented.md` | Rule-oriented |
| `autistic-non-speaking.md` | Non-speaking |

### Mental Health
| File | Persona |
|------|---------|
| `anxiety-mental-health.md` | Anxiety |
| `depression-major.md` | Depression |
| `ptsd-trauma.md` | PTSD/trauma |

### Speech
| File | Persona |
|------|---------|
| `speech-impairment-communication.md` | Speech impairment |
| `motor-impaired-non-speaking.md` | Non-speaking motor impaired |

### Temporary/Situational
| File | Persona |
|------|---------|
| `temp-broken-dominant-arm.md` | Broken arm |
| `temp-concussion-cognitive-fatigue.md` | Concussion |
| `temp-eye-patch-temporary-vision.md` | Eye patch |
| `temp-holding-child-one-handed.md` | Holding child |
| `temp-laryngitis-temporary-voice-loss.md` | Voice loss |
| `temp-migraine-light-sensitivity.md` | Migraine |
| `temp-noisy-environment-limited-audio.md` | Noisy environment |
| `temp-public-place-privacy-concern.md` | Public privacy concerns |
| `temp-crisis-situation.md` | Crisis/stress |

## Usage

### Get a specific persona
Read the relevant file from `repo/data/personas/` to understand:
- How the person interacts with technology
- What barriers they face
- What accessibility features they need
- Considerations for different teams

### Compare personas
Read multiple personas to understand overlapping and distinct needs.

### Design review
Use personas to evaluate if a design meets the needs of specific user groups.

### Generate test scenarios
Use the `cross_functional_considerations.testing` section for test case ideas.

## Persona Structure

Each persona follows this format:

```yaml
---
id: unique-identifier
title: Display Name
profile:
  - Key characteristic 1
  - Key characteristic 2
interaction_style:
  input: [methods used]
  output: [feedback needed]
  no_reliance_on: [inaccessible patterns]
key_needs:
  - Requirement 1
  - Requirement 2
cross_functional_considerations:
  customer_care: [support guidance]
  development: [implementation guidance]
  design_ux: [design guidance]
  testing: [testing guidance]
---

## Biography
Narrative description with representative quote.
```

## Quick Lookup by Assistive Technology

| Technology | Relevant Personas |
|------------|-------------------|
| Screen reader (NVDA) | `blindness-screen-reader-nvda.md` |
| Screen reader (VoiceOver) | `blindness-screen-reader-voiceover.md` |
| Braille display | `blindness-braille-user.md` |
| Screen magnifier | `low-vision.md`, `blindness-low-vision-progressive.md` |
| Keyboard only | Multiple motor personas |
| Switch device | `motor-impaired-non-speaking.md` |
| Voice control | `mobility-repetitive-stress-injury.md` |
| Captions | All hearing personas |
| Sign language | `deafness-sign-language-user.md` |
