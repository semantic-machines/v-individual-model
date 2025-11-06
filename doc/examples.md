# Examples

This document provides practical examples of using the V-Individual-Model library.

## Basic Usage

### Creating and Populating Individuals

```rust
use v_individual_model::onto::individual::Individual;
use v_individual_model::onto::datatype::Lang;

fn main() {
    // Create a new individual
    let mut person = Individual::default();

    // Set the identifier
    person.set_id("example:person:john-doe");

    // Add basic information
    person.add_string("rdfs:label", "John Doe", Lang::none());
    person.add_string("rdfs:label", "Джон Доу", Lang::new_from_str("ru"));
    person.add_integer("foaf:age", 30);
    person.add_uri("rdf:type", "foaf:Person");
    person.add_bool("schema:active", true);

    // Add contact information
    person.add_uri("foaf:mbox", "mailto:john.doe@example.com");
    person.add_string("foaf:phone", "+1-555-0123", Lang::none());

    // Add work information
    person.add_string("schema:jobTitle", "Software Engineer", Lang::new_from_str("en"));
    person.add_uri("schema:worksFor", "example:organization:acme-corp");

    println!("Created person: {}", person.get_id());
}
```

### Working with Different Data Types

```rust
use v_individual_model::onto::individual::Individual;
use v_individual_model::onto::datatype::Lang;

fn create_product() {
    let mut product = Individual::default();
    product.set_id("example:product:widget-123");

    // String with language support
    product.add_string("schema:name", "Super Widget", Lang::new_from_str("en"));
    product.add_string("schema:name", "Супер Виджет", Lang::new_from_str("ru"));

    // Numeric types
    product.add_integer("schema:sku", 12345);
    product.add_decimal_from_str("schema:price", "99.99");
    product.add_decimal_from_i64("schema:stockQuantity", 150);

    // URI references
    product.add_uri("schema:category", "example:category:electronics");
    product.add_uri("schema:manufacturer", "example:organization:acme-corp");

    // Date/time
    product.add_datetime_from_str("schema:releaseDate", "2023-06-15T00:00:00Z");

    // Boolean flags
    product.add_bool("schema:isAvailable", true);
    product.add_bool("schema:isDiscontinued", false);
}
```

## Serialization Examples

### JSON Operations

```rust
use v_individual_model::onto::json2individual;
use serde_json::Value;

fn json_example() -> Result<(), Box<dyn std::error::Error>> {
    // Create individual from JSON
    let json_str = r#"
    {
        "@": "example:person:123",
        "rdfs:label": [
            {"data": "Alice Smith", "lang": "en", "type": "String"}
        ],
        "foaf:age": [{"data": 28, "type": "Integer"}],
        "schema:email": [{"data": "alice@example.com", "type": "String"}]
    }
    "#;

    let json_value: Value = serde_json::from_str(json_str)?;
    let mut person = Individual::default();

    json2individual::parse_json_to_individual(&json_value, &mut person);

    // Access the data
    let name = person.get_first_literal("rdfs:label").unwrap();
    let age = person.get_first_integer("foaf:age").unwrap();

    println!("Name: {}, Age: {}", name, age);

    // Convert back to JSON
    let json_output = person.get_obj().as_json_str();
    println!("JSON output: {}", json_output);

    Ok(())
}
```

### MessagePack Operations

```rust
use v_individual_model::onto::individual::{Individual, RawObj};
use v_individual_model::onto::individual2msgpack;
use v_individual_model::onto::msgpack2individual;
use v_individual_model::onto::datatype::Lang;

fn msgpack_example() -> Result<(), Box<dyn std::error::Error>> {
    // Create and populate individual
    let mut person = Individual::default();
    person.set_id("example:person:456");
    person.add_string("rdfs:label", "Bob Johnson", Lang::none());
    person.add_integer("foaf:age", 35);

    // Serialize to MessagePack
    let mut msgpack_out = Vec::new();
    individual2msgpack::to_msgpack(&person, &mut msgpack_out)?;

    // Deserialize from MessagePack
    let mut raw_obj = RawObj::new(msgpack_out);
    let parsed_uri = msgpack2individual::parse_msgpack(&mut raw_obj)?;

    let mut parsed_person = Individual::new_raw(raw_obj);
    parsed_person.parse_all();

    let name = parsed_person.get_first_literal("rdfs:label").unwrap();
    println!("Parsed: {} - {}", parsed_uri, name);

    Ok(())
}
```

