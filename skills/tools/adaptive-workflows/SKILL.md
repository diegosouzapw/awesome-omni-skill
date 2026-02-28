---
name: adaptive-workflows
description: Self-learning workflow system that tracks what works best for your use cases. Records experiment results, suggests optimizations, creates custom templates, and builds a personal knowledge base. Use to learn from experience and optimize your LLM workflows over time.
---

# Adaptive Workflows

Build a self-learning system that gets better with every experiment you run.

## Overview

Learn from experience:

- **Experiment tracking** - Record all experiments and results
- **Pattern recognition** - Identify what works best for your use cases
- **Smart recommendations** - Get suggestions based on past success
- **Workflow templates** - Create reusable templates from successful experiments
- **A/B testing** - Compare approaches systematically
- **Knowledge base** - Build your personal best practices library
- **Continuous improvement** - Workflows get better over time

## Quick Start

### Initialize Workflow Tracker

```python
import json
from datetime import datetime
from pathlib import Path

class WorkflowTracker:
    def __init__(self, storage_path="./workflows.json"):
        self.storage_path = Path(storage_path)
        self.experiments = self.load_experiments()

    def load_experiments(self):
        """Load previous experiments"""
        if self.storage_path.exists():
            with open(self.storage_path, 'r') as f:
                return json.load(f)
        return []

    def save_experiments(self):
        """Save experiments to disk"""
        with open(self.storage_path, 'w') as f:
            json.dump(self.experiments, f, indent=2)

    def record_experiment(self, experiment: dict):
        """Record a new experiment"""
        experiment['timestamp'] = datetime.now().isoformat()
        experiment['id'] = len(self.experiments)
        self.experiments.append(experiment)
        self.save_experiments()
        return experiment['id']

# Initialize
tracker = WorkflowTracker()
```

### Record Your First Experiment

```python
# After training
experiment = {
    'task': 'medical_qa_finetuning',
    'model': 'Llama-3.2-7B',
    'dataset_size': 1000,
    'hyperparameters': {
        'learning_rate': 2e-4,
        'lora_rank': 16,
        'batch_size': 8,
        'epochs': 3
    },
    'results': {
        'final_loss': 0.42,
        'training_time_hours': 2.5,
        'cost_usd': 5.20,
        'eval_accuracy': 0.89
    },
    'notes': 'Worked well, converged smoothly'
}

tracker.record_experiment(experiment)
```

### Get Recommendations

```python
# Get recommendations for similar task
recommendations = tracker.suggest_config(
    task='medical_qa_finetuning',
    constraints={'cost': 10, 'time_hours': 4}
)

print(f"Recommended learning rate: {recommendations['learning_rate']}")
print(f"Recommended rank: {recommendations['lora_rank']}")
```

## Experiment Tracking

### Complete Experiment Schema

```python
experiment_schema = {
    # Identifiers
    'id': 'auto-generated',
    'timestamp': 'ISO-8601',
    'task': 'task_category',  # e.g., 'medical_qa', 'code_generation'

    # Model configuration
    'model': {
        'base_model': 'Llama-3.2-7B',
        'quantization': '4bit',
        'max_seq_length': 2048
    },

    # Dataset
    'dataset': {
        'name': 'medical_qa_v1',
        'size': 1000,
        'format': 'alpaca',
        'quality_score': 0.85,
        'source': 'synthetic'
    },

    # Hyperparameters
    'hyperparameters': {
        'learning_rate': 2e-4,
        'lr_scheduler': 'cosine',
        'warmup_ratio': 0.03,
        'lora_rank': 16,
        'lora_alpha': 16,
        'batch_size': 8,
        'gradient_accumulation': 1,
        'epochs': 3,
        'weight_decay': 0.01
    },

    # Results
    'results': {
        'final_loss': 0.42,
        'best_loss': 0.38,
        'eval_loss': 0.45,
        'eval_accuracy': 0.89,
        'training_time_hours': 2.5,
        'cost_usd': 5.20,
        'tokens_per_sec': 1200,
        'peak_memory_gb': 18.5
    },

    # Deployment
    'deployment': {
        'format': 'GGUF',
        'quantization': 'q4_k_m',
        'deployment_platform': 'Ollama',
        'inference_speed_tok_s': 50
    },

    # Evaluation
    'evaluation': {
        'test_accuracy': 0.87,
        'human_eval_score': 4.2,  # out of 5
        'sample_outputs': ['...', '...'],
        'issues_found': ['Occasional hallucination on rare conditions']
    },

    # Metadata
    'tags': ['medical', 'qa', 'production'],
    'success': True,
    'notes': 'Worked well. Deployed to production.',
    'git_commit': 'abc123'
}
```

