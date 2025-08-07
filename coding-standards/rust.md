# Silvermine Coding Standards Specific to Rust

Silvermine's standard Rust version and edition is:

   * Version: **`1.89.0`**
   * Edition: **`2024`**

## Code Standards

In principle, we try to stick to the Rust community's default linting and formatting
standards as defined by the default configuration for the [`clippy`][clippy] and
[`rustfmt`][rustfmt] tools.

However, we have a few exceptions where the default standards conflict with our own in a
way that would cause confusion or tooling complication when switching between editing Rust
and other languages. The exceptions are:

```toml
# The default `remove_nested_parens = true` value conflicts with our general practice to
# clarify order of operations in boolean logic and math operations with parentheses that
# are technically unnecessary but helpful for readability
remove_nested_parens = false
```

Use 3 spaces for indentation:

```toml
tab_spaces = 3
hard_tabs = false
```

Use Unix-style line endings:

```toml
newline_style = "Unix"
```

Standardize on a single syntax for try expressions and field initializers:

```toml
use_field_init_shorthand = true
use_try_shorthand = true
```

These exceptions are codified in the `rustfmt.toml` file specified in the "Project
Configuration" section below.

## Error Handling

Use the [`thiserror`][thiserror] crate to define error types for your application/library.
If there are only a few error types, or you do not need serializable errors, you may
consider just defining an error enum without using `thiserror`.

_See also: Tauri's [error handling suggestion for Tauri projects][tauri-error-handling]._

## Documentation

### General

All Rust libraries/crates should have well-structured, informative documentation available
when you run `cargo doc --open`.

Follow the Rust documentation guidelines here:

   * <https://doc.rust-lang.org/rust-by-example/meta/doc.html>
   * <https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#making-useful-documentation-comments>

Ensure that the home page of the generated documentation site explains the crate's purpose
and how to use it.

If the root README.md file contains instructions for project-developers, such as how to
run the tests, how to build the crate, etc., do not simply inline the README into the
crate's documentation using `#![doc = include_str!("../path/to/root/README.md")]`.

Either create a separate `DEVELOPERS.md` file in the root of the project that contains the
instructions for project-developers, or use module-level documentation comments (`//!`) on
the crate's root module.

For Rust applications (i.e. those not published as a crate), add Rust documentation blocks
to commonly-used modules, structs, enums, and functions.

### ⚠️ Code examples in documentation blocks ⚠️

**When writing Rust code examples in documentation blocks, set the code block type to
`no_run` to prevent `cargo test` from executing the code.**

```rust
// ✅ Good

/// # Examples
///
/// ```no_run
/// loop {
///    println!("Hello, world");\
///    my_function();
/// }
/// ```
fn my_function() {

}

// ❌ Bad - `cargo test` will automatically try to run this example code

