# Rule Examples for Similar Libraries

Below are examples of how rules are defined and used in each of the libraries mentioned earlier. These examples illustrate the syntax and capabilities of each rule engine, helping you understand how they might fit into your projects.

---

## 1. **json-rules-engine**

**Description**: A powerful, lightweight rules engine for Node.js and the browser. It allows you to define complex business rules in JSON format and evaluate them at runtime.

**Rule Example**:

```javascript
const { Engine } = require('json-rules-engine');

// Define a rule
let rule = {
  conditions: {
    all: [{
      fact: 'temperature',
      operator: 'greaterThan',
      value: 100
    }]
  },
  event: {
    type: 'high-temperature',
    params: {
      message: 'Temperature exceeds 100 degrees'
    }
  }
};

// Initialize the engine
let engine = new Engine();
engine.addRule(rule);

// Define facts
let facts = { temperature: 105 };

// Run the engine
engine
  .run(facts)
  .then(events => {
    events.map(event => console.log(event.params.message));
  });
```

**Explanation**:

- **Conditions**: Checks if the `temperature` fact is greater than 100.
- **Event**: Triggers an event with a message if the condition is met.
- **Usage**: Add rules to the engine and run it with facts; the engine evaluates the rules and fires events accordingly.

---

## 2. **Nools**

**Description**: A JavaScript rules engine based on the Rete algorithm, allowing you to define rules using a domain-specific language (DSL) or JSON.

**Rule Example (DSL)**:

```javascript
const nools = require('nools');

const flow = nools.flow('Discount Flow', function(flow) {
  flow.rule('Apply Discount', [
    ['and', 
      ['Product', 'p', 'p.price > 100'],
      ['Customer', 'c', 'c.loyaltyLevel === "GOLD"']
    ]
  ], function(facts) {
    facts.p.discount = 0.1;
    this.modify(facts.p);
  });
});

// Define facts
let Product = flow.getDefined('Product');
let Customer = flow.getDefined('Customer');

let session = flow.getSession(
  new Product({ name: 'Laptop', price: 150 }),
  new Customer({ name: 'Alice', loyaltyLevel: 'GOLD' })
);

session.match().then(() => {
  // Access modified facts here
  console.log('Discount applied:', session.getFacts()[0].discount); // Output: 0.1
});
```

**Explanation**:

- **Rule**: Applies a 10% discount if the product price is over 100 and the customer is a GOLD member.
- **Flow**: Represents a collection of rules.
- **Session**: Holds the facts and evaluates them against the rules.

---

## 3. **JSON Logic**

**Description**: A language-independent way to describe conditional logic using JSON.

**Rule Example**:

```json
{
  "if": [
    { ">": [{ "var": "temperature" }, 100] },
    "Temperature is too high!",
    "Temperature is normal."
  ]
}
```

**Usage in JavaScript**:

```javascript
const jsonLogic = require('json-logic-js');

let rule = {
  "if": [
    { ">": [{ "var": "temperature" }, 100] },
    "Temperature is too high!",
    "Temperature is normal."
  ]
};

let data = { "temperature": 105 };

let result = jsonLogic.apply(rule, data);
console.log(result); // Output: "Temperature is too high!"
```

**Explanation**:

- **Rule**: Uses JSON structures to define logic.
- **Variables**: Accessed using `"var"`.
- **Operators**: Logical operators like `>`, `==`, `and`, `or` are used within the JSON.

---

## 4. **RuleJS**

**Description**: A JavaScript rule engine that allows you to define business rules in JSON format.

**Rule Example**:

```javascript
const RuleEngine = require('node-rules');

// Define rules
let rules = [{
  condition: function(R) {
    R.when(this.transactionTotal < 500);
  },
  consequence: function(R) {
    this.result = false;
    this.reason = 'Transaction total less than 500';
    R.stop();
  }
}];

// Initialize the engine
let R = new RuleEngine(rules);

// Define a fact
let fact = { transactionTotal: 400 };

// Execute the rules
R.execute(fact, function(result) {
  if (result.result) {
    console.log('Transaction approved');
  } else {
    console.log('Transaction denied:', result.reason);
  }
});
```

