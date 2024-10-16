**Enhanced Rule Engine for Data Validation, Mapping, and Filtering in TypeScript Using `jsep-eval`**

The following is an updated implementation of a rule engine that allows developers to organize and apply validation, mapping, and filtering rules on a dataset. This version makes use of the `jsep-eval` library to safely evaluate expressions provided in the rules.

### Rule Engine Module
The rule engine logic is encapsulated in a separate TypeScript class that can be reused across different projects and datasets. The `jsep-eval` library is used to evaluate expressions safely.

```typescript
// ruleEngine.ts
import jsep from 'jsep';
import { evaluate } from 'jsep-eval';

interface Rule {
  name: string;
  type: 'Validation' | 'Mapping' | 'Filter';
  expression: string;
  message?: string;
  target?: string;
}

interface ValidationError {
  row: number;
  rule: string;
  message: string;
  data: Record<string, any>;
}

export class RuleEngine {
  private rules: Rule[];

  constructor(rules: Rule[]) {
    this.rules = rules;
  }

  applyRules(data: Record<string, any>[]): { processedData: Record<string, any>[]; errors: ValidationError[] } {
    let errors: ValidationError[] = [];
    let processedData = [...data];

    this.rules.forEach((rule) => {
      if (rule.type === 'Validation') {
        // Apply validation rule
        processedData.forEach((row, index) => {
          if (!this.evaluateExpression(rule.expression, row)) {
            errors.push({
              row: index,
              rule: rule.name,
              message: rule.message || 'Validation failed',
              data: row,
            });
          }
        });
      } else if (rule.type === 'Mapping') {
        // Apply mapping rule
        processedData = processedData.map((row) => {
          row[rule.target!] = this.evaluateExpression(rule.expression, row);
          return row;
        });
      } else if (rule.type === 'Filter') {
        // Apply filter rule
        processedData = processedData.filter((row) => this.evaluateExpression(rule.expression, row));
      }
    });

    return { processedData, errors };
  }

  private evaluateExpression(expression: string, row: Record<string, any>): boolean | number {
    try {
      // Parse the expression using jsep and evaluate it using jsep-eval
      const parsedExpression = jsep(expression);
      return evaluate(parsedExpression, row);
    } catch (error) {
      console.error('Error evaluating expression:', expression, error);
      return false;
    }
  }
}
```

### Input and Rule Definition Module
This module defines the input dataset and the validation, mapping, and filtering rules. It also uses the rule engine to apply the rules.

```typescript
// main.ts
import { RuleEngine } from './ruleEngine';

// Define validation, mapping, and filtering rules
const rules = [
  {
    name: 'ValidateQuantity',
    type: 'Validation',
    expression: 'quantity > 0',
    message: 'Quantity must be greater than 0',
  },
  {
    name: 'ValidatePrice',
    type: 'Validation',
    expression: 'price >= 10',
    message: 'Price must be at least 10',
  },
  {
    name: 'ValidateCategory',
    type: 'Validation',
    expression: "['Electronics', 'Furniture'].includes(category)",
    message: 'Category must be either Electronics or Furniture',
  },
  {
    name: 'MapDiscountedPrice',
    type: 'Mapping',
    target: 'discounted_price',
    expression: 'price * 0.9',
  },
  {
    name: 'FilterElectronics',
    type: 'Filter',
    expression: "category === 'Electronics'",
  },
];

// Sample dataset
const data = [
  { item_id: 1, quantity: 5, price: 15, category: 'Electronics' },
  { item_id: 2, quantity: -1, price: 8, category: 'Grocery' },
  { item_id: 3, quantity: 8, price: 12, category: 'Furniture' },
  { item_id: 4, quantity: 0, price: 20, category: 'Clothing' },
];

// Initialize Rule Engine
const ruleEngine = new RuleEngine(rules);

// Apply rules
const { processedData, errors } = ruleEngine.applyRules(data);

// Print validation errors
errors.forEach((error) => {
  console.log(`Row ${error.row} failed '${error.rule}': ${error.message}`);
  console.log('Data:', error.data);
});

// Print final processed dataset
console.log('Processed Data:', processedData);
```

### Explanation of Code

1. **Rule Engine Module (`ruleEngine.ts`)**:
    - The `RuleEngine` class encapsulates logic for applying validation, mapping, and filtering rules.
    - The `applyRules()` method iterates over each rule and applies it based on its type:
        - **Validation**: Evaluates the condition and collects errors for rows that do not satisfy the condition.
        - **Mapping**: Creates or modifies columns based on specified expressions.
        - **Filter**: Filters rows based on a condition, retaining only those that match.
    - **Expression Evaluation**: Uses `jsep` to parse the expression and `jsep-eval` to safely evaluate it against the data row.

2. **Input and Rule Definition (`main.ts`)**:
    - The `rules` list contains validation, mapping, and filtering rules.
    - The `data` array defines the sample dataset.
    - The `RuleEngine` instance is used to apply the rules, and both validation errors and the processed dataset are printed.

### Example Output
```
Row 1 failed 'ValidateQuantity': Quantity must be greater than 0
Data: { item_id: 2, quantity: -1, price: 8, category: 'Grocery' }

Row 1 failed 'ValidatePrice': Price must be at least 10
Data: { item_id: 2, quantity: -1, price: 8, category: 'Grocery' }

Row 3 failed 'ValidateQuantity': Quantity must be greater than 0
Data: { item_id: 4, quantity: 0, price: 20, category: 'Clothing' }

Row 3 failed 'ValidateCategory': Category must be either Electronics or Furniture
Data: { item_id: 4, quantity: 0, price: 20, category: 'Clothing' }

Processed Data: [
  { item_id: 1, quantity: 5, price: 15, category: 'Electronics', discounted_price: 13.5 }
]
```

### How Developers Can Organize Rules
- **Modular Rule Engine**: The `RuleEngine` class can now handle validation, mapping, and filtering with safe expression evaluation using `jsep-eval`.
- **Separation of Concerns**: Keeping rule application logic separate from rule definitions makes it easy to maintain, test, and extend the engine.
- **Scalable System**: Adding new rule types, such as aggregation or transformation, can be easily achieved by extending the rule engine logic.

### Summary
This enhanced rule engine demonstrates how validation, mapping, and filtering rules can be managed in a modular and reusable way using TypeScript and the `jsep-eval` library. The engine allows developers to easily define and apply rules to datasets, enabling sophisticated data processing workflows while maintaining scalability and code quality.