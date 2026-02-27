---
name: generate_quiz_batch
description: Generates batches of high quality quizzes in JSON format for the BlindsidedGames pool, categorised and rated by difficulty.
---

# Generate Quiz Batch Skill

Use this skill to generate specific numbers of quiz files for the BlindsidedGames quiz pool.

## Requirements
When the user asks you to generate quizzes, they will specify a topic/category, a difficulty level (Easy, Medium, Hard), and the **configurable number of quizzes** they want to generate in a batch (e.g., "generate 5 distinct Easy History quizzes").

For **EACH** quiz requested in the batch (repeat this process N times):
1. Generate exactly 10 questions on the requested category at the requested difficulty. Ensure they are unique compared to other quizzes in the batch.
2. The 10 questions MUST follow this exact distribution:
   - 1 `multiple-choice` question
   - 2 `true-false` questions
   - 7 `self-eval` questions
3. Ensure the quality of the questions is appropriate for the target audience. CRITICAL: Do not include the answer (or obvious parts of the answer) within the wording of the question itself (e.g., asking "What insects build anthills?" when the answer is "Ants"). 
4. Output the quiz as a JSON file saved in `quizzes/`. The filename format should be `[category]_[difficulty]_[xx].json` (e.g. `biology_hard_01.json`).

## JSON Schema Example
```json
{
  "id": "biology_hard_01",
  "title": "Hard Biology Quiz 1",
  "sections": [
    {
      "title": "Biology",
      "items": [
        {
          "type": "multiple-choice",
          "q": "What is the powerhouse of the cell?",
          "options": ["Nucleus", "Ribosome", "Mitochondria", "Lysosome"],
          "a": "Mitochondria",
          "explanation": "Mitochondria generate most of the chemical energy needed to power the cell's biochemical reactions."
        },
        {
          "type": "true-false",
          "q": "All mammals give birth to live young.",
          "options": ["True", "False"],
          "a": "False",
          "explanation": "Monotremes like the platypus are mammals that lay eggs."
        },
        {
          "type": "self-eval",
          "q": "Describe the process of photosynthesis.",
          "a": "Photosynthesis is the process by which plants convert light energy into chemical energy, using water and carbon dioxide to create oxygen and energy in the form of sugar.",
          "explanation": "This requires chlorophyll and sunlight."
        }
        // ... 7 more items to total 10
      ]
    }
  ]
}
```

## Post-Generation Steps
After writing the quiz JSON files, you MUST update `quizzes/manifest.json`.
1. Parse the existing `quizzes/manifest.json`.
2. Append each new quiz you generated into the `quizzes` array with this format:
```json
{
  "id": "biology_hard_01",
  "category": "Biology",
  "difficulty": "Hard",
  "file": "biology_hard_01.json"
}
```
3. Save the updated `manifest.json`.
