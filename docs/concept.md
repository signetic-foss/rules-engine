# Rules Engine Library - Concept Document

## Overview

The **Rules Engine Library** is a utility designed to parse **YAML-based rules** and apply them dynamically to datasets. It provides a declarative way to express business logic, data transformations, validations, and filters by leveraging a flexible configuration syntax. This rules engine is particularly suited for use cases where data needs to be processed, validated, or transformed according to changing business requirements, without modifying core application logic.

The library allows developers and non-technical users to create rules, transformations, and mappings in a human-readable format, making it easy to maintain, scale, and modify as requirements evolve.

## Key Features

1. **YAML-Based Rule Definition**:
    - Rules are defined in **YAML** format, which is human-readable and easy to manage.
    - Users can define rules for filtering, validation, mapping, grouping, and more, allowing great flexibility in processing datasets.

2. **Flexible Rule Types**:
    - The rules engine supports multiple types of rules, including:
        - **Filtering**: Removing unwanted data based on specified conditions.
        - **Validation**: Ensuring data integrity by validating conditions.
        - **Mapping**: Modifying or transforming data fields.
        - **Flattening**: Flattening nested structures for easier data manipulation.
        - **Grouping**: Grouping datasets by specific keys.

3. **Modular Conversion Pipeline**:
    - **Conversions** are specified as sequences that apply multiple rules in a defined order. This modular approach provides a clear, step-by-step process for transforming datasets.

4. **Crosswalk Handling**:
    - The engine can handle **crosswalks**, which are useful for matching and transforming data from multiple sources based on key relationships.

5. **JavaScript Integration**:
    - The rules engine can evaluate **JavaScript** expressions, enabling dynamic and powerful transformations and validations.
    - This allows for highly customizable data processing, giving users the ability to embed complex logic within rules.

## Use Cases

1. **Data Validation for ETL Processes**:
    - Use the rules engine to validate incoming datasets during ETL (Extract, Transform, Load) processes, ensuring that only correct data is processed further.

2. **Data Transformation and Enrichment**:
    - Apply mapping rules to transform data formats or to enrich datasets by adding derived fields, helping to prepare data for analytics or reporting.

3. **Business Logic Separation**:
    - By maintaining business rules outside of application code, non-developers can update and manage these rules, reducing dependency on engineering teams for simple rule changes.

## Architectural Overview

### Rule Parsing and Application Flow
- **Rule Definition**: The rules are defined in YAML files, specifying the type of operation, the target dataset, and any additional parameters needed for processing.
- **Rule Types Supported**:
    - **Filter**: Filter data based on conditions (`expression`).
    - **Validation**: Validate records and either accept or reject based on specified conditions.
    - **Mapping**: Transform data fields, including modifying values and performing computations.
    - **Flattening**: Simplify nested data structures, making data easier to process.
    - **Grouping**: Aggregate data based on specified key(s).
- **Conversion Flow**: Conversions are a sequence of rules to be applied in order, ensuring a consistent data transformation pipeline.

### Rule Configuration Examples

The **Rules Engine Library** supports a variety of rules to provide different functionalities. Below are some examples:

#### 1. Filtering Invalid Items (Example: "fairy")
```yaml
name: fairy
rules:
  - name: FilterInvalidItems
    expression: quantity > 0
    type: Filter
    language: javascript
    message: Quantity cannot be 0
    action: reject
    keys:
      - quantity

conversions:
  - FilterInvalidItems
```
- **Type**: **Filter**
- **Expression**: Uses JavaScript to filter out records where `quantity` is less than or equal to 0.
- **Action**: `reject` the record if the condition fails.
- This is useful for ensuring only valid items with a positive quantity are processed.

#### 2. Flatten Nested Structures (Example: "fortune")
```yaml
name: fortune
rules:
  - name: FlattenFunFacts
    flattenBy:
      key: funFacts
    type: Flatten
  - name: FlattenItemCatalog
    flattenBy:
      key: itemCatalog
      prefix: 'ic_'
    type: Flatten
  - name: SelectiveFields
    type: Mapping
    mappers:
      funFacts: ""
      itemCatalog: ""

crosswalks:
  - name: itemCatalog
    match:
      main:
        - item_id
      crosswalk:
        - id
  - name: funFacts

conversions:
  - FlattenFunFacts
  - FlattenItemCatalog
  - SelectiveFields
```
- **Flattening**: Flattens nested fields (`funFacts`, `itemCatalog`) into the main dataset. Useful for simplifying the data structure.
- **Mapping**: Allows mapping and selecting specific fields from the flattened dataset.
- **Crosswalks**: Facilitates cross-referencing between entities for consistency.

#### 3. Grouping Data (Example: "griffin")
```yaml
name: griffin
rules:
  - name: GroupingRule
    type: Grouping
    groupBy: item_id
conversions:
  - GroupingRule
```
- **Type**: **Grouping**
- **Group By**: Aggregates data based on `item_id`. This is helpful for summarizing or aggregating data based on unique identifiers.

#### 4. Data Mapping with Calculations (Example: "mystic")
```yaml
name: mystic
rules:
  - name: MapQuantitySquared
    type: Mapping
    language: javascript
    message: Quantity cannot be 0
    action: reject
    mappers:
      quantity: quantity*quantity
      trueField: fun.formulaJS.TRUE()
    keys:
      - quantity

conversions:
  - MapQuantitySquared
```
- **Type**: **Mapping**
- **Mapping Operation**: Maps `quantity` to `quantity * quantity`, effectively calculating its square.
- **JavaScript Integration**: Allows the use of JavaScript for computation, enabling complex calculations.

#### 5. Validation Rule (Example: "vial")
```yaml
name: vial
rules:
  - name: ValidateQuantity
    expression: quantity > 0
    type: Validation
    language: javascript
    message: "'Quantity cannot be 0'"
    action: reject
    keys:
      - quantity

conversions:
  - ValidateQuantity
```
- **Type**: **Validation**
- **Expression**: Ensures that `quantity` is greater than 0.
- **Action**: Rejects invalid data to ensure only accurate records are kept.

## Future Enhancements

1. **Extensible Rule Types**:
    - Add support for more advanced rule types, such as **Sorting**, **Aggregation**, and **Conditional Branching**.

2. **Graphical Rule Builder**:
    - Develop a web-based interface for non-technical users to define rules through a **drag-and-drop UI**, improving accessibility.

3. **Real-Time Execution**:
    - Extend the rules engine to allow real-time execution of rules, enabling immediate data validation and transformation for streaming data.

4. **Rule Versioning and History**:
    - Implement rule versioning, allowing for easy tracking and rollback of rule changes to support audit requirements.

## Contribution Guidelines

We welcome contributions to enhance the **Rules Engine Library**. Contributors can help by:
- Adding support for new rule types or expanding existing functionality.
- Improving parsing capabilities for additional data types.
- Writing tests to ensure the rules engine handles edge cases efficiently.
- Developing documentation and sample YAML configurations to help onboard new users.

Please refer to the `CONTRIBUTING.md` file in the repository for further instructions on how to contribute.

## Conclusion

The **Rules Engine Library** provides a robust and flexible approach to defining and applying business rules using a YAML-based format. It aims to decouple business logic from core application code, offering a more adaptable and maintainable solution for data transformation, validation, and management. By allowing dynamic JavaScript evaluations and providing a rich set of configurable rule types, the library empowers users to handle complex data processing needs with ease.

