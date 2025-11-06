# Architecture

This document describes the architecture and design principles of the V-Individual-Model library.

## Core Concepts

### Individual
The `Individual` is the main data structure that represents a semantic entity. Each individual has:

- **URI identifier** - unique identifier for the entity
- **Predicates** - named properties that can hold multiple values
- **Resources** - typed values associated with predicates

### IndividualObj
The underlying object that stores the actual data. It manages:

- Predicate-to-resource mappings
- Resource ordering and indexing
- Type-safe value storage

### RawObj
Container for raw binary data that supports lazy parsing. Features:

- Raw binary data storage
- Parsing cursor state management
- Support for partial parsing of large datasets

## Module Structure

```
src/
├── lib.rs              # Library entry point
└── onto/               # Ontology module
    ├── mod.rs          # Module definitions and XSD constants
    ├── individual.rs   # Core Individual struct
    ├── datatype.rs     # DataType enum and Value enum
    ├── resource.rs     # Resource struct
    ├── parser.rs       # Common parsing utilities
    ├── onto_impl.rs    # Implementation details
    ├── onto_index.rs   # Indexing functionality
    ├── cbor2individual.rs      # CBOR parsing
    ├── json2individual.rs      # JSON parsing
    ├── msgpack2individual.rs   # MessagePack parsing
    ├── individual2json.rs      # JSON serialization
    ├── individual2msgpack.rs   # MessagePack serialization
    ├── individual2turtle.rs    # Turtle serialization
    └── turtle_formatters_with_prefixes.rs
```

## Design Principles

### 1. Type Safety
All data operations are type-safe with compile-time guarantees:
- DataType enum ensures valid type handling
- Value enum provides type-safe value storage
- Resource struct maintains type information

### 2. Lazy Parsing
Raw binary data is parsed only when needed:
- RawObj stores binary data with cursor state
- Methods like `get_first_literal()` trigger parsing
- `parse_all()` forces complete parsing when required

### 3. Format Agnostic
Core data structures work independently of serialization format:
- Individual can be created from any supported format
- Conversion between formats is supported
- Format-specific code is isolated in dedicated modules

### 4. Memory Efficiency
Optimized for memory usage:
- Binary data stored efficiently
- Lazy evaluation prevents unnecessary allocations
- Resource reuse where possible

### 5. Extensibility
Designed for easy extension:
- New data types can be added to DataType enum
- New serialization formats can be implemented
- Modular architecture allows feature isolation

## Data Flow

### Serialization Flow
```
Individual → Format-specific serializer → Binary/String output
```

### Deserialization Flow
```
Raw binary → Format-specific parser → RawObj → Individual
```

### Parsing States
1. **Raw State**: Binary data stored, not parsed
2. **Partial State**: Some predicates parsed, others remain raw
3. **Complete State**: All data parsed into IndividualObj

## Error Handling

The library uses Result types for error handling:
- Parse errors return detailed error messages
- Type conversion errors provide context
- Invalid data operations return appropriate error codes

## Performance Considerations

### Parsing Strategies
- **Lazy parsing** for individual value access
- **Bulk parsing** for complete data processing
- **Streaming parsing** for large datasets

### Memory Management
- **Reference counting** for shared data
- **Copy-on-write** semantics
- **Efficient binary storage**

### Concurrency
- **Immutable by default** for thread safety
- **Mutable operations** require exclusive access
- **No internal locking** (caller responsibility)
