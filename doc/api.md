# API Reference

This document provides a complete API reference for the V-Individual-Model library.

## Core Types

### IndividualError

Enumeration for error states.

```rust
pub enum IndividualError {
    None,
    ParseError,
}
```

### Individual

Main data structure representing a semantic entity.

```rust
pub struct Individual {
    // Note: obj and raw fields are not public outside the crate
    // Use get_obj() to access IndividualObj
    pub(crate) obj: IndividualObj,
    pub(crate) raw: RawObj,
}
```

#### Methods

**Creation and Setup**
- `Individual::default()` - Create empty individual
- `Individual::new_raw(raw: RawObj)` - Create from raw data
- `Individual::new_from_obj(obj: &IndividualObj)` - Create from existing object
- `set_id(&mut self, id: &str)` - Set URI identifier
- `get_id(&self) -> &str` - Get URI identifier

**Raw Data Operations**
- `set_raw(&mut self, data: &[u8])` - Set raw binary data
- `get_raw_len(&self) -> usize` - Get raw data length
- `get_raw_data(&self) -> &[u8]` - Get raw binary data
- `parse_all(&mut self) -> &mut Individual` - Parse all raw data
- `is_empty(&self) -> bool` - Check if individual is empty
- `get_obj(&self) -> &IndividualObj` - Get object reference
- `reset(&mut self)` - Clear all data and reset parsing state

### IndividualObj

Underlying object storing predicate-resource mappings.

**Data Operations (available on Individual)**
- `add_string(&mut self, predicate: &str, value: &str, lang: Lang)` - Add string value
- `set_string(&mut self, predicate: &str, value: &str, lang: Lang)` - Set string value
- `add_uri(&mut self, predicate: &str, value: &str)` - Add URI value
- `set_uri(&mut self, predicate: &str, value: &str)` - Set URI value
- `set_uris(&mut self, predicate: &str, values: Vec<String>)` - Set multiple URIs
- `add_integer(&mut self, predicate: &str, value: i64)` - Add integer value
- `set_integer(&mut self, predicate: &str, value: i64)` - Set integer value
- `add_decimal_d(&mut self, predicate: &str, mantissa: i64, exponent: i64)` - Add decimal
- `set_decimal_d(&mut self, predicate: &str, mantissa: i64, exponent: i64)` - Set decimal
- `add_decimal_from_str(&mut self, predicate: &str, value: &str)` - Add decimal from string
- `set_decimal_from_str(&mut self, predicate: &str, value: &str)` - Set decimal from string
- `add_decimal_from_i64(&mut self, predicate: &str, value: i64)` - Add decimal from i64
- `set_decimal_from_i64(&mut self, predicate: &str, value: i64)` - Set decimal from i64
- `add_decimal_from_f64(&mut self, predicate: &str, value: f64)` - Add decimal from f64
- `set_decimal_from_f64(&mut self, predicate: &str, value: f64)` - Set decimal from f64
- `add_datetime(&mut self, predicate: &str, value: i64)` - Add datetime
- `set_datetime(&mut self, predicate: &str, value: i64)` - Set datetime
- `add_datetime_from_str(&mut self, predicate: &str, value: &str)` - Add datetime from string
- `set_datetime_from_str(&mut self, predicate: &str, value: &str)` - Set datetime from string
- `add_bool(&mut self, predicate: &str, value: bool)` - Add boolean value
- `set_bool(&mut self, predicate: &str, value: bool)` - Set boolean value
- `add_binary(&mut self, predicate: &str, value: Vec<u8>)` - Add binary data
- `set_binary(&mut self, predicate: &str, value: Vec<u8>)` - Set binary data
- `set_resources(&mut self, predicate: &str, values: &[Resource])` - Set resources from array

**Query Operations**
- `get_first_literal(&self, predicate: &str) -> Option<String>` - Get first literal value
- `get_first_literal_or_err(&self, predicate: &str) -> Result<String, std::io::Error>` - Get first literal with error
- `get_first_literal_with_lang(&self, predicate: &str, langs: &[Lang]) -> Option<String>` - Get literal with language filter
- `get_first_integer(&self, predicate: &str) -> Option<i64>` - Get first integer
- `get_first_bool(&self, predicate: &str) -> Option<bool>` - Get first boolean
- `get_first_datetime(&self, predicate: &str) -> Option<i64>` - Get first datetime
- `get_first_number(&self, predicate: &str) -> Option<(i64, i64)>` - Get first decimal as (mantissa, exponent)
- `get_first_float(&self, predicate: &str) -> Option<f64>` - Get first float
- `get_first_binobj(&self, predicate: &str) -> Option<Vec<u8>>` - Get first binary object

**Multiple Values**
- `get_resources(&self, predicate: &str) -> Option<Vec<Resource>>` - Get all resources
- `get_literals(&self, predicate: &str) -> Option<Vec<String>>` - Get all literals
- `get_literals_nm(&self, predicate: &str) -> Option<Vec<String>>` - Get literals without parsing
- `get_predicates(&self) -> Vec<String>` - Get all predicates
- `get_predicates_nm(&self) -> Vec<String>` - Get predicates without parsing
- `get_predicates_of_type(&self, datatype: DataType) -> Vec<String>` - Get predicates of specific type