### Track Experiment

```python
class ExperimentTracker(WorkflowTracker):
    def track_training_run(
        self,
        task: str,
        config: dict,
        trainer,  # SFTTrainer instance
        dataset_info: dict
    ):
        """Automatically track a training run"""

        experiment = {
            'task': task,
            'model': config['model_name'],
            'dataset': dataset_info,
            'hyperparameters': {
                'learning_rate': config['learning_rate'],
                'lora_rank': config['lora_rank'],
                'lora_alpha': config['lora_alpha'],
                'batch_size': config['batch_size'],
                'epochs': config['num_epochs']
            }
        }

        # Train
        import time
        start_time = time.time()

        result = trainer.train()

        training_time = (time.time() - start_time) / 3600  # hours

        # Record results
        experiment['results'] = {
            'final_loss': result.training_loss,
            'training_time_hours': training_time,
            'cost_usd': self.estimate_cost(training_time, config['gpu_type'])
        }

        # Save
        exp_id = self.record_experiment(experiment)
        print(f"Experiment {exp_id} recorded")

        return exp_id

    def estimate_cost(self, hours: float, gpu_type: str) -> float:
        """Estimate training cost"""
        gpu_costs = {
            'RTX_4090': 0.40,
            'A100': 1.50,
            'H100': 3.00
        }
        return hours * gpu_costs.get(gpu_type, 1.0)

# Usage
tracker = ExperimentTracker()

exp_id = tracker.track_training_run(
    task='medical_qa',
    config={...},
    trainer=trainer,
    dataset_info={...}
)
```

## Pattern Recognition

### Identify Best Configurations

```python
class PatternAnalyzer(ExperimentTracker):
    def find_best_experiments(
        self,
        task: str,
        metric: str = 'eval_accuracy',
        top_k: int = 5
    ):
        """Find top performing experiments for a task"""

        # Filter by task
        task_experiments = [
            exp for exp in self.experiments
            if exp['task'] == task and 'results' in exp
        ]

        # Sort by metric
        sorted_exps = sorted(
            task_experiments,
            key=lambda x: x['results'].get(metric, 0),
            reverse=True
        )

        return sorted_exps[:top_k]

    def analyze_hyperparameter_impact(self, task: str, hyperparameter: str):
        """Analyze how a hyperparameter affects results"""

        task_exps = [exp for exp in self.experiments if exp['task'] == task]

        # Group by hyperparameter value
        from collections import defaultdict
        groups = defaultdict(list)

        for exp in task_exps:
            value = exp['hyperparameters'].get(hyperparameter)
            if value and 'results' in exp:
                groups[value].append(exp['results'].get('eval_accuracy', 0))

        # Calculate averages
        analysis = {
            value: {
                'count': len(scores),
                'mean_accuracy': np.mean(scores),
                'std': np.std(scores)
            }
            for value, scores in groups.items()
        }

        return analysis

# Usage
analyzer = PatternAnalyzer()

# Find best medical QA experiments
best = analyzer.find_best_experiments('medical_qa', top_k=3)
for exp in best:
    print(f"Experiment {exp['id']}: {exp['results']['eval_accuracy']:.3f}")

# Analyze learning rate impact
lr_analysis = analyzer.analyze_hyperparameter_impact('medical_qa', 'learning_rate')
for lr, stats in lr_analysis.items():
    print(f"LR {lr}: {stats['mean_accuracy']:.3f} ± {stats['std']:.3f}")
```

### Detect Success Patterns

