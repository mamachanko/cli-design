---
name: cli-design
description: Design and build distinctive, production-grade command-line interfaces with exceptional UX quality. Use this skill when the user asks to build CLI tools, terminal applications, interactive prompts, or command-line utilities (examples include CLI apps, TUI dashboards, developer tools, shell scripts with user interaction, argument parsers, or when structuring any terminal-based interface). Generates well-architected, human-first CLI code that avoids lazy, hostile, or generic terminal UX.
license: Complete terms in LICENSE
---

This skill guides creation of distinctive, production-grade command-line interfaces that treat the terminal as a first-class user experience. Implement real working code with exceptional attention to usability, feedback, and composability.

The user provides CLI requirements: a command-line tool, interactive prompt, TUI application, or terminal utility to build. They may include context about the audience, ecosystem, or technical constraints.

## Design Thinking

Before writing code, understand the context and commit to a clear interaction model:

- **Purpose**: What problem does this tool solve? Who runs it -- developers, operators, end users? How often -- once, daily, in scripts? A deployment tool for SREs demands different affordances than a file converter for casual users.
- **Interaction Model**: Pick the right shape. A pure filter (stdin to stdout) is different from a subcommand tree, which is different from a full-screen TUI. Choose deliberately: non-interactive pipeline tool, single-command utility, subcommand-based CLI, interactive prompt flow, or full-screen terminal UI. Each has distinct rules.
- **Constraints**: Target shells and platforms, dependency tolerance, whether it runs in CI/containers (non-interactive), and performance budget. A tool that runs in CI must never block on a prompt. A tool for developers can assume more; a tool for end users must assume less.
- **Composability**: Will this tool's output be piped, parsed, or consumed by other programs? If yes, structured output (JSON, TSV) and clean stdout/stderr separation are non-negotiable.

**CRITICAL**: The best CLIs feel like they were designed by someone who uses the terminal every day. They are fast, predictable, and quietly helpful. Whether building a minimal single-purpose filter or a rich interactive TUI, the key is intentionality -- every flag, every message, every exit code should exist for a reason.

Then implement working code that is:

- Production-grade: handles errors, edge cases, signals, and non-interactive environments
- Human-first: clear help, useful errors, progressive disclosure
- Composable: plays well with pipes, scripts, and other tools
- Responsive: provides feedback within 100ms, progress for long operations

## Command Architecture

Design commands like sentences. Follow the pattern `TOOL VERB NOUN --MODIFIER`:

- **Commands are verbs**: `create`, `list`, `delete`, `sync`. Not `doCreate` or `manager`. Use familiar names -- `ls` not `enumerate`, `rm` not `remove-item`.
- **Arguments are nouns**: The things being acted on. Required inputs that are obvious from context. Keep positional args to two or fewer; beyond that, use flags.
- **Flags are modifiers**: `--output json`, `--recursive`, `--dry-run`. Use meaningful flag names over positional arguments. Always provide both long (`--verbose`) and short (`-v`) forms for common flags. Boolean flags should be opt-in, not opt-out (`--color` not `--no-no-color`).
- **Subcommands over flag soup**: If a tool does multiple things, use subcommands (`git commit`, `docker build`), not a single command with mode-switching flags.
- **Persistent vs local flags**: Global concerns (verbosity, output format, config path) are persistent flags. Command-specific options are local flags.

Accept what the user gives. If they provide all arguments inline, run immediately. If arguments are missing and stdin is a TTY, prompt interactively. If stdin is not a TTY, fail with a clear message. Never silently block waiting for input that will never come.

## Help and Documentation

Help text is your CLI's user interface. Treat it with the same care as a GUI:

- **Every command must have help**: Generated automatically from code structure, not maintained as a separate artifact.
- **Usage line first**: Show the pattern before explaining it. `Usage: tool deploy <environment> [flags]`
- **Examples are mandatory**: Show 2-3 real-world invocations, not abstract syntax. Put the most common use case first.
- **Progressive disclosure**: `tool --help` shows top-level commands. `tool deploy --help` shows deploy-specific flags. Do not dump every flag of every subcommand into one screen.
- **Suggest next steps**: After completing an action, suggest what the user might want to do next. After `tool deploy staging`, suggest `tool status staging` or `tool logs staging`.
- **Shell completions**: Generate them. Bash, Zsh, Fish at minimum. This is table stakes, not a nice-to-have.

## Output and Formatting

Every byte of output should be intentional:

- **Detect your environment**: Check if stdout is a TTY. If yes, use color, formatting, and interactive elements. If piped, output clean, parseable data. Respect `NO_COLOR`, `TERM=dumb`, and `CLICOLOR` environment variables unconditionally.
- **Stdout is for data, stderr is for humans**: Program output (results, data, structured responses) goes to stdout. Status messages, progress indicators, warnings, and errors go to stderr. This is the single most important rule for composability. Never mix them.
- **Structured output as a first-class feature**: Offer `--output json` (or `--output table`, `--output yaml`). JSON output must be stable and documented -- it is an API contract. Table output is for humans; JSON is for machines. Default to table for TTY, JSON for pipes.
- **Color with purpose**: Color encodes meaning -- green for success, red for errors, yellow for warnings, dim for secondary info. Never use color as the only indicator; pair it with text (a checkmark, a prefix, an icon). Use bold sparingly for emphasis, not decoration.
- **Progress for long operations**: Any operation over 500ms should show progress. Spinners for indeterminate waits, progress bars with ETA for determinate ones. Always on stderr. Clear the progress indicator before printing results.

## Error Handling

Errors are the most important output your CLI produces. A user who hits an error is already frustrated -- your job is to guide them forward:

- **Say what happened, why, and how to fix it**: Not `Error: ENOENT` but `Error: Config file not found at ~/.tool/config.yaml. Run "tool init" to create one, or set TOOL_CONFIG to a custom path.`
- **Exit codes are semantic**: 0 is success. 1 is general failure. 2 is usage error (wrong arguments). Use distinct codes for distinct failure modes so scripts can branch on them. Document your exit codes.
- **Fail fast, fail clearly**: Validate all inputs before doing work. Do not delete half the files and then fail on a permission error. Check preconditions upfront.
- **Actionable suggestions**: When a command fails, suggest the fix. When a flag is misspelled, suggest the closest match. When a resource is missing, say how to create it. When permissions are wrong, say what to change.
- **Warnings are not errors**: Use stderr and a `Warning:` prefix. Do not exit non-zero for warnings. Let the user decide if a warning is fatal by providing a `--strict` flag.
- **Respect the user's time**: If an operation is destructive or irreversible, require confirmation (`--yes` to skip). Show exactly what will happen before doing it. Dry-run mode (`--dry-run`) should be available for any destructive operation.

## Interactivity and TUI

When building interactive terminal experiences:

- **Degrade gracefully**: Every interactive prompt must have a non-interactive equivalent via flags. CI and scripts must never hit a blocking prompt. Detect non-TTY and either use defaults or fail explicitly.
- **Selection prompts over free-text**: When the set of valid inputs is known, offer a selection list, not a text field. Fuzzy-filter long lists.
- **Spinners and progress**: Use spinners for indeterminate operations, progress bars for determinate ones. Show what is happening, not just that something is happening. `Deploying to staging...` not `Loading...`
- **Keyboard conventions**: `q` or Escape to quit. Arrow keys and vim-like `j/k` for navigation. Enter to confirm. Ctrl+C always exits cleanly with proper signal handling and cleanup. Show key hints at the bottom of the screen.
- **Full-screen TUIs**: For complex interactive applications, use a proper TUI architecture (the Elm Architecture works well: Model, Update, View). Manage state explicitly. Handle terminal resize. Restore terminal state on exit, even on panic or crash.

## Configuration

Follow the configuration hierarchy (highest to lowest priority):

1. Command-line flags (explicit, per-invocation)
2. Environment variables (per-session or per-environment, prefixed: `TOOLNAME_*`)
3. Local config file (per-project, `.tool.yaml` or `tool.config`)
4. Global config file (per-user, `~/.config/tool/config.yaml`, respecting `XDG_CONFIG_HOME`)
5. Sensible defaults (always present, always documented)

**IMPORTANT**: Every configuration option must be settable via both a flag and an environment variable. Environment variables use the pattern `TOOLNAME_SECTION_KEY`. Document the full hierarchy in help output.

## Anti-Patterns

NEVER build CLIs that exhibit these hostile patterns:

- **Wall of text help**: Dumping every option into an unstructured paragraph. Help must be scannable.
- **Silent failure**: Exiting 0 with no output when something went wrong. If a command did nothing, say so.
- **Chatty defaults**: Printing info-level logs to stdout by default. Quiet by default, verbose on request.
- **Inconsistent flag naming**: Mixing `--dry-run`, `--dryRun`, and `--dryrun` across subcommands. Pick a convention (kebab-case) and enforce it everywhere.
- **Hidden required arguments**: Requiring arguments that are not shown in the usage line or help text.
- **Prompt in a pipeline**: Blocking on interactive input when stdin is not a TTY. This breaks every script and CI job.
- **Unparseable output**: Mixing human prose and machine data in stdout. Use stderr for commentary.
- **Missing exit codes**: Exiting 0 on failure. Scripts depend on exit codes. Lie to them and they break silently.
- **Reinventing conventions**: Custom flag syntax (`/flag`, `flag:value`) instead of POSIX `--flag value` or `--flag=value`. Surprising behavior for `-rf` vs `-r -f`.
- **Ignoring signals**: Not handling SIGINT/SIGTERM, leaving temp files, zombie processes, or corrupted state on Ctrl+C.

Remember: Claude is capable of building extraordinary CLI tools. Do not settle for bare-bones argument parsing with no help text, no error handling, and no attention to the terminal experience. The terminal deserves the same design rigor as any graphical interface. Build tools that people love to use.
