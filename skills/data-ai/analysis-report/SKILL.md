---
name: analysis-report
description: Generates comprehensive, structured research reports.
---

# Scientific Analysis & Reporting

## Instructions

### 1. Project Exploration & Domain Mapping
Before analyzing data, map the scientific context of the repository:
- **Dependency & Logic Scan**: Check `pyproject.toml` for libraries and `main.py` (or equivalent) for the execution flow.
- **Consult References**: Check the `references/` directory for background materials, standard definitions (e.g., NEMA, IEC), or methodology specifications. Use these files to define terms and expected behaviors.
- **Identify Physical Models**: Locate the core logic defining the system (constants, equations like Inverse Square Law, statistical models).
- **Locate Data**: All experimental and simulation data is stored in `data/` with comprehensive filenames (e.g., `data/radial_positions_10min_run.csv`). Always inspect file headers to confirm units and column definitions. 
- **Locate Assets**: All assets like images or plots are stored in `assets/` with comprehensive filenames.

### 2. Data Analysis & Verification
Do not rely solely on existing summary text; verify findings by inspecting raw data or running code:
- **Execution**: If the environment allows, run analysis scripts (e.g., `uv run main.py`) to generate the most recent metrics. You are also allowed to write new Python files/scripts for analyzing data. If a package you need does not exist, you are allowed to use `uv add <package>` to add it. 
- **Extract Key Metrics**:
  - **Performance**: Efficiency, throughput, sensitivity, etc.
  - **Signal Quality**: SNR, Contrast, Resolution, etc.
  - **Statistics**: Mean, Standard Deviation, CV, etc.
- **Cross-Reference**: Compare your calculated results against theoretical expectations found in `references/`.

### 3. Goal Confirmation
**Crucial Step**: Before generating the full text of the report, pause and present a brief plan to the user to ensure alignment:
1.  **Objective**: State what you understand the primary goal to be (e.g., "I will compare the sensitivity of X vs Y").
2.  **Data Sources**: List the specific files you intend to use (e.g., "Using `data/contrast.csv` and `references/nema_standards.pdf`").
3.  **Proposed Structure**: Briefly outline the sections you will write.
4.  **Action**: Ask the user, "Does this plan match your requirements?" and wait for their confirmation or correction.

### 4. Report Generation
Unless otherwise specified, always consolidate findings into a new file named `docs/analysis-report.md`.

**Report Structure:**
1.  **Objective**: Define the goal (e.g., "Compare Method A vs. Method B").
2.  **Methodology**: Describe the experimental setup. Explicitly cite the specific data files used from `data/` and standards from `references/`.
3.  **Quantitative Results**: Present data in Markdown tables. Compare distinct groups (e.g., Control vs. Variable).
4.  **Discussion & Interpretation**:
    - Explain *why* the results occurred using the identified physical/math models.
    - Justify any approximations used in the code.
5.  **Conclusion**: Summary of the primary findings.

### 5. Writing Standards
- **Quantify Everything**: Avoid vague terms. Use "12.5% higher efficiency" rather than "better efficiency."
- **Writing Style**: Use a professional tone. Lean towards writing in natural language paragraphs instead of using bullet points or lists. 
- **Visuals**: If plots are generated, reference their filenames in the report.
- **Language**: Write in Simplified Chinese. For specific translations, see [translations.md](references/translations.md).
- **Headings**: 
    - Do not number headings. 
    - The title should not be a heading, so sections should use heading 1 instead of 2.
- **Formulas**: 
    - Use LaTex for isotopic notation (e.g., `^{99m}Tc`).
    - Use LaTeX-style formulas (e.g., `$E = mc^2$`).
    - Use `$$` to delimit multi-line formulas.


## Examples

### Example 1: General Performance Analysis
**User:** "Analyze the stability of the sensor data in this repo."
**Action:**
1.  Read `references/sensor_datasheet.md` to find the nominal operating range.
2.  Load `data/sensor_stability_log_24hours.csv`.
3.  Calculate mean and variance.
4.  **Generate `docs/analysis-report.md`**:
    - **Methods**: "Compared observed variance in `data/sensor_stability_log_24hours.csv` against specs in `references/sensor_datasheet.md`."
    - **Results**: Table showing stability metrics.
    - **Discussion**: Explain deviations based on noise models found in `main.py`.

### Example 2: Comparative Method Study
**User:** "Compare the simulation results between the 'Fast' and 'Accurate' algorithms."
**Action:**
1.  Locate `data/simulation_output_fast_algo.csv` and `data/simulation_output_accurate_algo.csv`.
2.  Compare key metrics: Execution time vs. Error rate.
3.  **Generate `docs/analysis-report.md`**:
    - **Objective**: "Evaluate trade-off between speed and precision."
    - **Results**: "The 'Fast' algorithm is 10x faster but introduces a 2.3% systematic error."
    - **Discussion**: Link the error to the approximation found in the code logic.