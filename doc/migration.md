# Migration Guide

This document provides guidance for migrating between versions of V-Individual-Model.

## Version History

### Version 0.2.1 (Current)

**Features:**
- Support for seven data types: String, Integer, Decimal, Boolean, Datetime, URI, Binary
- Four serialization formats: JSON, MessagePack, CBOR, Turtle
- Language tag support for internationalization
- Lazy parsing for performance
- Comprehensive API for data manipulation

**Dependencies:**
- `rmp` 0.8
- `serde` 1.0
- `chrono` 0.4.19
- `rust_decimal` 1.36

## Migration from Earlier Versions

### Upgrading from 0.1.x to 0.2.x

#### API Changes

**Individual Creation:**
```rust
// Old way (0.1.x)
let indv = Individual::new();

// New way (0.2.x)
let indv = Individual::default();
```

**Raw Data Handling:**
```rust
// Old way (0.1.x)
let mut indv = Individual::new();
indv.set_raw_data(data);

// New way (0.2.x)
let raw = RawObj::new(data);
let mut indv = Individual::new_raw(raw);
```

**Language Tags:**
```rust
// Old way (0.1.x)
indv.add_string("label", "Hello", "en");

// New way (0.2.x)
use v_individual_model::onto::datatype::Lang;
indv.add_string("label", "Hello", Lang::new_from_str("en"));
```

#### Data Type Changes

**Decimal Handling:**
```rust
// Old way (0.1.x)
indv.add_decimal("price", 12.34);

// New way (0.2.x)
indv.add_decimal_from_str("price", "12.34");
// or
indv.add_decimal_d("price", 1234, -2); // 12.34
```

**URI Operations:**
```rust
// Old way (0.1.x)
indv.add_uri_value("type", "Person");

// New way (0.2.x)
indv.add_uri("rdf:type", "foaf:Person");
```

#### Serialization Changes

**JSON Operations:**
```rust
// Old way (0.1.x)
let json = indv.to_json();

// New way (0.2.x)
let json = indv.get_obj().as_json_str();
```

**MessagePack Operations:**
```rust
// Old way (0.1.x)
let data = indv.to_msgpack();

// New way (0.2.x)
use v_individual_model::onto::individual2msgpack;
let mut output = Vec::new();
individual2msgpack::to_msgpack(&indv, &mut output);
```

### Breaking Changes in 0.2.0

1. **Individual constructor renamed:** `Individual::new()` → `Individual::default()`
2. **Raw data handling:** Now requires `RawObj` wrapper
3. **Language tags:** String-based → `Lang` struct
4. **Decimal storage:** Float-based → mantissa/exponent pairs
5. **URI methods:** `add_uri_value()` → `add_uri()`
6. **JSON serialization:** `to_json()` → `obj.as_json_str()`
7. **MessagePack:** Separate import required

## Migration Examples

### Simple Individual Creation

```rust
// Before (0.1.x)
fn create_person_v1() {
    let mut person = Individual::new();
    person.set_id("person:123");
    person.add_string("name", "John", "");
    person.add_integer("age", 30);
}

// After (0.2.x)
fn create_person_v2() {
    use v_individual_model::onto::datatype::Lang;
    let mut person = Individual::default();
    person.set_id("person:123");
    person.add_string("foaf:name", "John", Lang::none());
    person.add_integer("foaf:age", 30);
}
```

### Data Parsing

```rust
// Before (0.1.x)
fn parse_data_v1(data: Vec<u8>) {
    let mut indv = Individual::new();
    indv.set_raw_data(data);
    indv.parse_all();
}

// After (0.2.x)
fn parse_data_v2(data: Vec<u8>) {
    let raw = RawObj::new(data);
    let mut indv = Individual::new_raw(raw);
    indv.parse_all();
}
```

### JSON Processing

```rust
// Before (0.1.x)
fn json_processing_v1(json_str: &str) -> Result<Individual, String> {
    let mut indv = Individual::new();
    indv.parse_json(json_str)?;
    Ok(indv)
}

// After (0.2.x)
fn json_processing_v2(json_str: &str) -> Result<Individual, Box<dyn std::error::Error>> {
    use v_individual_model::onto::json2individual;

    let json_value: serde_json::Value = serde_json::from_str(json_str)?;
    let mut indv = Individual::default();
    json2individual::parse_json_to_individual(&json_value, &mut indv);
    Ok(indv)
}
```

