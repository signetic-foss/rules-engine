
```plaintext
rule-engine/
├── README.md
├── package.json
├── tsconfig.json
├── src/
│   ├── main.ts
│   ├── parsers/
│   │   ├── index.ts
│   │   └── ymlParser.ts
│   ├── engine/
│   │   ├── index.ts
│   │   └── ruleEngine.ts
│   ├── models/
│   │   ├── index.ts
│   │   ├── rule.ts
│   │   └── ruleTypes.ts
│   ├── utils/
│   │   └── expressionEvaluator.ts
│   ├── examples/
│   │   ├── data/
│   │   │   └── sampleData.json
│   │   └── rules/
│   │       └── exampleRules.yml
└── test/
    ├── parsers/
    │   └── ymlParser.test.ts
    ├── engine/
    │   └── ruleEngine.test.ts
    └── utils/
        └── expressionEvaluator.test.ts
```

## Folder and File Descriptions

### **Root Level**

- **`README.md`**: Provides an overview of the project, how to set it up, and how to use it.
- **`package.json`**: Contains project metadata and dependencies.
- **`tsconfig.json`**: TypeScript configuration file.

### **`src/` Directory**

Contains all the source code for the application.

#### **`main.ts`**

- The entry point of the application.
- Orchestrates the loading of rules and data, initializes the parser and engine, and executes the processing.

#### **`parsers/` Module**

Handles the parsing of YAML files into JavaScript objects.

- **`index.ts`**
    - Exports functionalities from the parsers module.
- **`ymlParser.ts`**
    - Contains logic to read and parse YAML files.
    - Converts YAML-defined rules into JavaScript objects that the rule engine can consume.

#### **`engine/` Module**

Contains the core logic for applying rules to datasets.

- **`index.ts`**
    - Exports functionalities from the engine module.
- **`ruleEngine.ts`**
    - Implements the `RuleEngine` class.
    - Contains methods to apply validation, mapping, and filtering rules to data.

#### **`models/` Module**

Defines TypeScript interfaces and types used across the application.

- **`index.ts`**
    - Re-exports all models for easy import elsewhere in the application.
- **`rule.ts`**
    - Defines the `Rule` interface representing the structure of a rule.
- **`ruleTypes.ts`**
    - Contains enumerations or constants defining possible rule types.

#### **`utils/` Module**

Utility functions and helpers.

- **`expressionEvaluator.ts`**
    - Wraps the logic for evaluating expressions safely, possibly using libraries like `jsep` and `jsep-eval`.
    - Ensures that the expression evaluation logic is centralized and reusable.

#### **`examples/` Directory**

Contains example data and rules for testing and demonstration purposes.

- **`data/`**
    - **`sampleData.json`**: A sample dataset to be used in examples.
- **`rules/`**
    - **`exampleRules.yml`**: An example YAML file defining rules.

### **`test/` Directory**

Contains all tests for the application, organized similarly to the `src/` directory.

#### **`parsers/`**

- **`ymlParser.test.ts`**
    - Tests for the YAML parser to ensure correct parsing of YAML files.

#### **`engine/`**

- **`ruleEngine.test.ts`**
    - Tests for the rule engine to verify that rules are applied correctly to datasets.

#### **`utils/`**

- **`expressionEvaluator.test.ts`**
    - Tests for the expression evaluation utility to ensure safe and accurate evaluation of expressions.

## Detailed Explanation

### **1. Segregation of YAML Parsing and Rule Processing**

- **YAML Parsing (`parsers/` Module)**:
    - Responsible solely for reading and parsing YAML files.
    - Converts YAML content into JavaScript objects or TypeScript interfaces that represent rules.
    - This module does not concern itself with how the rules are applied to data.

- **Rule Processing (`engine/` Module)**:
    - Takes the parsed rules and applies them to datasets.
    - Contains the logic for validation, mapping, filtering, and other rule types.
    - Relies on the models defined in the `models/` directory and utilities from the `utils/` directory.

### **2. Modularity and Reusability**

- **`models/` Module**:
    - Centralizes all data structures and types used in the application.
    - Promotes reusability and consistency across modules.

- **`utils/` Module**:
    - Houses shared utility functions, such as expression evaluation.
    - Encourages code reuse and separation of concerns.

### **3. Examples and Testing**

- **`examples/` Directory**:
    - Provides sample data and rules for demonstration purposes.
    - Helps new developers understand how to use the rule engine.

- **`test/` Directory**:
    - Ensures that both the parsing and rule application functionalities work as expected.
    - Organized in a way that mirrors the `src/` directory for consistency.

## Sample Implementation Snippets