/// # Examples
///
/// ```
/// loop {
///     println!("Hello, world");\
///     my_function();
/// }
/// ```
fn my_function() {

}
```

## Project Configuration

Quick project configuration checklist:

   * [ ] `rust-toolchain.toml` file in the root of the project
   * [ ] `.cargo/config.toml` file with lint and fix aliases
   * [ ] `Cargo.toml` file that specifies the Silvermine standard Rust `edition` (see the
         first heading in this document)
   * [ ] `rustfmt.toml` file in the same folder as `Cargo.toml`
   * [ ] Up-to-date README with instructions for linting and fixing Rust code
   * [ ] _(Node.js projects only):_ `package.json` file with Rust lint and fix scripts

### Toolchain Configuration

Create a `rust-toolchain` file in the root of the project with the following contents:

```toml
[toolchain]
channel = "1.88.0"
components = [
   "rustfmt",
   "clippy"
]
```

Optionally, define your project-specific `targets` below the `components` entry.

For [Tauri][tauri] projects that target Windows, macOS, Android, and iOS, the targets
should be:

```toml
targets = [
   "x86_64-pc-windows-msvc",
   "aarch64-apple-ios",
   "aarch64-apple-ios-sim",
   "aarch64-linux-android",
   "armv7-linux-androideabi",
   "i686-linux-android",
   "x86_64-linux-android"
]
```

### Linting

In principle, we try to stick to the Rust community's default linting and formatting
standards as defined by the default configuration for the [`clippy`][clippy] and
[`rustfmt`][rustfmt] tools. However, we have a few exceptions where the default standards
conflict with our own in a way that would either cause confusion or tooling complication
when switching between editing Rust and other languages.

Those exceptions are codified in the `rustfmt.toml` file specified below. Unfortunately,
`clippy`'s config file format is "unstable" and only allows configuration with command
line flags. However, we do not override the default `clippy` linting rules right now
anyway.

#### Linting Commands

Use this command to run linting and formatting checks:

```sh
cargo clippy --all-targets --all-features -- -D warnings && cargo fmt -- --check
```

Use this command to apply the linting and formatting auto-fixes:

```sh
cargo clippy --all-targets --all-features --fix && cargo fmt
```

Note: Because `clippy` and `rustfmt` are separate but complementary tools, it is possible
that after running the "fix" commands above the linting commands will then fail with new
errors.

If that happens, open an issue in this repo so we can investigate and resolve it.

#### Configuring linting in new projects

##### `rustfmt.toml`

Create a `rustfmt.toml` file in the same folder as your Cargo.toml file with these
contents:

```toml
tab_spaces = 3
hard_tabs = false
newline_style = "Unix"
remove_nested_parens = false
use_field_init_shorthand = true
use_try_shorthand = true
```

##### Cargo linting aliases

Add a `.cargo/config.toml` file to the root of your project with the following contents:

```toml
[alias]
lint-clippy = "clippy --all-targets --all-features -- -D warnings"
fix-clippy = "clippy --fix"
lint-fmt = "fmt -- --check"
fix-fmt = "fmt"
```

Unfortunately, the `alias` config does not support shell commands or shell operators like
`&&` or `||`, so `rustfmt` and `clippy` are defined as separate aliases.

**NOTE: If the project is a Rust-only project, ensure that the CI configuration will run:
`cargo lint-clippy && cargo lint-fmt` and fail the build with any errors. Also, ensure
that the linting aliases above are documented in the project's README file.**

###### Projects that also include JavaScript/TypeScript

Some projects, such as those built for [Tauri][tauri], may include both Rust and
JavaScript/TypeScript code. In that case, add the aliased Rust linting commands to
`package.json` and add `rust:lint` to the list of tasks that execute when running the `npm
run standards` command.

Then, ensure that whatever CI build runner image you are using to run the `npm run
standards` command also has Rust and Cargo installed.

Example `package.json`:

```json
{
   "scripts": {
      "rust:lint": "cargo lint-clippy && cargo lint-fmt",
      "rust:lint:fix": "cargo fix-clippy && cargo fix-fmt",
      "standards": "npm run rust:lint"
   }
}
```

### Cargo.toml

Each project's `Cargo.toml` file should at a minimum specify these fields:

   * `[package] name`
   * `[package] description`
   * `[package] version` - Use `semver`, and start at `0.1.0` for new projects
   * `[package] edition` - Use the Silvermine standard version
   * `[package] rust-version` - Use the Silvermine standard version. This should always
     match the version in the project's `rust-toolchain.toml` file

Public (non-internal) Rust projects should also specify a `license` and `repository`.

#### Dependencies

   * Always pin to exact semver versions. Do not use caret, tilde, wildcard, or
     comparison requirements
   * When you must use a `git` dependency, avoid using branch dependencies in production
     code. Instead, specify a tag or hash version

```toml
[dependencies]

# ✅ Good - Uses exact version
tauri-plugin-os = "2.3.0"

# ✅ Good - Uses a hash
regex = { git = "https://github.com/rust-lang/regex.git", rev = "0c0990399270277832fbb5b91a1fa118e6f63dba" }

# ❌ Bad - Uses tilde version
tauri-plugin-log = "~2.6"

# ❌ Bad - Uses git branch
regex = { git = "https://github.com/rust-lang/regex.git", branch = "next" }
```

Do not introduce new dependencies that have not been used in other Silvermine projects
without consulting with the Silvermine team first.

### Tests

Follow the Rust community standard:

   * Write unit tests [in the same file as the code you are testing][unit-tests]
   * Write integration tests [in a `tests` folder that is a sibling to the `src`
     folder][integration-tests]

**Do not write [documentation tests][doc-tests]. Always ensure code in documentation
blocks are annotated with `no_run` to prevent them from executing when running `cargo
test`.**

[tauri]: https://tauri.app/
[clippy]: https://doc.rust-lang.org/clippy/
[rustfmt]: https://github.com/rust-lang/rustfmt
[rustfmt-toml]: https://github.com/silvermine/standardization/blob/master/rustfmt.toml
[thiserror]: https://github.com/dtolnay/thiserror
[tauri-error-handling]: https://v2.tauri.app/develop/calling-rust/#error-handling
[unit-tests]: https://doc.rust-lang.org/rust-by-example/testing/unit_testing.html
[doc-tests]: https://doc.rust-lang.org/rust-by-example/testing/doc_testing.html
[integration-tests]: https://doc.rust-lang.org/rust-by-example/testing/integration_testing.html