## Dependency Updates

### Rust Version
- Minimum supported Rust version: 1.70.0
- Recommended Rust version: 1.75.0+

### Cargo.toml Changes

```toml
# Update these dependencies
[dependencies]
v-individual-model = "0.2.1"
rmp = "0.8"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
chrono = "0.4.19"
rust_decimal = "1.36"
```

## Testing Migration

### Unit Tests Update

```rust
// Before (0.1.x)
#[test]
fn test_individual_creation() {
    let indv = Individual::new();
    assert!(!indv.is_exists("name"));
}

// After (0.2.x)
#[test]
fn test_individual_creation() {
    let indv = Individual::default();
    assert!(!indv.is_exists("foaf:name"));
}
```

### Integration Tests

```rust
// Before (0.1.x)
#[test]
fn test_json_roundtrip() {
    let mut indv = Individual::new();
    indv.add_string("name", "Test", "");
    let json = indv.to_json();
    // ... parse and compare
}

// After (0.2.x)
#[test]
fn test_json_roundtrip() {
    use v_individual_model::onto::datatype::Lang;
    let mut indv = Individual::default();
    indv.add_string("foaf:name", "Test", Lang::none());
    let json = indv.get_obj().as_json_str();
    // ... parse and compare
}
```

## Performance Considerations

### Memory Usage
- Version 0.2.x uses more memory for language tags
- Raw data parsing is more efficient
- Consider memory usage in migration

### Performance Improvements
- Lazy parsing reduces initial memory usage
- Binary formats are faster in 0.2.x
- Decimal operations are more precise but slower

## Common Issues and Solutions

### Issue: Compilation Errors with Language Tags

**Problem:**
```rust
// This won't compile in 0.2.x
indv.add_string("label", "Hello", "en");
```

**Solution:**
```rust
use v_individual_model::onto::datatype::Lang;
indv.add_string("label", "Hello", Lang::new_from_str("en"));
```

### Issue: Raw Data Handling

**Problem:**
```rust
// This won't compile in 0.2.x
let mut indv = Individual::new();
indv.set_raw_data(data);
```

**Solution:**
```rust
let raw = RawObj::new(data);
let mut indv = Individual::new_raw(raw);
```

### Issue: Decimal Precision Loss

**Problem:** Decimals are now stored as mantissa/exponent pairs instead of floats.

**Solution:** Use appropriate decimal methods:
```rust
// For precise decimal values
indv.add_decimal_d("price", 12345, -2); // 123.45

// For string-based input
indv.add_decimal_from_str("price", "123.45");
```

### Issue: JSON Serialization Changes

**Problem:** `to_json()` method no longer exists.

**Solution:**
```rust
// Use the new method
let json_str = indv.get_obj().as_json_str();
let json_value = indv.get_obj().as_json();
```

## Future Compatibility

### Upcoming Changes in 0.3.x

**Planned Features:**
- Enhanced error types
- Streaming serialization support
- Improved memory management
- Additional RDF formats support

**Migration Preparation:**
- Start using `Result` types consistently
- Avoid deprecated methods
- Prepare for API changes in error handling

### Long-term API Stability

The library aims to maintain API stability within major versions. Breaking changes will be clearly documented with migration guides.

## Support

For migration assistance:
1. Check the [examples](examples.md) for current API usage
2. Review the [API reference](api.md) for method signatures
3. Run tests to identify breaking changes
4. Update dependencies incrementally

## Version Compatibility Matrix

| Feature | 0.1.x | 0.2.x |
|---------|-------|-------|
| Basic Individual | ✅ | ✅ |
| Language Tags | ❌ | ✅ |
| Lazy Parsing | ❌ | ✅ |
| Decimal Precision | ❌ | ✅ |
| Multiple Formats | ✅ | ✅ |
| Memory Efficiency | ⚠️ | ✅ |
| Type Safety | ⚠️ | ✅ |
