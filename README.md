# Tree-sitter Tags

This fork extracts code navigation tags and preserves custom `@metadata`
captures on each tag. It requires Rust 1.90 or newer and Tree-sitter 0.27.

### Usage

Add this crate and the grammar for the language you want to parse to your
`Cargo.toml`:

```toml
[dependencies]
npezza93_tree-sitter-tags = "0.22.0"
tree-sitter-ruby = "0.23.1"
```

Create one tag context per thread, configure a query, and iterate over its tags:

```rust
use npezza93_tree_sitter_tags::{TagsConfiguration, TagsContext};

let config = TagsConfiguration::new(
    tree_sitter_ruby::LANGUAGE.into(),
    r#"((call method: (identifier) @metadata
              (argument_list (simple_symbol) @name)) @definition.macro)"#,
    "",
).unwrap();
let mut context = TagsContext::new();
let (tags, has_error) = context
    .generate_tags(&config, b"attr_reader :name", None)
    .unwrap();
assert!(!has_error);
for tag in tags {
    let tag = tag.unwrap();
    println!("kind: {}", config.syntax_type_name(tag.syntax_type_id));
    println!("name range: {:?}", tag.name_range);
    assert_eq!(tag.metadata.as_deref(), Some("attr_reader"));
}
```

Pass an `AtomicUsize` cancellation flag to `generate_tags` to interrupt parsing
or tag iteration when its value becomes nonzero. Each call resets the parser;
a later call can reuse the context with a cleared flag or without one.