## Query and Retrieval Examples

### Retrieving Data

```rust
use v_individual_model::onto::individual::Individual;

fn query_examples(person: &mut Individual) {
    // Get single values
    if let Some(name) = person.get_first_literal("rdfs:label") {
        println!("Name: {}", name);
    }

    if let Some(age) = person.get_first_integer("foaf:age") {
        println!("Age: {}", age);
    }

    // Get all values for a predicate
    if let Some(emails) = person.get_literals("foaf:mbox") {
        println!("Email addresses:");
        for email in emails {
            println!("  - {}", email);
        }
    }

    // Get all predicates
    let predicates = person.get_predicates();
    println!("Available predicates: {:?}", predicates);

    // Check existence
    if person.is_exists("foaf:knows") {
        println!("Person has known contacts");
    }

    // Check for specific values
    if person.any_exists("rdf:type", &["foaf:Person", "schema:Person"]) {
        println!("This is a person entity");
    }
}
```

### Working with Language Tags

```rust
use v_individual_model::onto::individual::Individual;
use v_individual_model::onto::datatype::Lang;

fn language_examples() {
    let mut article = Individual::default();
    article.set_id("example:article:123");

    // Add title in multiple languages
    article.add_string("dct:title", "Introduction to Rust", Lang::new_from_str("en"));
    article.add_string("dct:title", "Введение в Rust", Lang::new_from_str("ru"));
    article.add_string("dct:title", "Rust 简介", Lang::new_from_str("zh"));

    // Get title with language preference
    let preferred_langs = vec![
        Lang::new_from_str("ru"),  // Russian first
        Lang::new_from_str("en"),  // English second
        Lang::none()               // Any language as fallback
    ];

    if let Some(title) = article.get_first_literal_with_lang("dct:title", &preferred_langs) {
        println!("Title (preferred language): {}", title);
    }

    // Get all titles with their languages
    if let Some(titles) = article.get_literals("dct:title") {
        println!("All titles:");
        for title in titles {
            println!("  - {}", title);
        }
    }
}
```

## Advanced Operations

### Merging and Manipulating Individuals

```rust
use v_individual_model::onto::individual::Individual;
use v_individual_model::onto::datatype::Lang;

fn merge_examples() -> Result<(), Box<dyn std::error::Error>> {
    let mut person1 = Individual::default();
    person1.set_id("example:person:1");
    person1.add_string("foaf:firstName", "John", Lang::none());
    person1.add_string("foaf:lastName", "Doe", Lang::none());

    let mut person2 = Individual::default();
    person2.set_id("example:person:1"); // Same ID
    person2.add_string("foaf:phone", "+1-555-0100", Lang::none());
    person2.add_uri("foaf:knows", "example:person:2");

    // Apply predicates from person2 to person1
    person1.apply_predicate_as_add_unique("foaf:knows", &mut person2);

    // Override phone number
    person1.apply_predicate_as_set("foaf:phone", &mut person2);

    println!("Merged person has {} predicates", person1.get_predicates().len());

    Ok(())
}
```

### Working with Collections

```rust
use v_individual_model::onto::individual::Individual;
use v_individual_model::onto::datatype::Lang;

fn collection_example() {
    let mut organization = Individual::default();
    organization.set_id("example:org:acme");

    // Add multiple employees
    organization.add_uri("schema:employee", "example:person:alice");
    organization.add_uri("schema:employee", "example:person:bob");
    organization.add_uri("schema:employee", "example:person:charlie");

    // Add multiple addresses
    organization.add_string("schema:address", "123 Main St, Anytown, USA", Lang::none());
    organization.add_string("schema:address", "456 Second Ave, Somewhere, USA", Lang::none());

    // Get all employees
    if let Some(employees) = organization.get_literals("schema:employee") {
        println!("Organization has {} employees", employees.len());
        for employee in employees {
            println!("  - {}", employee);
        }
    }
}
```

## Error Handling

### Comprehensive Error Handling

