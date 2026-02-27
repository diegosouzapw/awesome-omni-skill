---
name: ab-testing-statistician
description: Expert in statistical analysis for blind A/B and ABX audio testing. Validates randomization, calculates statistical significance, and ensures proper experimental design. Use when implementing A/B test features or analyzing test results.
---

# A/B Testing Statistician

Specialized agent for designing and validating blind audio comparison tests (A/B, Blind AB, ABX) with proper statistical analysis.

## Overview of Audio A/B Testing

### Test Modes

| Mode | Description | User Knows? | Purpose |
|------|-------------|-------------|---------|
| **AB** | Switch between A and B | Yes | Quick comparison, training |
| **Blind AB** | A and B randomly mapped to Options 1 and 2 | No | Unbiased preference detection |
| **ABX** | X is secretly either A or B, user guesses | No | Audibility testing (can you hear the difference?) |

### Why Blind Testing Matters

**Confirmation Bias**: Listeners tend to prefer what they expect to be better.

**Example:**
```
Non-blind: "This expensive cable sounds clearer!" (placebo effect)
Blind: "I can't tell the difference" (objective reality)
```

## Session Management

### Session State (Rust)

```rust
#[derive(Clone, Serialize, Deserialize)]
pub struct ABSession {
    pub mode: ABTestMode,           // AB, BlindAB, or ABX
    pub preset_a_name: String,
    pub preset_b_name: String,
    pub trim_db: f32,               // Loudness compensation for B
    pub total_trials: usize,
    pub current_trial: usize,
    pub hidden_mapping: Vec<bool>,  // For BlindAB: true = Option1 is A
    pub x_is_a: Vec<bool>,          // For ABX: true = X is A
    pub answers: Vec<ABAnswer>,     // User responses
}

#[derive(Clone, Serialize, Deserialize)]
pub enum ABTestMode {
    AB,       // Non-blind switching
    BlindAB,  // Blind preference test
    ABX,      // Blind audibility test
}

#[derive(Clone, Serialize, Deserialize)]
pub struct ABAnswer {
    pub trial: usize,
    pub selected_option: String,    // "A", "B", "1", "2", or "X"
    pub timestamp: u64,             // Milliseconds since session start
}
```

### Randomization (Critical!)

**BlindAB Mode:**
Each trial randomly maps A/B to Options 1/2:

```rust
pub fn create_blind_ab_session(
    preset_a: String,
    preset_b: String,
    num_trials: usize,
    trim_db: f32,
) -> ABSession {
    use rand::Rng;
    let mut rng = rand::thread_rng();

    // Randomize each trial independently
    let hidden_mapping: Vec<bool> = (0..num_trials)
        .map(|_| rng.gen_bool(0.5)) // 50% chance Option1 = A
        .collect();

    ABSession {
        mode: ABTestMode::BlindAB,
        preset_a_name: preset_a,
        preset_b_name: preset_b,
        trim_db,
        total_trials: num_trials,
        current_trial: 0,
        hidden_mapping,
        x_is_a: vec![],
        answers: vec![],
    }
}
```

**ABX Mode:**
X is randomly set to A or B for each trial:

```rust
pub fn create_abx_session(
    preset_a: String,
    preset_b: String,
    num_trials: usize,
    trim_db: f32,
) -> ABSession {
    use rand::Rng;
    let mut rng = rand::thread_rng();

    // Randomize X for each trial
    let x_is_a: Vec<bool> = (0..num_trials)
        .map(|_| rng.gen_bool(0.5)) // 50% chance X = A
        .collect();

    ABSession {
        mode: ABTestMode::ABX,
        preset_a_name: preset_a,
        preset_b_name: preset_b,
        trim_db,
        total_trials: num_trials,
        current_trial: 0,
        hidden_mapping: vec![],
        x_is_a,
        answers: vec![],
    }
}
```

**Critical Rule:** Randomize PER TRIAL, not once for all trials!

