[![crates.io](https://img.shields.io/crates/v/cargo-leptos)](https://crates.io/crates/cargo-leptos)
[![Discord](https://img.shields.io/discord/1031524867910148188?color=%237289DA&label=discord)](https://discord.gg/YdRAhS7eQB)

Build tool for [Leptos](https://crates.io/crates/leptos):

[<img src="https://raw.githubusercontent.com/gbj/leptos/main/docs/logos/Leptos_logo_RGB.png" alt="Leptos Logo" style="width: 30%; height: auto; display: block; margin: auto;">](http://https://crates.io/crates/leptos)

<br/>

- [Features](#features)
- [Getting started](#getting-started)
- [Single-package setup](#single-package-setup)
- [Workspace setup](#workspace-setup)
- [Build features](#build-features)
- [Parameters reference](#parameters-reference)
  - [Compilation parameters](#compilation-parameters)
  - [Site parameters](#site-parameters)
  - [Environment variables](#environment-variables)

<br/>

# Features

- Parallel build of server and client in watch mode for fast developer feedback.
- Build server and client for hydration (client-side rendering mode not supported).
- Support for both workspace and single-package setup.
- SCSS compilation using [dart-sass](https://sass-lang.com/dart-sass).
- CSS transformation and minification using [Lightning CSS](https://lightningcss.dev). See docs for full details.
- Builds server and client (wasm) binaries using Cargo.
- Generates JS - Wasm bindings with [wasm-bindgen](https://crates.io/crates/wasm-bindgen)
- Optimises the wasm with _wasm-opt_ from [Binaryen](https://github.com/WebAssembly/binaryen)
- Generation of rust code for integrating with a server of choice.
- `watch` command for automatic rebuilds with browser live-reload.
- `test` command for running tests. Note that this runs `cargo test` for the two different modes (`hydrate` and `ssr`).
- `build` build the server and client.
- `end2end` command for building, running the server and calling a bash shell hook. The hook would typically launch Playwright or similar.
- `new` command for creating a new project based on templates, using [cargo-generate](https://cargo-generate.github.io/cargo-generate/index.html). WIP: You'll need to ask on the Leptos [discord](https://discord.gg/YdRAhS7eQB) for the url of a template.

  <br/>

# Getting started

Install:

> `cargo install --locked cargo-leptos`

If you for any reason needs the bleeding-edge super fresh version:

> `cargo install --git https://github.com/akesson/cargo-leptos cargo-leptos`

Help:

> `cargo leptos --help`

For setting up your project, have a look at the [examples](https://github.com/akesson/cargo-leptos/tree/main/examples)

<br/>

# Single-package setup

The single-package setup is where the code for both the frontend and the server is defined in a single package.

Configuration parameters are defined in the package `Cargo.toml` section `[package.metadata.leptos]`. See the Parameters reference for
a full list of parameters that can be used. All paths are relative to the package root (i.e. to the `Cargo.toml` file)

<br/>

# Workspace setup

When using a workspace setup both single-package and multi-package projects are supported. The latter is when the frontend
and the server reside in different packages.

All workspace members whose `Cargo.toml` define the `[package.metadata.leptos]` section are automatically included as Leptos
single-package projects. The multi-package projects are defined on the workspace level in the `Cargo.toml`'s
section `[[workspace.metadata.leptos]]` which takes three mandatory parameters:

```toml
[[workspace.metadata.leptos]]
# project name
name = "leptos-project"
bin-package = "server"
lib-package = "front"

# more configuration parameters...
```

Note the double braces: several projects can be defined and one package can be used in several projects.

<br/>

# Build features

When building with cargo-leptos, the frontend, library package, is compiled into wasm using target
`wasm-unknown-unknown` and the features `--no-default-features --features=hydrate`
The server binary is compiled with the features `--no-default-features --features=ssr`

<br/>

# Parameters reference

These parameters are used either in the workspace section `[[workspace.metadata.leptos]]` or the package,
for single-package setups, section `[package.metadata.leptos]`.

## Compilation parameters

```toml
# Sets the name of the binary target used.
#
# Optional, only necessary if the bin-package defines more than one target
bin-target = "my-bin-name"

# The features to use when compiling the bin target
#
# Optional. Can be over-ridden with the command line parameter --bin-features
bin-features = ["ssr"]

# If the --no-default-features flag should be used when compiling the bin target
#
# Optional. Defaults to false.
bin-default-features = false

# The features to use when compiling the lib target
#
# Optional. Can be over-ridden with the command line parameter --lib-features
lib-features = ["hydrate"]

# If the --no-default-features flag should be used when compiling the lib target
#
# Optional. Defaults to false.
lib-default-features = false
```

## Site parameters

These parameters can be overridden by setting the corresponding environment variable. They can also be
set in a `.env` file as cargo-leptos reads the first it finds in the package or workspace directory and
any parent directory.

```toml
# Sets the name of the output js, wasm and css files.
#
# Optional, defaults to the lib package name or, in a workspace, the project name. Env: LEPTOS_OUTPUT_NAME.
output-name = "myproj"

# The site root folder is where cargo-leptos generate all output.
# NOTE: It is relative to the workspace root when running in a workspace.
# WARNING: all content of this folder will be erased on a rebuild.
#
# Optional, defaults to "target/site". Env: LEPTOS_SITE_ROOT.
site-root = "target/site"

# The site-root relative folder where all compiled output (JS, WASM and CSS) is written.
#
# Optional, defaults to "pkg". Env: LEPTOS_SITE_PKG_DIR.
site-pkg-dir = "pkg"

# The source style file. If it ends with _.sass_ or _.scss_ then it will be compiled by `dart-sass`
# into CSS and processed by lightning css. When release is set, then it will also be minified.
#
# Optional. Env: LEPTOS_STYLE_FILE.
style-file = "style/main.scss"

# The browserlist https://browsersl.ist query used for optimizing the CSS.
#
# Optional, defaults to "defaults". Env: LEPTOS_BROWSERQUERY.
browserquery = "defaults"

# Assets source dir. All files found here will be copied and synchronized to site-root.
# The assets-dir cannot have a sub directory with the same name/path as site-pkg-dir.
#
# Optional. Env: LEPTOS_ASSETS_DIR.
assets-dir = "assets"

# The IP and port where the server serves the content. Use it in your server setup.
#
# Optional, defaults to 127.0.0.1:3000. Env: LEPTOS_SITE_ADDR.
site-addr = "127.0.0.1:3000"

# The port number used by the reload server (only used in watch mode).
#
# Optional, defaults 3001. Env: LEPTOS_RELOAD_PORT
reload-port = 3001

# The command used for running end-to-end tests.
#
# Optional. Env: LEPTOS_END2END_CMD.
end2end-cmd = "npx playwright test"

# The directory from which the end-to-end tests are run.
#
# Optional. Env: LEPTOS_END2END_DIR
end2end-dir = "integration"
```

<br/>

## Environment variables

The following environment variables are set when compiling the lib (front) or bin (server) and when the server is run.

Echoed from the Leptos config:

- LEPTOS_OUTPUT_NAME
- LEPTOS_SITE_ROOT
- LEPTOS_SITE_PKG_DIR
- LEPTOS_SITE_ADDR
- LEPTOS_RELOAD_PORT

Directories used when building:

- LEPTOS_LIB_DIR: The path (relative to the working directory) to the library package
- LEPTOS_BIN_DIR: The path (relative to the working directory) to the binary package

Note when using directories:

- `cargo-leptos` changes the working directory to the project root or if in a workspace, the workspace root before building and running.
- the two are set to the same value when running in a single-package config.
- Avoid using them at run-time unless you can guarantee that the entire project struct is available at runtime as well.
