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

**5. `src/outputs/entity_object.rs`** - Dereference `Box<Value>` for `Value::from_json()`:

```rust
value.map(|it| match Value::from_json(*it.clone()) {
    Ok(v) => v,
    Err(_) => Value::from(it.to_string()),
})
```

**6. `src/builder_context/types_map.rs`** - Box the value for `sea_orm::Value::Json`:

```rust
sea_orm::Value::Json(Some(Box::new(value)))
```

Patches 5-6 fix compatibility with sea-orm 2.0.0-rc.32, which changed `Value::Json` to use `Box<serde_json::Value>` instead of plain `serde_json::Value`.

**7. `Cargo.toml`** - Widen `async-graphql` version constraint from `~7.0.17` to `^7.0.17`:

Upstream pins `async-graphql` to `~7.0.17` (`>=7.0.17, <7.1.0`), which conflicts with tradauri's `^7.2` requirement. Changed to `^7.0.17` to allow resolution with newer minor versions.