❌ Wrong:
```rust
let option1_is_a = rng.gen_bool(0.5);
// Use same mapping for all trials
```

✅ Correct:
```rust
let hidden_mapping: Vec<bool> = (0..num_trials)
    .map(|_| rng.gen_bool(0.5))
    .collect();
```

## Loudness Compensation (Trim Parameter)

**Problem:** Louder = perceived as "better" (Fletcher-Munson curves)

**Solution:** Level-match presets before testing

### Auto-Calculate Trim

```rust
pub fn calculate_auto_trim(
    bands_a: &[ParametricBand],
    preamp_a: f32,
    bands_b: &[ParametricBand],
    preamp_b: f32,
) -> f32 {
    use crate::audio_math::calculate_peak_gain;

    let peak_a = calculate_peak_gain(bands_a, preamp_a);
    let peak_b = calculate_peak_gain(bands_b, preamp_b);

    // Adjust B to match A's peak level
    peak_a - peak_b
}
```

### Apply Trim to Preset B

```rust
pub fn apply_preset_with_trim(
    bands: &[ParametricBand],
    preamp: f32,
    trim_db: f32,
) -> Result<(), String> {
    let adjusted_preamp = preamp + trim_db;

    // Apply to EqualizerAPO
    write_eapo_config(bands, adjusted_preamp)?;

    Ok(())
}
```

**Example:**
```
Preset A: Peak gain = -2 dB
Preset B: Peak gain = +1 dB
Trim = -2 - (+1) = -3 dB

Apply Preset B with -3 dB trim → Both have -2 dB peak
```

## Statistical Analysis

### Preference Analysis (BlindAB)

Count how many times each preset was preferred:

```rust
pub struct PreferenceResults {
    pub a_selected: usize,
    pub b_selected: usize,
    pub total_trials: usize,
    pub a_percentage: f64,
    pub b_percentage: f64,
    pub p_value: f64,  // Statistical significance
}

pub fn analyze_blind_ab(session: &ABSession) -> PreferenceResults {
    let mut a_count = 0;
    let mut b_count = 0;

    for (i, answer) in session.answers.iter().enumerate() {
        let option1_is_a = session.hidden_mapping[i];

        let selected_a = match answer.selected_option.as_str() {
            "1" => option1_is_a,
            "2" => !option1_is_a,
            _ => continue,
        };

        if selected_a {
            a_count += 1;
        } else {
            b_count += 1;
        }
    }

    let total = a_count + b_count;
    let a_pct = (a_count as f64 / total as f64) * 100.0;
    let b_pct = (b_count as f64 / total as f64) * 100.0;

    // Binomial test: is this significantly different from 50/50?
    let p_value = binomial_test(a_count, total, 0.5);

    PreferenceResults {
        a_selected: a_count,
        b_selected: b_count,
        total_trials: total,
        a_percentage: a_pct,
        b_percentage: b_pct,
        p_value,
    }
}
```

### ABX Analysis (Audibility Test)

Count correct vs incorrect identifications:

```rust
pub struct ABXResults {
    pub correct: usize,
    pub incorrect: usize,
    pub total_trials: usize,
    pub accuracy: f64,
    pub p_value: f64,
}

pub fn analyze_abx(session: &ABSession) -> ABXResults {
    let mut correct = 0;
    let mut incorrect = 0;

    for (i, answer) in session.answers.iter().enumerate() {
        let x_is_a = session.x_is_a[i];

        let guessed_a = match answer.selected_option.as_str() {
            "A" => true,
            "B" => false,
            _ => continue,
        };

        if guessed_a == x_is_a {
            correct += 1;
        } else {
            incorrect += 1;
        }
    }

    let total = correct + incorrect;
    let accuracy = (correct as f64 / total as f64) * 100.0;

    // Binomial test: is this better than 50% guessing?
    let p_value = binomial_test(correct, total, 0.5);

    ABXResults {
        correct,
        incorrect,
        total_trials: total,
        accuracy,
        p_value,
    }
}
```

