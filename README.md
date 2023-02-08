<p align="center">
  <img src="website/home/images/logo.svg">
</p>

<h1 align="center">
  The Warp Programming Language
</h1>

<p align="center">
  Warp is a programming language that’s natural to read, write and learn.
</p>

## Learn Warp

-   Visit the [Warp Playground](https://playground.warp.za16.co) and click Learn in the top right for Warp tutorials.
-   Visit the [Warp Guide](https://warp.za16.co/guide) for quick start guides, tutorials for advanced concepts, and the language reference.

## Development workflow

The Warp project is split across several folders. The compiler and tooling are written in [Rust](https://rust-lang.org), and the playground is written in [Next.js](https://nextjs.org). The guide is published using [mdBook](https://github.com/rust-lang/mdBook).

You can use these commands to build or test Warp:

```shell
# Test using a local file
cargo run --bin warp -- run path/to/test.wpl

# Test with diagnostic tracing enabled
cargo run --bin warp -- run path/to/test.wpl --trace

# Run automated tests
cargo run --bin warp-test -- tools/test/tests

# Run the playground on a local development server
cd website/playground && npm install && npm run dev

# View the documentation on a local development server
cd website/guide && mdbook serve
```