```python
def detect_patterns(self, task: str):
    """Detect what makes experiments successful"""

    successful = [
        exp for exp in self.experiments
        if exp['task'] == task
        and exp['results'].get('eval_accuracy', 0) > 0.85
    ]

    failed = [
        exp for exp in self.experiments
        if exp['task'] == task
        and exp['results'].get('eval_accuracy', 0) < 0.70
    ]

    patterns = {}

    # Analyze each hyperparameter
    for hyperparam in ['learning_rate', 'lora_rank', 'batch_size']:
        success_values = [
            exp['hyperparameters'][hyperparam]
            for exp in successful
            if hyperparam in exp['hyperparameters']
        ]

        fail_values = [
            exp['hyperparameters'][hyperparam]
            for exp in failed
            if hyperparam in exp['hyperparameters']
        ]

        patterns[hyperparam] = {
            'success_mean': np.mean(success_values),
            'success_std': np.std(success_values),
            'fail_mean': np.mean(fail_values),
            'fail_std': np.std(fail_values),
            'recommendation': 'Use higher value' if np.mean(success_values) > np.mean(fail_values) else 'Use lower value'
        }

    return patterns
```

## Smart Recommendations

### Configuration Recommender

```python
class ConfigRecommender(PatternAnalyzer):
    def suggest_config(
        self,
        task: str,
        constraints: dict = None
    ):
        """Suggest optimal configuration based on past experiments"""

        # Get successful experiments
        successful_exps = [
            exp for exp in self.experiments
            if exp['task'] == task
            and exp['results'].get('eval_accuracy', 0) > 0.80
        ]

        if not successful_exps:
            return self.get_default_config(task)

        # Apply constraints
        if constraints:
            successful_exps = self.filter_by_constraints(
                successful_exps,
                constraints
            )

        if not successful_exps:
            return self.get_default_config(task)

        # Find best configuration
        best_exp = max(
            successful_exps,
            key=lambda x: x['results'].get('eval_accuracy', 0)
        )

        # Extract configuration
        recommended = {
            'model': best_exp['model'],
            'hyperparameters': best_exp['hyperparameters'],
            'dataset': best_exp['dataset'],
            'confidence': self.calculate_confidence(successful_exps),
            'based_on_experiments': len(successful_exps),
            'expected_accuracy': best_exp['results'].get('eval_accuracy'),
            'expected_cost': best_exp['results'].get('cost_usd'),
            'expected_time_hours': best_exp['results'].get('training_time_hours')
        }

        return recommended

    def filter_by_constraints(self, experiments, constraints):
        """Filter experiments by constraints"""

        filtered = experiments

        if 'max_cost' in constraints:
            filtered = [
                exp for exp in filtered
                if exp['results'].get('cost_usd', float('inf')) <= constraints['max_cost']
            ]

        if 'max_time_hours' in constraints:
            filtered = [
                exp for exp in filtered
                if exp['results'].get('training_time_hours', float('inf')) <= constraints['max_time_hours']
            ]

        if 'min_accuracy' in constraints:
            filtered = [
                exp for exp in filtered
                if exp['results'].get('eval_accuracy', 0) >= constraints['min_accuracy']
            ]

        return filtered

    def calculate_confidence(self, experiments):
        """Calculate confidence in recommendation"""

        if len(experiments) >= 10:
            return 'high'
        elif len(experiments) >= 5:
            return 'medium'
        else:
            return 'low'

# Usage
recommender = ConfigRecommender()

# Get recommendation with constraints
config = recommender.suggest_config(
    task='medical_qa',
    constraints={
        'max_cost': 10,
        'max_time_hours': 3,
        'min_accuracy': 0.85
    }
)

print(f"Recommended learning rate: {config['hyperparameters']['learning_rate']}")
print(f"Expected accuracy: {config['expected_accuracy']:.3f}")
print(f"Confidence: {config['confidence']}")
print(f"Based on {config['based_on_experiments']} similar experiments")
```

### Automated Optimization Suggestions