### Binomial Test (P-Value)

**Null Hypothesis**: User is guessing randomly (50% chance)

**P-Value**: Probability of seeing this result (or more extreme) by chance

```rust
fn binomial_test(successes: usize, trials: usize, p_null: f64) -> f64 {
    use statrs::distribution::{Binomial, Discrete};

    let dist = Binomial::new(p_null, trials as u64).unwrap();

    // Two-tailed test
    let observed = successes as u64;
    let expected = (trials as f64 * p_null) as u64;

    let p_observed = dist.pmf(observed);
    let mut p_value = p_observed;

    // Add probabilities of more extreme outcomes
    for k in 0..=trials as u64 {
        let p_k = dist.pmf(k);
        if p_k <= p_observed && k != observed {
            p_value += p_k;
        }
    }

    p_value.min(1.0)
}
```

**Interpretation:**
- `p < 0.05`: **Significant** - unlikely to be chance (95% confidence)
- `p < 0.01`: **Highly significant** - very unlikely to be chance (99% confidence)
- `p >= 0.05`: **Not significant** - could be random guessing

**Example:**
```
ABX Test: 15/20 correct (75% accuracy)
P-value = 0.041

Interpretation: Statistically significant at 95% level.
User can reliably hear the difference.
```

### Sample Size Requirements

How many trials needed for reliable results?

**Rule of Thumb:**
- **Small effect**: 50+ trials
- **Medium effect**: 20-30 trials
- **Large effect**: 10-15 trials

**Formula (ABX test, 80% power):**
```
n = (Z_α/2 + Z_β)² * p(1-p) / (p - 0.5)²

Where:
- Z_α/2 = 1.96 (for α = 0.05, two-tailed)
- Z_β = 0.84 (for 80% power)
- p = expected accuracy
```

**Example:**
```
Expected accuracy: 70%
n = (1.96 + 0.84)² * 0.7 * 0.3 / (0.7 - 0.5)²
n ≈ 41 trials
```

### Recommended Trial Counts

```rust
pub fn recommended_trial_count(expected_accuracy: f64) -> usize {
    if expected_accuracy <= 0.55 {
        100 // Very subtle difference
    } else if expected_accuracy <= 0.65 {
        50  // Small difference
    } else if expected_accuracy <= 0.75 {
        25  // Medium difference
    } else {
        15  // Large difference
    }
}
```

## Results Export

### CSV Format

```rust
pub fn export_to_csv(session: &ABSession) -> String {
    let mut csv = String::from("Trial,Option1,Option2,Selected,Timestamp\n");

    for (i, answer) in session.answers.iter().enumerate() {
        let (opt1, opt2) = if session.mode == ABTestMode::BlindAB {
            if session.hidden_mapping[i] {
                (&session.preset_a_name, &session.preset_b_name)
            } else {
                (&session.preset_b_name, &session.preset_a_name)
            }
        } else {
            ("A", "B")
        };

        csv.push_str(&format!(
            "{},{},{},{},{}\n",
            i + 1,
            opt1,
            opt2,
            answer.selected_option,
            answer.timestamp
        ));
    }

    csv
}
```

**Output:**
```csv
Trial,Option1,Option2,Selected,Timestamp
1,Flat,Boosted,1,1234
2,Boosted,Flat,2,2456
3,Flat,Boosted,1,3789
```

### JSON Format

```rust
pub fn export_to_json(
    session: &ABSession,
    results: &PreferenceResults,
) -> String {
    let export = serde_json::json!({
        "mode": session.mode,
        "presets": {
            "a": session.preset_a_name,
            "b": session.preset_b_name,
        },
        "trim_db": session.trim_db,
        "trials": session.total_trials,
        "results": {
            "a_selected": results.a_selected,
            "b_selected": results.b_selected,
            "a_percentage": results.a_percentage,
            "b_percentage": results.b_percentage,
            "p_value": results.p_value,
            "significant": results.p_value < 0.05,
        },
        "answers": session.answers,
    });

    serde_json::to_string_pretty(&export).unwrap()
}
```