```rust
use v_individual_model::onto::individual::{Individual, RawObj};
use v_individual_model::onto::msgpack2individual;
use std::error::Error;

fn robust_example() -> Result<(), Box<dyn Error>> {
    let mut person = Individual::default();

    // Safe parsing with error handling
    match person.get_first_literal_or_err("foaf:name") {
        Ok(name) => println!("Name: {}", name),
        Err(err) => println!("Name not found: {}", err),
    }

    // Safe numeric conversion
    if let Some(age_str) = person.get_first_literal("foaf:age") {
        match age_str.parse::<i64>() {
            Ok(age) => println!("Parsed age: {}", age),
            Err(_) => println!("Invalid age format"),
        }
    }

    // Safe MessagePack operations
    let mut raw_obj = RawObj::new(vec![1, 2, 3]); // Invalid data
    match msgpack2individual::parse_msgpack(&mut raw_obj) {
        Ok(uri) => println!("Parsed URI: {}", uri),
        Err(code) => println!("Parse error code: {}", code),
    }

    Ok(())
}
```

## Performance Patterns

### Efficient Bulk Operations

```rust
use v_individual_model::onto::individual::Individual;

fn bulk_processing_example(individuals: Vec<Individual>) {
    // Parse all data at once for bulk operations
    let mut parsed_individuals = Vec::new();

    for mut indv in individuals {
        indv.parse_all(); // Parse everything at once
        parsed_individuals.push(indv);
    }

    // Now all data is readily available
    for indv in &parsed_individuals {
        if let (Some(name), Some(age)) = (
            indv.get_first_literal("foaf:name"),
            indv.get_first_integer("foaf:age")
        ) {
            println!("{} is {} years old", name, age);
        }
    }
}
```

### Memory-Efficient Processing

```rust
use v_individual_model::onto::individual::{Individual, RawObj};
use v_individual_model::onto::msgpack2individual;

fn memory_efficient_example(large_raw_data: RawObj) {
    // Process large datasets without loading everything into memory
    let mut person = Individual::new_raw(large_raw_data);

    // Only parse what you need
    if person.get_first_bool("schema:active").unwrap_or(false) {
        // Parse additional data only for active users
        msgpack2individual::parse_msgpack_to_predicate("foaf:name", &mut person).ok();
        msgpack2individual::parse_msgpack_to_predicate("foaf:mbox", &mut person).ok();

        // Process active user
        if let Some(email) = person.get_first_literal("foaf:mbox") {
            println!("Sending email to: {}", email);
        }
    }
}
```

## Real-World Use Cases

### User Profile Management

```rust
use v_individual_model::onto::individual::Individual;
use v_individual_model::onto::json2individual;
use v_individual_model::onto::datatype::Lang;
use std::error::Error;

fn create_user_profile(user_data: serde_json::Value) -> Result<Individual, Box<dyn Error>> {
    let mut profile = Individual::default();

    // Parse JSON data
    if !json2individual::parse_json_to_individual(&user_data, &mut profile) {
        return Err("Failed to parse user profile JSON".into());
    }

    // Validate required fields
    if profile.get_first_literal("foaf:name").is_none() {
        return Err("Name is required".into());
    }

    // Set defaults
    if !profile.is_exists("schema:active") {
        profile.add_bool("schema:active", true);
    }

    // Add metadata
    profile.add_datetime("dct:created", chrono::Utc::now().timestamp());
    profile.add_uri("rdf:type", "foaf:Person");

    Ok(profile)
}
```

### Product Catalog

```rust
use v_individual_model::onto::individual::Individual;
use v_individual_model::onto::json2individual;
use std::error::Error;

fn create_product_catalog(products: Vec<serde_json::Value>) -> Result<Vec<Individual>, Box<dyn Error>> {
    let mut catalog = Vec::new();

    for product_data in products {
        let mut product = Individual::default();
        if !json2individual::parse_json_to_individual(&product_data, &mut product) {
            return Err(format!("Failed to parse product data: {:?}", product_data).into());
        }

        // Validate product data
        if product.get_first_literal("schema:name").is_none() {
            return Err(format!("Product missing name: {:?}", product_data).into());
        }

        // Add catalog metadata
        product.add_datetime("schema:dateModified", chrono::Utc::now().timestamp());
        product.add_bool("schema:inStock", true);

        catalog.push(product);
    }

    Ok(catalog)
}
```