```python
def suggest_improvements(self, exp_id: int):
    """Suggest improvements for a specific experiment"""

    exp = self.experiments[exp_id]
    suggestions = []

    # Compare to best experiments
    best_exps = self.find_best_experiments(exp['task'], top_k=3)

    if not best_exps:
        return ["Not enough data for comparison"]

    best_exp = best_exps[0]
    current_accuracy = exp['results'].get('eval_accuracy', 0)
    best_accuracy = best_exp['results'].get('eval_accuracy', 0)

    if current_accuracy < best_accuracy:
        # Compare hyperparameters
        for hyperparam in ['learning_rate', 'lora_rank', 'batch_size']:
            current_value = exp['hyperparameters'].get(hyperparam)
            best_value = best_exp['hyperparameters'].get(hyperparam)

            if current_value != best_value:
                suggestions.append({
                    'parameter': hyperparam,
                    'current': current_value,
                    'suggested': best_value,
                    'reason': f'Top experiment uses {best_value}',
                    'expected_improvement': (best_accuracy - current_accuracy) * 100
                })

    # Check for common issues
    if exp['results'].get('training_time_hours', 0) > 5:
        suggestions.append({
            'issue': 'slow_training',
            'suggestion': 'Increase batch size or reduce gradient accumulation',
            'impact': 'Faster training'
        })

    if exp['results'].get('final_loss') > exp['results'].get('best_loss', float('inf')) * 1.5:
        suggestions.append({
            'issue': 'unstable_training',
            'suggestion': 'Reduce learning rate or add gradient clipping',
            'impact': 'More stable convergence'
        })

    return suggestions
```

## Workflow Templates

### Create Template from Experiment

```python
class WorkflowTemplateManager(ConfigRecommender):
    def __init__(self, storage_path="./workflows.json"):
        super().__init__(storage_path)
        self.templates = self.load_templates()

    def load_templates(self):
        """Load workflow templates"""
        template_path = Path(self.storage_path).parent / "templates.json"
        if template_path.exists():
            with open(template_path, 'r') as f:
                return json.load(f)
        return []

    def save_templates(self):
        """Save templates"""
        template_path = Path(self.storage_path).parent / "templates.json"
        with open(template_path, 'w') as f:
            json.dump(self.templates, f, indent=2)

    def create_template(
        self,
        name: str,
        exp_id: int,
        description: str = ""
    ):
        """Create a reusable template from successful experiment"""

        exp = self.experiments[exp_id]

        template = {
            'name': name,
            'description': description,
            'task': exp['task'],
            'created_from': exp_id,
            'created_at': datetime.now().isoformat(),
            'config': {
                'model': exp['model'],
                'dataset': exp['dataset'],
                'hyperparameters': exp['hyperparameters']
            },
            'expected_results': exp['results'],
            'usage_count': 0
        }

        self.templates.append(template)
        self.save_templates()

        return template

    def use_template(self, template_name: str, overrides: dict = None):
        """Use a template with optional overrides"""

        template = next(
            (t for t in self.templates if t['name'] == template_name),
            None
        )

        if not template:
            raise ValueError(f"Template '{template_name}' not found")

        # Start with template config
        config = template['config'].copy()

        # Apply overrides
        if overrides:
            for key, value in overrides.items():
                if '.' in key:  # Nested key like 'hyperparameters.learning_rate'
                    parts = key.split('.')
                    d = config
                    for part in parts[:-1]:
                        d = d[part]
                    d[parts[-1]] = value
                else:
                    config[key] = value

        # Update usage count
        template['usage_count'] += 1
        self.save_templates()

        return config

    def list_templates(self, task: str = None):
        """List available templates"""

        templates = self.templates

        if task:
            templates = [t for t in templates if t['task'] == task]

        return [
            {
                'name': t['name'],
                'task': t['task'],
                'description': t['description'],
                'usage_count': t['usage_count'],
                'expected_accuracy': t['expected_results'].get('eval_accuracy')
            }
            for t in templates
        ]

# Usage
manager = WorkflowTemplateManager()

# Create template from best experiment
template = manager.create_template(
    name='medical_qa_standard',
    exp_id=42,  # ID of successful experiment
    description='Standard configuration for medical Q&A fine-tuning'
)

# Use template
config = manager.use_template(
    'medical_qa_standard',
    overrides={'hyperparameters.learning_rate': 3e-4}
)

# List templates
templates = manager.list_templates(task='medical_qa')
for t in templates:
    print(f"{t['name']}: {t['expected_accuracy']:.3f} accuracy (used {t['usage_count']} times)")
```

## A/B Testing

### Compare Configurations