### **`parsers/ymlParser.ts`**

```typescript
// parsers/ymlParser.ts
import fs from 'fs';
import yaml from 'js-yaml';
import { Rule } from '../models';

export class YMLParser {
  static parseRuleFile(filePath: string): Rule[] {
    try {
      const fileContents = fs.readFileSync(filePath, 'utf8');
      const data = yaml.load(fileContents) as any;
      // Transform YAML data into Rule objects
      const rules: Rule[] = data.rules.map((ruleData: any) => ({
        name: ruleData.name,
        type: ruleData.type,
        expression: ruleData.expression,
        message: ruleData.message,
        target: ruleData.target,
        mappers: ruleData.mappers,
        // Add other properties as needed
      }));
      return rules;
    } catch (error) {
      console.error('Error parsing YAML file:', error);
      throw error;
    }
  }
}
```

### **`engine/ruleEngine.ts`**

```typescript
// engine/ruleEngine.ts
import { Rule } from '../models';
import { evaluateExpression } from '../utils/expressionEvaluator';

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
          if (!evaluateExpression(rule.expression, row)) {
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
          Object.keys(rule.mappers || {}).forEach((targetField) => {
            const expression = rule.mappers![targetField];
            row[targetField] = evaluateExpression(expression, row);
          });
          return row;
        });
      } else if (rule.type === 'Filter') {
        // Apply filter rule
        processedData = processedData.filter((row) => evaluateExpression(rule.expression, row));
      }
      // Handle other rule types...
    });

    return { processedData, errors };
  }
}
```

### **`utils/expressionEvaluator.ts`**

```typescript
// utils/expressionEvaluator.ts
import jsep from 'jsep';
import { evaluate } from 'jsep-eval';

export function evaluateExpression(expression: string, context: Record<string, any>): any {
  try {
    const parsedExpression = jsep(expression);
    return evaluate(parsedExpression, context);
  } catch (error) {
    console.error('Error evaluating expression:', expression, error);
    throw error;
  }
}
```

### **`models/rule.ts`**

```typescript
// models/rule.ts
export interface Rule {
  name: string;
  type: 'Validation' | 'Mapping' | 'Filter' | 'Grouping' | 'Flatten' | 'UnGrouping';
  expression?: string;
  message?: string;
  target?: string;
  mappers?: { [key: string]: string };
  groupBy?: string | string[];
  flattenBy?: {
    key: string;
    prefix?: string;
  };
  // Add other properties as needed
}
```

### **`main.ts`**

```typescript
// main.ts
import { YMLParser } from './parsers';
import { RuleEngine } from './engine';
import sampleData from './examples/data/sampleData.json';

async function main() {
  // Parse rules from YAML file
  const rules = YMLParser.parseRuleFile('./src/examples/rules/exampleRules.yml');

  // Initialize the rule engine with parsed rules
  const ruleEngine = new RuleEngine(rules);

  // Apply rules to the dataset
  const { processedData, errors } = ruleEngine.applyRules(sampleData);

  // Handle errors and output processed data
  if (errors.length > 0) {
    console.error('Validation Errors:', errors);
  } else {
    console.log('Processed Data:', processedData);
  }
}

main().catch((error) => console.error('Error in processing:', error));
```

## Benefits of This Structure

- **Separation of Concerns**: By segregating YAML parsing and rule processing into independent modules, each module can be developed, tested, and maintained separately.

- **Modularity**: The structure allows for easy addition of new features or rule types without impacting other parts of the application.

- **Scalability**: As the complexity of the rules or the size of the dataset grows, the application can be scaled appropriately.

- **Testability**: With clear module boundaries, unit tests can be written for each module, improving code reliability.

- **Reusability**: Modules like `parsers` and `utils` can be reused in other projects or contexts.

## Setting Up the Project

1. **Initialize the Project**

   ```bash
   mkdir rule-engine
   cd rule-engine
   npm init -y
   ```

2. **Install Dependencies**

   ```bash
   npm install typescript jsep jsep-eval js-yaml
   npm install --save-dev @types/node @types/js-yaml jest ts-jest @types/jest
   ```

3. **Initialize TypeScript Configuration**

   ```bash
   npx tsc --init
   ```

4. **Configure `tsconfig.json`**

   Ensure that the `outDir` is set to `./dist` and `rootDir` is set to `./src`.

5. **Set Up Testing**

   Configure Jest for testing TypeScript code.

6. **Add Scripts to `package.json`**

   ```json
   "scripts": {
     "build": "tsc",
     "start": "node ./dist/main.js",
     "test": "jest"
   }
   ```
