# Serialization Formats

This document describes the serialization formats supported by V-Individual-Model and how to convert between them.

## Supported Formats

The library supports four serialization formats:

1. **JSON** - Human-readable, web-friendly
2. **MessagePack** - Efficient binary format
3. **CBOR** - Schema-aware binary format
4. **Turtle** - RDF-compatible text format

## JSON Format

JSON is the most human-readable format, ideal for APIs and debugging.

### Structure
```json
{
  "@": "individual-uri",
  "predicate1": ["value1", "value2"],
  "predicate2": [{"data": "value", "lang": "en", "type": "String"}]
}
```

Where:
- `"@"` - Individual URI identifier
- Array values for multiple resources
- Object values for complex data (strings with language, decimals)

### JSON Operations

```rust
use v_individual_model::onto::individual::Individual;
use v_individual_model::onto::json2individual;
use serde_json::Value;

// Create individual from JSON
let json_data: Value = serde_json::from_str(r#"
{
  "@": "example:person:123",
  "rdfs:label": [
    {"data": "John Doe", "lang": "", "type": "String"},
    {"data": "Джон Доу", "lang": "ru", "type": "String"}
  ],
  "foaf:age": [{"data": 30, "type": "Integer"}]
}
"#).unwrap();

let mut indv = Individual::default();
json2individual::parse_json_to_individual(&json_data, &mut indv);

// Convert individual to JSON
let json_str = indv.get_obj().as_json_str();
let json_value = indv.get_obj().as_json();
```

## MessagePack Format

MessagePack is a binary format that's more compact and faster than JSON.

### Structure
MessagePack uses a binary array format:
```
[uri_string, {predicate1: [resources], predicate2: [resources], ...}]
```

### MessagePack Operations

```rust
use v_individual_model::onto::individual::{Individual, RawObj};
use v_individual_model::onto::msgpack2individual;
use v_individual_model::onto::individual2msgpack;
use v_individual_model::onto::datatype::Lang;

// Convert individual to MessagePack
let mut indv = Individual::default();
indv.set_id("example:person:123");
indv.add_string("rdfs:label", "John Doe", Lang::none());

let mut out = Vec::new();
individual2msgpack::to_msgpack(&indv, &mut out).ok();

// Parse MessagePack data
let mut raw_obj = RawObj::new(out);
let uri = msgpack2individual::parse_msgpack(&mut raw_obj).unwrap();

// Parse specific predicate from raw data
let mut indv2 = Individual::new_raw(raw_obj);
msgpack2individual::parse_msgpack_to_predicate("rdfs:label", &mut indv2).unwrap();
```

## CBOR Format

CBOR (Concise Binary Object Representation) is a binary format with schema awareness.

### CBOR Operations

```rust
use v_individual_model::onto::cbor2individual;
use v_individual_model::onto::individual::RawObj;

// Parse CBOR data
let mut raw_obj = RawObj::new(cbor_data);
let uri = cbor2individual::parse_cbor(&mut raw_obj).unwrap();

// Parse specific predicate
let mut indv = Individual::new_raw(raw_obj);
cbor2individual::parse_cbor_to_predicate("predicate", &mut indv).unwrap();
```

## Turtle Format

Turtle is a text format for RDF data, useful for semantic web applications.

### Turtle Operations

```rust
use v_individual_model::onto::individual2turtle;
use std::collections::HashMap;

// Convert to Turtle
let mut prefixes = HashMap::new();
prefixes.insert("rdf".to_string(), "http://www.w3.org/1999/02/22-rdf-syntax-ns#".to_string());
prefixes.insert("foaf".to_string(), "http://xmlns.com/foaf/0.1/".to_string());

let individuals = vec![&indv];
let turtle_data = individual2turtle::to_turtle_with_counter_refs(&individuals, &prefixes);
```

## Format Conversion

### Converting Between Formats

```rust
// JSON -> Individual -> MessagePack
let json_value: serde_json::Value = /* ... */;
let mut indv = Individual::default();
json2individual::parse_json_to_individual(&json_value, &mut indv);

let mut msgpack_out = Vec::new();
individual2msgpack::to_msgpack(&indv, &mut msgpack_out).ok();

// MessagePack -> Individual -> JSON
let mut raw_obj = RawObj::new(msgpack_out);
let uri = msgpack2individual::parse_msgpack(&mut raw_obj).unwrap();
let mut indv2 = Individual::new_raw(raw_obj);
indv2.parse_all();

let json_output = indv2.get_obj().as_json_str();
```

## Performance Comparison

| Format | Size | Speed | Human-Readable | Use Case |
|--------|------|-------|----------------|----------|
| JSON | Large | Medium | Yes | APIs, debugging |
| MessagePack | Small | Fast | No | Storage, network |
| CBOR | Small | Fast | No | IoT, constrained devices |
| Turtle | Medium | Slow | Yes | Semantic web, RDF |

## Raw Data Handling

### Lazy Parsing

The library supports lazy parsing for efficiency:

```rust
// Raw data is stored but not parsed
let mut indv = Individual::new_raw(raw_obj);

// Parsing happens only when data is accessed
let name = indv.get_first_literal("rdfs:label"); // Triggers parsing for this predicate

// Force complete parsing
indv.parse_all();
```

### Partial Parsing

Parse only specific predicates when you know what you need:

```rust
// Parse only specific predicate
msgpack2individual::parse_msgpack_to_predicate("foaf:name", &mut indv).unwrap();

// Access the parsed data
let name = indv.get_first_literal("foaf:name");
```

## Error Handling

All parsing operations return `Result` types:

```rust
// Handle parsing errors
match msgpack2individual::parse_msgpack(&mut raw_obj) {
    Ok(uri) => println!("Parsed individual: {}", uri),
    Err(code) => eprintln!("Parse error: {}", code),
}

// Handle predicate parsing errors
match msgpack2individual::parse_msgpack_to_predicate("predicate", &mut indv) {
    Ok(()) => println!("Predicate parsed successfully"),
    Err(msg) => eprintln!("Predicate parse error: {}", msg),
}
```

## Best Practices

### Format Selection
- **JSON** for human-readable data, APIs, and debugging
- **MessagePack** for efficient storage and network communication
- **CBOR** for IoT applications and constrained environments
- **Turtle** for RDF data and semantic web applications

### Performance Optimization
- Use lazy parsing when accessing individual values
- Parse all data at once for bulk operations
- Choose binary formats for storage and network transfer
- Use JSON for development and debugging

### Data Integrity
- Validate data before serialization
- Handle parsing errors gracefully
- Use appropriate formats for your use case
- Consider compression for large binary data

### Memory Management
- Raw data remains in memory until parsed
- Parsed data uses more memory but faster access
- Consider streaming for very large datasets
- Clean up unused individuals to free memory