```python
class ABTester(WorkflowTemplateManager):
    def setup_ab_test(
        self,
        task: str,
        config_a: dict,
        config_b: dict,
        test_name: str,
        num_runs: int = 3
    ):
        """Setup A/B test to compare two configurations"""

        test = {
            'name': test_name,
            'task': task,
            'config_a': config_a,
            'config_b': config_b,
            'num_runs': num_runs,
            'completed_runs': 0,
            'results_a': [],
            'results_b': [],
            'status': 'pending'
        }

        # Save test
        tests_path = Path(self.storage_path).parent / "ab_tests.json"
        tests = []
        if tests_path.exists():
            with open(tests_path, 'r') as f:
                tests = json.load(f)

        tests.append(test)

        with open(tests_path, 'w') as f:
            json.dump(tests, f, indent=2)

        return test

    def record_ab_result(
        self,
        test_name: str,
        config: str,  # 'a' or 'b'
        results: dict
    ):
        """Record results from one run of A/B test"""

        tests_path = Path(self.storage_path).parent / "ab_tests.json"
        with open(tests_path, 'r') as f:
            tests = json.load(f)

        # Find test
        test = next((t for t in tests if t['name'] == test_name), None)
        if not test:
            raise ValueError(f"Test '{test_name}' not found")

        # Record result
        if config == 'a':
            test['results_a'].append(results)
        else:
            test['results_b'].append(results)

        test['completed_runs'] += 1

        # Check if complete
        if (len(test['results_a']) >= test['num_runs'] and
            len(test['results_b']) >= test['num_runs']):
            test['status'] = 'complete'
            test['analysis'] = self.analyze_ab_results(test)

        # Save
        with open(tests_path, 'w') as f:
            json.dump(tests, f, indent=2)

        return test

    def analyze_ab_results(self, test: dict):
        """Analyze A/B test results"""

        results_a = test['results_a']
        results_b = test['results_b']

        # Extract metrics
        accuracy_a = [r['eval_accuracy'] for r in results_a]
        accuracy_b = [r['eval_accuracy'] for r in results_b]

        cost_a = [r['cost_usd'] for r in results_a]
        cost_b = [r['cost_usd'] for r in results_b]

        time_a = [r['training_time_hours'] for r in results_a]
        time_b = [r['training_time_hours'] for r in results_b]

        # Statistical analysis
        from scipy import stats

        # T-test for accuracy
        t_stat, p_value = stats.ttest_ind(accuracy_a, accuracy_b)

        analysis = {
            'accuracy': {
                'config_a_mean': np.mean(accuracy_a),
                'config_a_std': np.std(accuracy_a),
                'config_b_mean': np.mean(accuracy_b),
                'config_b_std': np.std(accuracy_b),
                'difference': np.mean(accuracy_b) - np.mean(accuracy_a),
                'p_value': p_value,
                'significant': p_value < 0.05
            },
            'cost': {
                'config_a_mean': np.mean(cost_a),
                'config_b_mean': np.mean(cost_b),
                'difference': np.mean(cost_b) - np.mean(cost_a)
            },
            'time': {
                'config_a_mean': np.mean(time_a),
                'config_b_mean': np.mean(time_b),
                'difference': np.mean(time_b) - np.mean(time_a)
            },
            'winner': 'b' if np.mean(accuracy_b) > np.mean(accuracy_a) else 'a',
            'recommendation': self.generate_recommendation(
                accuracy_a, accuracy_b,
                cost_a, cost_b,
                time_a, time_b
            )
        }

        return analysis

    def generate_recommendation(
        self,
        accuracy_a, accuracy_b,
        cost_a, cost_b,
        time_a, time_b
    ):
        """Generate recommendation from A/B test"""

        acc_diff = np.mean(accuracy_b) - np.mean(accuracy_a)
        cost_diff = np.mean(cost_b) - np.mean(cost_a)
        time_diff = np.mean(time_b) - np.mean(time_a)

        if acc_diff > 0.02:  # B is 2% better
            if cost_diff < 5:  # And not much more expensive
                return "Use Config B: significantly better accuracy at similar cost"
            else:
                return f"Use Config B if budget allows: {acc_diff*100:.1f}% better accuracy, ${cost_diff:.2f} more expensive"
        elif acc_diff < -0.02:  # A is better
            return "Use Config A: better accuracy"
        else:  # Similar accuracy
            if cost_diff < -2:  # B is cheaper
                return "Use Config B: similar accuracy, lower cost"
            else:
                return "Configs perform similarly, choose based on other factors"

# Usage
tester = ABTester()

# Setup A/B test
test = tester.setup_ab_test(
    task='medical_qa',
    config_a={'learning_rate': 2e-4, 'lora_rank': 16},
    config_b={'learning_rate': 5e-4, 'lora_rank': 32},
    test_name='lr_and_rank_test',
    num_runs=3
)

# After each training run
tester.record_ab_result(
    'lr_and_rank_test',
    config='a',
    results={'eval_accuracy': 0.87, 'cost_usd': 5.2, 'training_time_hours': 2.5}
)

# After all runs complete, get analysis
test = tester.get_test_results('lr_and_rank_test')
print(test['analysis']['recommendation'])
```

