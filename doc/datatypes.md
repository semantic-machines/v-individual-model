# Data Types

This document describes the data types supported by the V-Individual-Model library and how to work with them.

## Overview

The library supports seven core data types defined in the `DataType` enum:

```rust
pub enum DataType {
    String,     // Text data with optional language tags
    Integer,    // 64-bit signed integers
    Decimal,    // Arbitrary precision decimal numbers
    Boolean,    // True/false values
    Datetime,   // Unix timestamps (i64)
    Uri,        // URI strings
    Binary,     // Raw binary data
}
```

## String Type

Strings are the most flexible data type, supporting internationalization through language tags.

### Adding String Values

```rust
use v_individual_model::onto::individual::Individual;
use v_individual_model::onto::datatype::Lang;

let mut indv = Individual::default();

// Add string without language
indv.add_string("rdfs:label", "John Doe", Lang::none());

// Add string with language
indv.add_string("rdfs:label", "Джон Доу", Lang::new_from_str("ru"));

// Set (replace) string value
indv.set_string("foaf:name", "Jane Smith", Lang::new_from_str("en"));
```

### Retrieving String Values

```rust
// Get first literal value (any language)
let name = indv.get_first_literal("rdfs:label");

// Get literal with specific language preference
let langs = vec![Lang::new_from_str("en"), Lang::new_from_str("ru")];
let localized_name = indv.get_first_literal_with_lang("rdfs:label", &langs);

// Get all string values for a predicate
if let Some(literals) = indv.get_literals("rdfs:label") {
    for literal in literals {
        println!("Label: {}", literal);
    }
}
```

### Language Support

Language tags follow IETF BCP 47 standards:
- `Lang::none()` - No language specified
- `Lang::new_from_str("en")` - English
- `Lang::new_from_str("ru")` - Russian
- `Lang::new_from_str("zh-CN")` - Chinese (Simplified)

## Integer Type

64-bit signed integers for whole numbers.

```rust
// Add integer value
indv.add_integer("foaf:age", 30);
indv.add_integer("schema:quantity", -5);

// Set integer value
indv.set_integer("v-s:count", 100);

// Retrieve integer
if let Some(age) = indv.get_first_integer("foaf:age") {
    println!("Age: {}", age);
}
```

**Range**: -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807

## Decimal Type

Arbitrary precision decimal numbers stored as mantissa and exponent pairs.

```rust
// Add decimal using mantissa and exponent
indv.add_decimal_d("schema:price", 12345, -2);  // 123.45
indv.add_decimal_d("schema:weight", 1500, -3);  // 1.500

// Add decimal from string
indv.add_decimal_from_str("schema:rate", "12.34");

// Add decimal from other numeric types
indv.add_decimal_from_i64("schema:count", 42);
indv.add_decimal_from_f64("schema:ratio", 0.85);

// Retrieve decimal as (mantissa, exponent) pair
if let Some((mantissa, exponent)) = indv.get_first_number("schema:price") {
    let value = mantissa as f64 * 10f64.powi(exponent as i32);
    println!("Price: {}", value);
}

// Retrieve as float
if let Some(price) = indv.get_first_float("schema:price") {
    println!("Price: {:.2}", price);
}
```

## Boolean Type

Simple true/false values.

```rust
// Add boolean values
indv.add_bool("schema:active", true);
indv.add_bool("foaf:available", false);

// Set boolean value
indv.set_bool("v-s:enabled", true);

// Retrieve boolean
if let Some(is_active) = indv.get_first_bool("schema:active") {
    println!("Active: {}", is_active);
}

// Check for specific boolean value
if indv.is_exists_bool("schema:active", true) {
    println!("User is active");
}
```

## Datetime Type

Unix timestamps stored as 64-bit integers (seconds since epoch).

```rust
use chrono::{DateTime, Utc};

// Add datetime as timestamp
indv.add_datetime("schema:birthDate", 1634567890);

// Add datetime from string (ISO 8601 format)
indv.add_datetime_from_str("dct:created", "2023-10-18T15:30:00Z");

// Set datetime
indv.set_datetime("schema:modified", Utc::now().timestamp());

// Retrieve datetime
if let Some(timestamp) = indv.get_first_datetime("schema:birthDate") {
    if let Some(datetime) = chrono::Utc.timestamp_opt(timestamp, 0).single() {
        println!("Birth date: {}", datetime.format("%Y-%m-%d"));
    }
}
```

## URI Type

Uniform Resource Identifiers as strings.

```rust
// Add single URI
indv.add_uri("foaf:homepage", "https://example.com");
indv.add_uri("rdf:type", "foaf:Person");

// Set single URI
indv.set_uri("schema:url", "https://example.org/profile");

// Set multiple URIs
indv.set_uris("owl:sameAs", vec![
    "https://example.com/person/123".to_string(),
    "https://data.example.org/person/456".to_string(),
]);

// URIs are retrieved as literals
if let Some(uri) = indv.get_first_literal("foaf:homepage") {
    println!("Homepage: {}", uri);
}
```

## Binary Type

Raw binary data for files, images, or other binary content.

```rust
// Add binary data
let image_data = std::fs::read("image.png").unwrap();
indv.add_binary("schema:image", image_data);

// Set binary data
let document_data = vec![1, 2, 3, 4, 5];
indv.set_binary("foaf:document", document_data);

// Retrieve binary data
if let Some(binary_data) = indv.get_first_binobj("schema:image") {
    std::fs::write("output.png", binary_data).unwrap();
}
```

## Type Checking and Validation

### Existence Checks

```rust
// Check if predicate exists (any type)
if indv.is_exists("foaf:name") {
    println!("Name is set");
}

// Check for specific string values
if indv.any_exists("rdf:type", &["foaf:Person", "schema:Person"]) {
    println!("Is a person");
}

// Check for specific boolean value
if indv.is_exists_bool("schema:active", true) {
    println!("Is active");
}
```

### Getting Predicates by Type

```rust
use v_individual_model::onto::datatype::DataType;

// Get all predicates
let all_predicates = indv.get_predicates();

// Get predicates of specific type
let string_predicates = indv.get_predicates_of_type(DataType::String);
let integer_predicates = indv.get_predicates_of_type(DataType::Integer);
```

## Best Practices

### Type Selection
- Use `String` for text data that may need internationalization
- Use `Integer` for counts, IDs, and discrete values
- Use `Decimal` for monetary values, measurements, and precise calculations
- Use `Boolean` for flags and simple yes/no states
- Use `Datetime` for timestamps and calendar dates
- Use `URI` for identifiers, links, and type references
- Use `Binary` for files, images, and serialized data

### Performance Considerations
- Strings with language tags have higher memory overhead
- Decimal operations are more expensive than integer operations
- Binary data should be compressed when possible
- Use appropriate retrieval methods to avoid unnecessary parsing

### Data Integrity
- Validate data before adding to individuals
- Use appropriate types for your domain
- Consider using URI types for references to other individuals
- Handle missing data gracefully with Option types
