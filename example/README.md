# Rust2Go Example

## Steps
1. Add dependency and build-dependency to `Cargo.toml`:
```toml
[dependencies]
rust2go = { path = "../rust2go" }

[build-dependencies]
rust2go = { path = "../rust2go", features = ["gen"] }
```

    And also install commandline tool:
```sh
cargo install --force rust2go-cli
```

2. Create an empty `user.rs` and add it to `main.rs`: `mod user;`.

3. Define request and response structs, and calling conventions in `user.rs`:
```rust
pub struct DemoRequest {
    pub name: String,
    pub age: u8,
}

pub struct DemoResponse {
    pub pass: bool,
}

pub trait DemoCall {
    fn demo_check(req: DemoRequest) -> DemoResponse;
    fn demo_check_async(req: DemoRequest) -> impl std::future::Future<Output = DemoResponse>;
}
```

4. Create an empty go project and initialize it: `mkdir go && cd go && go mod init r2gexample`; or you can use any existed project.

5. Generate golang code with `rust2go-cli --src src/user.rs --dst go/gen.go`.

6. Write a `build.rs` with the following content:
```rust
fn main() {
    rust2go::Builder::new()
        .with_rs_idl("./src/user.rs")
        .with_go_src("go/gen.go")
        .build();
}
```

7. Add an include to the top of `user.rs` to make sure the generated code is used:
```rust
include!(concat!(env!("OUT_DIR"), "/rust2go.rs"));
```

8. Call the golang with `user::{$trait}Impl`.
```rust
fn main() {
    let req = DemoRequest {
        name: "ihciah".to_string(),
        age: 28,
    };
    println!("User pass: {}", DemoCallImpl::demo_check(req).pass);
}
```

    You can also run a async call with `DemoCallImpl::demo_check_async(req).await`.

9. Run it and it will show `User pass: false`! Then you can edit the golang code in `go/gen.go` and customize golang side logic.