## Knowledge Base

### Build Personal Best Practices

````python
class KnowledgeBase(ABTester):
    def extract_insights(self):
        """Extract insights from all experiments"""

        insights = {
            'total_experiments': len(self.experiments),
            'tasks': {},
            'best_practices': [],
            'common_issues': []
        }

        # Group by task
        from collections import defaultdict
        by_task = defaultdict(list)
        for exp in self.experiments:
            by_task[exp['task']].append(exp)

        # Analyze each task
        for task, exps in by_task.items():
            successful = [e for e in exps if e['results'].get('eval_accuracy', 0) > 0.80]

            if len(successful) < 3:
                continue  # Not enough data

            insights['tasks'][task] = {
                'total_experiments': len(exps),
                'success_rate': len(successful) / len(exps),
                'avg_accuracy': np.mean([e['results']['eval_accuracy'] for e in successful]),
                'avg_cost': np.mean([e['results'].get('cost_usd', 0) for e in successful]),
                'optimal_config': self.suggest_config(task)
            }

        # Extract best practices
        insights['best_practices'] = self.extract_best_practices()

        # Identify common issues
        insights['common_issues'] = self.identify_common_issues()

        return insights

    def extract_best_practices(self):
        """Extract best practices from successful experiments"""

        practices = []

        # Analyze learning rate
        lr_analysis = self.analyze_all_hyperparameter('learning_rate')
        if lr_analysis:
            practices.append({
                'category': 'learning_rate',
                'recommendation': f"Use learning rate around {lr_analysis['optimal']:.0e}",
                'confidence': lr_analysis['confidence'],
                'data_points': lr_analysis['count']
            })

        # Analyze LoRA rank
        rank_analysis = self.analyze_all_hyperparameter('lora_rank')
        if rank_analysis:
            practices.append({
                'category': 'lora_rank',
                'recommendation': f"Use LoRA rank {rank_analysis['optimal']}",
                'confidence': rank_analysis['confidence'],
                'data_points': rank_analysis['count']
            })

        return practices

    def identify_common_issues(self):
        """Identify common issues from failed experiments"""

        issues = []

        failed = [
            exp for exp in self.experiments
            if exp['results'].get('eval_accuracy', 1.0) < 0.70
        ]

        if not failed:
            return []

        # Check for common patterns in failures
        # High loss
        high_loss = [
            exp for exp in failed
            if exp['results'].get('final_loss', 0) > 1.0
        ]
        if len(high_loss) > len(failed) * 0.3:
            issues.append({
                'issue': 'High final loss in 30%+ of failed experiments',
                'suggestion': 'Consider longer training or higher learning rate',
                'affected_experiments': len(high_loss)
            })

        # Slow training
        slow = [
            exp for exp in failed
            if exp['results'].get('training_time_hours', 0) > 5
        ]
        if len(slow) > len(failed) * 0.3:
            issues.append({
                'issue': 'Slow training in failed experiments',
                'suggestion': 'Optimize batch size and gradient checkpointing',
                'affected_experiments': len(slow)
            })

        return issues

    def generate_report(self):
        """Generate comprehensive report"""

        insights = self.extract_insights()

        report = f"""
# Adaptive Workflow Report

Generated: {datetime.now().strftime('%Y-%m-%d %H:%M')}

## Summary

- Total Experiments: {insights['total_experiments']}
- Tasks Tracked: {len(insights['tasks'])}

## Task Performance

"""

        for task, stats in insights['tasks'].items():
            report += f"""
### {task}

- Experiments: {stats['total_experiments']}
- Success Rate: {stats['success_rate']:.1%}
- Average Accuracy: {stats['avg_accuracy']:.3f}
- Average Cost: ${stats['avg_cost']:.2f}

**Recommended Configuration:**
```python
{json.dumps(stats['optimal_config']['hyperparameters'], indent=2)}
````

---

"""

        report += "\n## Best Practices\n\n"
        for practice in insights['best_practices']:
            report += f"- **{practice['category']}**: {practice['recommendation']} "
            report += f"(Confidence: {practice['confidence']}, based on {practice['data_points']} experiments)\n"

        if insights['common_issues']:
            report += "\n## Common Issues\n\n"
            for issue in insights['common_issues']:
                report += f"- **{issue['issue']}**: {issue['suggestion']}\n"

        return report

