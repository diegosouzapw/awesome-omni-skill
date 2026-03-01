---
name: calculator
description: Performs basic arithmetic operations including addition, subtraction, multiplication, and division. Use when the user needs to calculate numerical results, perform math operations, or solve arithmetic problems.
license: MIT
metadata:
  author: agent-skills-poc
  version: "1.0"
  category: math
---

# Calculator Skill

This skill performs basic arithmetic calculations.

## When to Use

Use this skill when:

- The user asks to perform mathematical calculations
- Addition, subtraction, multiplication, or division is needed
- Numerical results need to be computed
- The user mentions terms like "calculate", "add", "subtract", "multiply", "divide", "sum", "difference", "product", or "quotient"

## Instructions

1. **Identify the operation** the user wants to perform:
   - Addition: combining two or more numbers
   - Subtraction: finding the difference between numbers
   - Multiplication: repeated addition or scaling
   - Division: splitting into equal parts

2. **Extract the numbers** from the user's request

3. **Perform the calculation**:
   - For addition: Add all numbers together
   - For subtraction: Subtract the second number from the first
   - For multiplication: Multiply the numbers together
   - For division: Divide the first number by the second

4. **Handle edge cases**:
   - Division by zero: Return an error stating "Division by zero is not allowed"
   - Very large numbers: Use appropriate precision
   - Decimal numbers: Maintain precision to at least 2 decimal places

5. **Present the result** clearly with the operation performed

## Examples

### Example 1: Addition

**Input:** "What is 15 + 27?"
**Steps:**

1. Operation: Addition
2. Numbers: 15 and 27
3. Calculation: 15 + 27 = 42
**Output:** "The sum of 15 and 27 is 42"

### Example 2: Division

**Input:** "Divide 100 by 4"
**Steps:**

1. Operation: Division
2. Numbers: 100 and 4
3. Calculation: 100 ÷ 4 = 25
**Output:** "100 divided by 4 equals 25"

### Example 3: Multiplication with decimals

**Input:** "Calculate 3.5 times 8"
**Steps:**

1. Operation: Multiplication
2. Numbers: 3.5 and 8
3. Calculation: 3.5 × 8 = 28
**Output:** "3.5 multiplied by 8 equals 28"

### Example 4: Division by zero (error case)

**Input:** "What is 10 divided by 0?"
**Steps:**

1. Operation: Division
2. Numbers: 10 and 0
3. Detect division by zero
**Output:** "Error: Division by zero is not allowed. Please provide a non-zero divisor."

## Common Edge Cases

- **Division by zero**: Always check if the divisor is zero before performing division
- **Negative numbers**: Handle negative results appropriately
- **Decimal precision**: Round results to a reasonable number of decimal places (typically 2-4)
- **Very large numbers**: Be aware of potential overflow issues
- **Multiple operations**: If the user requests multiple operations, perform them in the correct order (following PEMDAS/BODMAS if applicable)

## Tips

- Always confirm the operation and numbers with the user if ambiguous
- Show your work when helpful for understanding
- Use clear mathematical notation in your response
- For complex expressions, break them down into simpler steps