**Existence Checks**
- `is_exists(&self, predicate: &str) -> bool` - Check if predicate exists
- `any_exists(&self, predicate: &str, values: &[&str]) -> bool` - Check if any value exists
- `any_exists_v(&self, predicate: &str, values: &Vec<String>) -> bool` - Check if any value exists (vector)
- `is_exists_bool(&self, predicate: &str, value: bool) -> bool` - Check boolean value exists

**Resource Management**
- `remove(&mut self, predicate: &str)` - Remove predicate completely
- `clear(&mut self, predicate: &str)` - Clear predicate values
- `apply_predicate_as_set(&mut self, predicate: &str, other: &mut Individual)` - Set predicate from another individual
- `apply_predicate_as_add_unique(&mut self, predicate: &str, other: &mut Individual)` - Add unique values from another individual
- `apply_predicate_as_remove(&mut self, predicate: &str, other: &mut Individual)` - Remove values present in another individual

**Serialization (available on IndividualObj via get_obj())**
- `as_json_str(&self) -> String` - Convert to JSON string
- `as_json(&self) -> serde_json::Value` - Convert to JSON value

Example usage:
```rust
let json_str = indv.get_obj().as_json_str();
let json_value = indv.get_obj().as_json();
```

**Comparison**
- `compare(&self, other: &Individual, ignore_predicates: Vec<&str>) -> bool` - Compare individuals

### DataType

Enumeration of supported data types.

```rust
pub enum DataType {
    String,
    Integer,
    Decimal,
    Boolean,
    Datetime,
    Uri,
    Binary,
}
```

### Value

Enumeration for storing typed values.

```rust
pub enum Value {
    Int(i64),
    Str(String, Lang),
    Uri(String),
    Bool(bool),
    Num(i64, i64),  // (mantissa, exponent) for decimal numbers
    Binary(Vec<u8>),
    Datetime(i64),
}
```

### Resource

Structure representing a typed resource with ordering.

```rust
pub struct Resource {
    pub rtype: DataType,
    pub value: Value,
    pub order: u16,
}
```

### RawObj

Container for raw binary data with parsing state.

```rust
pub struct RawObj {
    pub data: Vec<u8>,
    // Internal parsing state fields (cur, len_predicates, cur_predicates, raw_type)
}
```

#### Methods
- `RawObj::new(data: Vec<u8>) -> RawObj` - Create with data
- `RawObj::new_empty() -> RawObj` - Create empty
- `reset(&mut self)` - Reset parsing state

### Lang

Language tag for internationalization support.

```rust
pub struct Lang {
    // Implementation details
}
```

#### Methods
- `Lang::none() -> Lang` - No language specified
- `Lang::new_from_str(lang: &str) -> Lang` - Create from string
- `Lang::new_from_i64(lang: i64) -> Lang` - Create from integer

## Serialization Modules

### MessagePack
- `msgpack2individual::parse_msgpack(raw: &mut RawObj) -> Result<String, i8>` - Parse MessagePack to get URI
- `msgpack2individual::parse_msgpack_to_predicate(predicate: &str, iraw: &mut Individual) -> Result<(), String>` - Parse specific predicate
- `individual2msgpack::to_msgpack(indv: &Individual, out: &mut Vec<u8>) -> Result<(), Error>` - Convert to MessagePack

### JSON
- `json2individual::parse_json_to_individual(json: &serde_json::Value, indv: &mut Individual) -> bool` - Parse from JSON
- JSON serialization methods are available on `IndividualObj`:
  - `as_json_str(&self) -> String` - Convert to JSON string
  - `as_json(&self) -> serde_json::Value` - Convert to JSON value
  - Access via `indv.get_obj().as_json_str()` or `indv.get_obj().as_json()`

### CBOR
- `cbor2individual::parse_cbor(raw: &mut RawObj) -> Result<String, i8>` - Parse CBOR to get URI
- `cbor2individual::parse_cbor_to_predicate(predicate: &str, iraw: &mut Individual) -> Result<(), String>` - Parse specific predicate

### Turtle
- `individual2turtle::to_turtle_with_counter_refs(indvs: &[&Individual], all_prefixes: &HashMap<String, String>) -> Result<Vec<u8>, io::Error>` - Convert to Turtle format

## Constants

XSD datatype URIs are available as constants in the `onto` module:

- `XSD_LONG` - 64-bit integers
- `XSD_INT` - 32-bit integers
- `XSD_INTEGER` - Arbitrary-size integers
- `XSD_BOOLEAN` - Boolean values
- `XSD_DECIMAL` - Decimal numbers
- `XSD_FLOAT` - 32-bit floats
- `XSD_DOUBLE` - 64-bit floats
- `XSD_DATE_TIME` - Date and time
- `XSD_STRING` - Character strings
- `XSD_NORMALIZED_STRING` - Normalized strings
- `XSD_NEGATIVE_INTEGER` - Negative integers
- `XSD_NON_NEGATIVE_INTEGER` - Non-negative integers
- `XSD_NON_POSITIVE_INTEGER` - Non-positive integers
- `XSD_POSITIVE_INTEGER` - Positive integers