# Usage

kb = KnowledgeBase()

# Generate report

report = kb.generate_report()
print(report)

# Save report

with open('workflow_report.md', 'w') as f:
f.write(report)

````

## Integration with Other Skills

### Auto-Apply Learnings

```python
class AdaptiveTrainer(KnowledgeBase):
    def smart_train(
        self,
        task: str,
        dataset,
        base_config: dict = None,
        auto_optimize: bool = True
    ):
        """Train with automatic optimization based on past experience"""

        # Get recommended config
        if auto_optimize:
            recommended = self.suggest_config(
                task=task,
                constraints=base_config.get('constraints', {})
            )

            print(f"Using optimized configuration based on {recommended['based_on_experiments']} past experiments")
            config = recommended
        else:
            config = base_config or {}

        # Setup model
        model, tokenizer = FastLanguageModel.from_pretrained(
            config['model'],
            max_seq_length=config.get('max_seq_length', 2048),
            load_in_4bit=True
        )

        # Setup LoRA
        model = FastLanguageModel.get_peft_model(
            model,
            r=config['hyperparameters']['lora_rank'],
            lora_alpha=config['hyperparameters']['lora_alpha'],
            target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                          "gate_proj", "up_proj", "down_proj"]
        )

        # Setup training
        trainer = SFTTrainer(
            model=model,
            tokenizer=tokenizer,
            train_dataset=dataset,
            args=TrainingArguments(
                learning_rate=config['hyperparameters']['learning_rate'],
                per_device_train_batch_size=config['hyperparameters']['batch_size'],
                num_train_epochs=config['hyperparameters']['epochs'],
                # ... other args
            )
        )

        # Train and track
        exp_id = self.track_training_run(
            task=task,
            config=config,
            trainer=trainer,
            dataset_info={'size': len(dataset)}
        )

        print(f"Training complete. Experiment ID: {exp_id}")

        # Get suggestions for next time
        suggestions = self.suggest_improvements(exp_id)
        if suggestions:
            print("\nSuggestions for next experiment:")
            for s in suggestions:
                print(f"  - {s}")

        return exp_id

# Usage
trainer = AdaptiveTrainer()

exp_id = trainer.smart_train(
    task='medical_qa',
    dataset=my_dataset,
    auto_optimize=True  # Use past learnings
)
````

## Best Practices

### 1. Record Everything

```python
# Always record experiments
# Even "failed" experiments provide valuable data
```

### 2. Be Consistent

```python
# Use consistent task names and metrics
# Makes pattern recognition more effective
```

### 3. Regular Analysis

```python
# Generate reports monthly
kb = KnowledgeBase()
report = kb.generate_report()

# Review and update templates
```

### 4. Version Control

```python
# Keep workflow files in git
# Track evolution of your best practices
```

### 5. Start Simple

```python
# Begin with basic tracking
# Add complexity as you gather data
```

## Summary

Adaptive workflows enable continuous improvement:

1. ✓ Track every experiment
2. ✓ Analyze patterns in success/failure
3. ✓ Get smart recommendations
4. ✓ Create reusable templates
5. ✓ Run A/B tests systematically
6. ✓ Build personal knowledge base
7. ✓ Improve with every iteration

**The more you use it, the smarter it gets.**

Start tracking today, and within 10-20 experiments, you'll have personalized best practices that work specifically for your use cases.
