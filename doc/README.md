# V-Individual-Model Documentation

This documentation provides comprehensive information about the V-Individual-Model library, a Rust crate for managing semantic data structures within the Veda platform.

## Overview

V-Individual-Model is a Rust library that implements an Individual data model for semantic data management. It provides:

- **Individual data structure** for storing semantic data with URI identifiers and predicates
- **Multiple serialization formats**: JSON, MessagePack, CBOR, and Turtle
- **Rich data types**: strings, integers, decimals, dates, booleans, and binary data
- **Language support** for internationalization
- **Efficient parsing and conversion** between different formats

## Quick Start

```rust
use v_individual_model::onto::individual::Individual;
use v_individual_model::onto::datatype::Lang;

// Create a new individual
let mut indv = Individual::default();

// Set identifier
indv.set_id("example:person:123");

// Add some data
indv.add_string("rdfs:label", "John Doe", Lang::none());
indv.add_integer("foaf:age", 30);
indv.add_bool("schema:active", true);

// Convert to JSON
let json_str = indv.get_obj().as_json_str();
```

## Documentation Sections

- [Architecture](./architecture.md) - System architecture and design decisions
- [API Reference](./api.md) - Complete API documentation
- [Data Types](./datatypes.md) - Supported data types and their usage
- [Serialization](./serialization.md) - Working with different formats
- [Examples](./examples.md) - Code examples and use cases
- [Migration](./migration.md) - Migration guides and breaking changes

## Building and Testing

```bash
# Build the library
cargo build

# Run tests
cargo test

# Generate documentation
cargo doc --open
```

## Dependencies

The library uses several key dependencies:
- `rmp` - MessagePack serialization
- `serde` - Serialization framework with JSON support
- `serde_json` - JSON serialization
- `chrono` - Date/time handling
- `rust_decimal` - Decimal number support
- `rio_turtle` - Turtle format support
- `rio_api` - RDF API support
- `derivative` - Custom derive macros
- `num` - Numeric traits
- `num-traits` - Additional numeric traits
- `v-cbr-codec` - CBOR codec
- `iri-string` - IRI string handling
- `bincode` - Binary serialization
- `log` - Logging framework
- `base64` - Base64 encoding/decoding

## Contributing

See the main project repository for contribution guidelines and development setup.