## Experimental Design Best Practices

### 1. Counterbalancing

Ensure equal distribution of A and B across trials:

```rust
pub fn validate_counterbalancing(hidden_mapping: &[bool]) -> f64 {
    let a_count = hidden_mapping.iter().filter(|&&x| x).count();
    let total = hidden_mapping.len();
    let ratio = a_count as f64 / total as f64;

    // Should be close to 0.5
    (ratio - 0.5).abs()
}
```

**Warning threshold:**
```rust
if validate_counterbalancing(&session.hidden_mapping) > 0.15 {
    println!("Warning: Unbalanced randomization (>15% deviation from 50/50)");
}
```

### 2. Trial Independence

Each trial should be independent:
- ✅ Randomize per trial
- ❌ Use patterns (ABABAB...)
- ❌ Fixed order

### 3. Rest Breaks

Prevent listener fatigue:

```typescript
if (currentTrial % 10 === 0 && currentTrial !== totalTrials) {
  showRestBreakDialog();
}
```

### 4. Reference Switching

Allow listeners to switch between options multiple times before answering:

```typescript
let switchCount = 0;

function handleSwitch() {
  switchCount++;
  applyOpposite();
}

// Log switch count as quality metric
```

## Common Pitfalls

### ❌ Volume Mismatch

```typescript
// WRONG: Apply presets without level matching
applyPresetA();
applyPresetB();

// CORRECT: Apply with trim
applyPreset(presetA, 0);
applyPreset(presetB, trimDb);
```

### ❌ Non-Random Patterns

```rust
// WRONG: Alternating pattern
let hidden_mapping = vec![true, false, true, false, ...];

// CORRECT: True randomization
let hidden_mapping: Vec<bool> = (0..trials)
    .map(|_| rng.gen_bool(0.5))
    .collect();
```

### ❌ Ignoring P-Value

```typescript
// WRONG: Report raw percentages without significance
"Preset A preferred 55% of the time"

// CORRECT: Include statistical context
"Preset A preferred 55% (p=0.42, not significant)"
```

### ❌ Too Few Trials

```typescript
// WRONG: Only 5 trials
const trials = 5; // Unreliable!

// CORRECT: Adequate sample size
const trials = 20; // Minimum for medium effects
```

## Validation Tests

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_randomization_distribution() {
        let session = create_blind_ab_session("A".into(), "B".into(), 1000, 0.0);

        let a_count = session.hidden_mapping.iter().filter(|&&x| x).count();
        let ratio = a_count as f64 / 1000.0;

        // With 1000 trials, should be very close to 0.5
        assert!((ratio - 0.5).abs() < 0.05, "Randomization biased: {}", ratio);
    }

    #[test]
    fn test_trial_independence() {
        let session = create_blind_ab_session("A".into(), "B".into(), 100, 0.0);

        // Count runs (consecutive same values)
        let mut runs = 1;
        for i in 1..session.hidden_mapping.len() {
            if session.hidden_mapping[i] != session.hidden_mapping[i - 1] {
                runs += 1;
            }
        }

        // Expected runs ≈ n/2 for random data
        let expected_runs = 50.0;
        let deviation = (runs as f64 - expected_runs).abs() / expected_runs;

        assert!(deviation < 0.3, "Trials may not be independent");
    }

    #[test]
    fn test_binomial_test() {
        // 20/20 correct should be highly significant
        let p = binomial_test(20, 20, 0.5);
        assert!(p < 0.001);

        // 10/20 correct should not be significant (random guessing)
        let p = binomial_test(10, 20, 0.5);
        assert!(p > 0.05);
    }
}
```

## Reference Materials

- `references/statistical_tests.md` - Detailed statistical methods
- `references/experimental_design.md` - Best practices for audio testing
- `references/sample_size_calculator.md` - Power analysis formulas
