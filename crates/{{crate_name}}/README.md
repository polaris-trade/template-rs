# {{crate_name}}

{{description}}

## Usage

```rust
use {{crate_name}}::{DefaultGreeter, Greeter};

let g = DefaultGreeter;
let msg = g.greet("world").unwrap();
assert_eq!(msg, "hello, world");
```

## License

MIT — see the workspace-root `LICENSE`.