**Explanation**:

- **Condition**: Function that determines if the rule should fire.
- **Consequence**: Function that executes if the condition is met.
- **Flow**: The engine processes the rules and facts, executing consequences as needed.

---

## 5. **Jexl (JavaScript Expression Language)**

**Description**: A powerful expression language for JavaScript, which can be used to evaluate expressions in your applications.

**Expression Example**:

```javascript
const jexl = require('jexl');

// Define an expression
let expression = 'quantity > 0 && price >= 10';

// Define context data
let context = { quantity: 5, price: 15 };

// Evaluate the expression
jexl.eval(expression, context).then(result => {
  console.log(result); // Output: true
});
```

**Explanation**:

- **Expression**: A string containing the logic to evaluate.
- **Context**: Data against which the expression is evaluated.
- **Usage**: Useful for building custom rule engines or dynamic evaluations.

---

## 6. **Rools**

**Description**: A lightweight rule engine for Node.js with a simple and intuitive API.

**Rule Example**:

```javascript
const { Rools, Rule } = require('rools');

// Define a rule
const rule = new Rule({
  name: 'High Value Order',
  when: facts => facts.order.amount > 1000,
  then: facts => {
    facts.order.isHighValue = true;
  },
});

// Initialize the engine
const rools = new Rools();
await rools.register([rule]);

// Define facts
const facts = {
  order: { amount: 1500 }
};

// Evaluate the rules
await rools.evaluate(facts);

console.log(facts.order.isHighValue); // Output: true
```

**Explanation**:

- **Rule Definition**: Uses `when` for conditions and `then` for actions.
- **Facts**: Data passed into the engine for evaluation.
- **Evaluation**: The engine processes the facts against the registered rules.

---

## 7. **Easy Rules (Java-based)**

**Description**: A simple yet powerful Java rules engine. Rules can be defined in YAML, MVEL, or Java code.

**Rule Example (YAML)**:

```yaml
name: "Weather Rule"
description: "If it rains, take an umbrella"
condition: "rain == true"
actions:
  - "System.out.println(\"It rains, take an umbrella!\");"
```

**Usage in Java**:

```java
import org.jeasy.rules.api.*;
import org.jeasy.rules.mvel.MVELRuleFactory;

RuleFactory ruleFactory = new MVELRuleFactory();
Rule weatherRule = ruleFactory.createRule(new FileReader("weather-rule.yml"));

Rules rules = new Rules();
rules.register(weatherRule);

Facts facts = new Facts();
facts.put("rain", true);

RulesEngine rulesEngine = new DefaultRulesEngine();
rulesEngine.fire(rules, facts);
```

**Explanation**:

- **Rule File**: Defined in YAML format with condition and actions.
- **Rule Engine**: Loads the rule and fires it based on facts.

---

## 8. **Camunda BPM Platform**

**Description**: An open-source platform for workflow and business process management using BPMN and DMN.

**Rule Example (DMN Table)**:

In Camunda Modeler, you can create a DMN decision table:

| **Order Amount** | **Customer Type** | **Discount** |
|------------------|-------------------|--------------|
| > 1000           | Premium           | 15%          |
| <= 1000          | Regular           | 5%           |

**Usage**:

- **Decision Table**: Defines rules for calculating discounts.
- **Integration**: The DMN decision can be called from BPMN processes or via REST API.

**Explanation**:

- **Input**: Order amount and customer type.
- **Output**: Discount percentage.
- **Execution**: Camunda evaluates the table based on inputs provided.

---

## 9. **OpenRules**

**Description**: A BRMS that allows defining rules in Excel spreadsheets, which are then converted into executable rules.

**Rule Example (Excel)**:

In an Excel spreadsheet, you might have a decision table like:

| **Rule ID** | **Condition**                                              | **Action**                           |
|-------------|------------------------------------------------------------|--------------------------------------|
| Rule1       | If LoanAmount > 100000 and CreditScore > 700               | Approve loan                         |
| Rule2       | If LoanAmount <= 100000 or CreditScore <= 700              | Reject loan                          |

**Usage**:

- **Decision Table**: Defines rules in tabular format.
- **Execution**: OpenRules engine reads the Excel file and processes the rules.

**Explanation**:

- **Condition**: Logical expressions in the condition column.
- **Action**: What to do when conditions are met.

---

## 10. **Node-RED**

**Description**: A flow-based programming tool for wiring together hardware devices, APIs, and online services.

**Rule Example**:

In Node-RED, you create flows using nodes. For example:

1. **Inject Node**: Triggers the flow manually or at intervals.
2. **Function Node**: Contains JavaScript code for rule logic.
   ```javascript
   // Function node code
   if (msg.payload.temperature > 100) {
     msg.payload.alert = "High temperature detected!";
   }
   return msg;
   ```
3. **Debug Node**: Displays the output in the debug panel.

**Explanation**:

- **Flow**: Visual representation of data processing.
- **Function Node**: Where you implement your custom rule logic.

---

## 11. **json-rules-engine-simplified**

**Description**: A simplified version of `json-rules-engine` focusing on ease of use.

**Rule Example**:

```javascript
const Engine = require('json-rules-engine-simplified');

// Define rules
const rules = [
  {
    conditions: {
      age: { greaterThan: 18 }
    },
    event: {
      type: 'adult-check',
      params: { message: 'User is an adult.' }
    }
  }
];

// Initialize the engine
const engine = new Engine(rules);

// Define facts
const facts = { age: 20 };

// Run the engine
engine.run(facts).then(events => {
  events.forEach(event => {
    console.log(event.params.message); // Output: "User is an adult."
  });
});
```

**Explanation**:

- **Conditions**: Simple JSON conditions without nesting.
- **Event**: Contains the action to perform if conditions are met.
- **Execution**: The engine evaluates the facts against the rules.

---

## 12. **Custom Rule Engine Using JSEP and jsep-eval**

**Description**: Building a custom rule engine using expression parsers like JSEP and evaluators like `jsep-eval`.

**Rule Example**:

```javascript
const jsep = require('jsep');
const { evaluate } = require('jsep-eval');

// Define a rule
const rule = {
  name: 'Discount Eligibility',
  expression: 'purchaseAmount >= 1000',
  action: (data) => {
    data.isEligibleForDiscount = true;
  }
};

// Define data
let data = { purchaseAmount: 1500 };

// Evaluate the expression
const parsedExpression = jsep(rule.expression);
const result = evaluate(parsedExpression, data);

if (result) {
  rule.action(data);
}

console.log(data.isEligibleForDiscount); // Output: true
```

**Explanation**:

- **Expression Parsing**: Uses JSEP to parse the expression string.
- **Evaluation**: `jsep-eval` evaluates the parsed expression against the data.
- **Action**: Executes a function if the expression evaluates to true.

---

# Conclusion

These examples showcase how different rule engines allow you to define and execute rules using various formats, such as JSON, YAML, DSLs, or even Excel spreadsheets. Each engine has its own strengths and use cases:

- **json-rules-engine** and **json-rules-engine-simplified**: Great for JSON-based rule definitions with event handling.
- **Nools**: Suitable for complex pattern matching and forward-chaining rules.
- **JSON Logic**: Ideal for language-independent rule definitions.
- **RuleJS** and **Rools**: Provide simple APIs for defining and executing rules in JavaScript.
- **Jexl**: Useful when you need to evaluate expressions dynamically.
- **Easy Rules** and **OpenRules**: Offer options for Java applications with rule definitions in YAML or Excel.
- **Camunda BPM Platform**: Best for integrating business rules within workflows.
- **Node-RED**: Excellent for visual programming and quick integrations.
- **Custom Engine with JSEP**: Gives you full control over rule parsing and execution.

**Next Steps**:

- **Choose the Right Engine**: Based on your project's requirements, select the engine that best fits your needs.
- **Experiment with Examples**: Try out these examples to get a feel for how each engine works.
- **Extend and Customize**: Many of these engines can be extended or customized to better suit your specific use case.

---

If you have any questions about these examples or need further assistance in implementing a rule engine for your project, feel free to ask!