# Fork patches

This fork tracks Seaography 2.0.0-rc.9 and keeps two compatibility changes for downstream users.

## Nullable GraphQL query arguments

Upstream tries to parse an explicit GraphQL `null` as an input object. A query such as
`film(filters: null, having: null, orderBy: null, pagination: null)` then fails with
`internal: not an object`. The fork treats null the same as an omitted optional
argument in four places:

- `src/query/filtering.rs`: return an empty filter condition.
- `src/query/having.rs`: retain the existing condition.
- `src/inputs/order_input.rs`: return no ordering clauses.
- `src/inputs/pagination_input.rs`: return no pagination strategy.

`examples/sqlite/tests/query_tests.rs` checks each nullable argument against the
same query with that argument omitted. The regression test fails on upstream
2.0.0-rc.9 and passes on this fork.

## async-graphql version range

Upstream pins `async-graphql` to `~7.0.17`, which excludes the 7.2 series.
This fork uses `^7.0.17` so Cargo can resolve one GraphQL version across
Seaography and downstream crates. This fork compiles with async-graphql 7.2.1
and SeaORM 2.0.4.

The earlier boxed JSON value compatibility changes are present in upstream
2.0.0-rc.9 and no longer need separate fork patches.
