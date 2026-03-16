# cli-design

A [skill](https://github.com/vercel-labs/skills) for Claude Code that guides the creation of distinctive, production-grade command-line interfaces.

## Install

```sh
npx skills add mamachanko/cli-design
```

## What it does

When you ask Claude Code to build a CLI tool, terminal application, or command-line utility, this skill activates and provides opinionated guidance on:

- Command architecture and flag design
- Help text and documentation
- Output formatting and composability
- Error handling and user feedback
- Interactive prompts and TUI patterns
- Configuration hierarchy

The skill is tech-agnostic -- it applies whether you are building in Go, Rust, Python, Node.js, or any other language.

## Inspiration

This skill distills best practices from:

- [Command Line Interface Guidelines](https://clig.dev)
- [12 Factor CLI Apps](https://medium.com/@jdxcode/12-factor-cli-apps-dd3c227a0e46)
- [Cobra](https://cobra.dev) (command structure patterns)
- [Charm](https://charm.sh) (TUI interaction patterns)

## License

Apache 2.0
