**YAML Examples for Data Transformation Workflows**

**1. Basic Mapping Example**

*Description*: This example demonstrates a basic mapping operation where the `quantity` field is transformed into a new field called `quantity_squared` by squaring its value.
```yaml
name: map_basic
source: dataset1.csv
rules:
  - name: MapQuantity
    type: Mapping
    mappers:
      quantity_squared: quantity * quantity
```

**2. Validation Example**

*Description*: This example shows a validation rule applied to ensure that the `quantity` field is greater than zero. If the condition is not met, an error message is generated.
```yaml
name: validate_quantity
source: dataset1.csv
rules:
  - name: ValidateQuantity
    type: Validation
    expression: quantity > 0
    message: Quantity must be greater than 0
```

**3. Filtering Example**

*Description*: This example applies a filter to select only those records where the `category` is 'Electronics'. All other records are excluded.
```yaml
name: filter_category
source: dataset1.csv
rules:
  - name: FilterCategory
    type: Filter
    expression: category == 'Electronics'
```

**4. Grouping Example**

*Description*: This example groups the dataset by `item_id`, allowing for subsequent aggregate calculations or operations to be performed on grouped data.
```yaml
name: group_items
source: dataset1.csv
rules:
  - name: GroupByItem
    type: Grouping
    groupBy: item_id
```

**5. Flattening Example**

*Description*: This example flattens nested data under the `itemCatalog` field, simplifying complex nested structures into a flat format.
```yaml
name: flatten_catalog
source: dataset1.csv
rules:
  - name: FlattenCatalog
    type: Flatten
    flattenBy:
      key: itemCatalog
```

**6. Combined Mapping and Validation**

*Description*: This example combines mapping and validation operations. First, the `quantity` is squared, and then a validation rule checks if the `quantity` is greater than zero.
```yaml
name: combined_mapping_validation
source: dataset1.csv
rules:
  - name: MapQuantitySquared
    type: Mapping
    mappers:
      quantity_squared: quantity * quantity
  - name: ValidateQuantity
    type: Validation
    expression: quantity > 0
    message: Quantity must be greater than 0
```

**7. Interdependent Stages**

*Description*: This example demonstrates two interdependent stages. The first stage filters out items with a `quantity` greater than 10, while the second stage maps a discounted price for the filtered items.
```yaml
---
name: initial_stage
source: dataset_initial.csv
rules:
  - name: FilterHighQuantity
    type: Filter
    expression: quantity > 10
---
name: intermediate_stage
source: initial_stage
rules:
  - name: MapDiscount
    type: Mapping
    mappers:
      discounted_price: price * 0.9
```

**8. Automatic Join of Multiple Stages**

*Description*: This example shows how multiple stages (`unicorn_stage`, `sorcery_stage`, and `griffin_stage`) are automatically joined based on common fields to create a unified dataset.
```yaml
name: join_multiple_stages
joins:
  - unicorn_stage
  - sorcery_stage
  - griffin_stage
```

**9. Filtering and Mapping Combined**

*Description*: This example first filters records where `quantity` is greater than zero and then maps a new field called `adjusted_quantity` by multiplying `quantity` by 1.2.
```yaml
name: filter_and_map
source: dataset1.csv
rules:
  - name: FilterValidItems
    type: Filter
    expression: quantity > 0
  - name: MapAdjustedQuantity
    type: Mapping
    mappers:
      adjusted_quantity: quantity * 1.2
```

**10. Grouping and Ungrouping Example**

*Description*: This example first groups items by `item_id` and then performs an ungrouping operation, effectively reversing the grouping to bring data back to its original granularity.
```yaml
name: group_ungroup_example
source: dataset1.csv
rules:
  - name: GroupByItem
    type: Grouping
    groupBy: item_id
  - name: UngroupItems
    type: UnGrouping
```

**11. Chained Processing with Multiple Stages**

*Description*: This example has two stages. The first stage filters out values less than or equal to 5, and the second stage squares the remaining values, resulting in a new `value_squared` field.
```yaml
---
name: stage_one
source: dataset_initial.csv
rules:
  - name: FilterOutLowValues
    type: Filter
    expression: value > 5
---
name: stage_two
source: stage_one
rules:
  - name: MapSquareValues
    type: Mapping
    mappers:
      value_squared: value * value
```

**12. Data Enrichment Using Crosswalks**

*Description*: This example uses a crosswalk to enrich the dataset by joining additional information from an `itemCatalog`, allowing for the mapping of a new field called `catalog_name`.
```yaml
name: enrich_with_crosswalk
source: dataset1.csv
crosswalks:
  - name: itemCatalog
    match:
      main:
        - item_id
      crosswalk:
        - id
rules:
  - name: MapCatalogDetails
    type: Mapping
    mappers:
      catalog_name: itemCatalog.name
```

**13. Validation with Custom Message**

*Description*: This example validates that the `price` field is greater than zero. If the condition fails, it generates a custom error message indicating that the price must be greater than zero.
```yaml
name: validate_price
source: dataset1.csv
rules:
  - name: ValidatePrice
    type: Validation
    expression: price > 0
    message: Price must be greater than 0
```

**14. Filtering by Multiple Conditions**

*Description*: This example filters records based on multiple conditions: the `quantity` must be greater than zero, and the `category` must be 'Electronics'.
```yaml
name: filter_multiple_conditions
source: dataset1.csv
rules:
  - name: FilterByConditions
    type: Filter
    expression: quantity > 0 and category == 'Electronics'
```

**15. Mapping with Complex Calculations**

*Description*: This example performs a mapping operation with a complex calculation. It calculates a discounted price if the `category` is 'Electronics'; otherwise, it keeps the original price.
```yaml
name: complex_mapping
source: dataset1.csv
rules:
  - name: CalculateDiscountedPrice
    type: Mapping
    mappers:
      discounted_price: price * 0.85 if category == 'Electronics' else price
```