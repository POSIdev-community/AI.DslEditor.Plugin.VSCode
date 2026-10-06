# Change Log

All notable changes to the "JSA DSL" extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2026-10-06

### Added

- Added completion, syntax highlighting, and hover documentation for the no-argument `DetectReachability` statement.
- Added completion, validation, and hover documentation for .NET and Ruby PVO grammars.
- Added completion, validation, and hover documentation for weakness and secret types used in `filter` expressions.

### Changed

- Type references containing characters outside regular DSL identifiers can now be enclosed in backticks.
- `filter` now supports multiple comma-separated arguments, chained calls, and direct use with `Taint`.
- Refreshed the built-in vulnerability and PVO catalogs used by completion, validation, and hover documentation, including new common, .NET, JVM, and Ruby entries.
- Weakness and secret identifiers are now offered only where they are accepted by the JSA analysis engine: as `filter` arguments, not as vulnerability arguments of `Detect`.
- Certificate validation remains enabled for language-server downloads by default. On certificate errors, users can explicitly approve a one-time retry without validation.


## [1.1.0] - 2026-02-09

### Added
- Added support for self-contained Language Server builds.


## [1.0.0] - 2025-10-17

### Added

- Initial release of JSA DSL language support for Visual Studio Code.
- Syntax highlighting for all JSA DSL constructs.
- IntelliSense and auto-completion features.
- Comprehensive diagnostics including syntax, semantic and code quality checks.
- Quick fixes for common issues like unused parameters or redundant `pass` statements.
- Go to Definition and Find All References navigation support.
- Real-time document formatting enabled by default for `.jsa.dsl` files.
- Automatic download and setup of the JSA DSL Language Server.
- Manual installation instructions and configuration options via settings UI.
- Support for both `.jsa.dsl` and `.dsl` file extensions with toggle option.
