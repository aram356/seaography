# Patches

## Seaography

This fork contains patches to handle null GraphQL variable values correctly.

### Why Patch?

Seaography doesn't handle null GraphQL variable values correctly. When a query declares optional variables like:

```graphql
query GetBrokers($filters: BrokersFilterInput, $orderBy: BrokersOrderInput, $pagination: PaginationInput) {
  brokers(filters: $filters, orderBy: $orderBy, pagination: $pagination) { ... }
}
```

And these variables are not provided (passed as `null`), seaography's code incorrectly assumes that `Some(value)` means the value is a valid object and calls `.object()` on it. However, `null` is a valid `Some` value in GraphQL that isn't an object, causing an "internal: not an object" error.

### Patches Applied

**1. `src/query/filtering.rs`** - Added null check in `get_filter_conditions()`:

```rust
if let Some(filters) = filters {
    if filters.is_null() {
        return Ok(Condition::all());
    }
    let filters = filters.object()?;
    // ...
}
```

**2. `src/inputs/order_input.rs`** - Added null check in `parse_object()`:

```rust
Some(value) => {
    if value.is_null() {
        return Ok(Vec::new());
    }
    let order_by = value.object()?;
    // ...
}
```

**3. `src/inputs/pagination_input.rs`** - Added null check in `parse_object()`:

```rust
let binding = value.expect("Checked not null");
if binding.is_null() {
    return Ok(PaginationInput { cursor: None, offset: None, page: None });
}
let object = binding.object()?;
```

**4. `src/query/having.rs`** - Added null check in `get_having_conditions()`:

```rust
if let Some(having) = having {
    if having.is_null() {
        return Ok(condition);
    }
    let having = having.object()?;
    // ...
}
```
