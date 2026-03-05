# Modern Dart Guidelines

This repository contains guidelines for code agents that help them write modern Dart code.

For example, an agent with these guidelines uses records instead of single-use data classes, switch expressions instead of switch statements, sealed classes for closed type hierarchies, and extension types for zero-cost wrappers. It also knows about recent additions like null-aware collection elements from Dart 3.8 and dot shorthand syntax from Dart 3.10.

The guidelines cover the most useful features from Dart 2.12 through Dart 3.10, including null safety, patterns, records, class modifiers, and more. An agent will:

- Detect the project's Dart SDK version from `pubspec.yaml`
- Use language features and library additions available up to and including that version
- Prefer modern idioms over older patterns

## Motivation

All coding agents tend to generate outdated Dart. Two reasons:

1. **Training data lag.** Models don't know about features added after their training cutoff. They can't use dot shorthands (Dart 3.10) if they've never seen them.

2. **Frequency bias.** Even for features the model knows, it often picks older patterns. There's more `if (x != null) x` in the training data than `?x` in collection literals, so that's what comes out.

These guidelines fix both problems by giving the agent an explicit reference.

## Instructions

### [Claude Code](https://claude.com/product/claude-code)

The guidelines are distributed as a Claude Code plugin.

#### Installation

Run the following commands inside a Claude Code session.

1. Add this repository as a marketplace:
```
/plugin marketplace add mfurkanyuceal/dart-modern-guidelines
```

2. Install the plugin:
```
/plugin install modern-dart-guidelines@dart-claude-marketplace
```

#### Usage

The plugin adds the `/use-modern-dart` command. Run it at the start of a session to activate the guidelines:

```
/use-modern-dart
```

The command detects the Dart SDK version from `pubspec.yaml` and tells the agent to use features up to that version:

```
> /use-modern-dart

This project is using Dart 3.6.0, so I'll stick to modern Dart best practices
and freely use language features up to and including this version.
If you'd prefer a different target version, just let me know.
```

After this, any Dart code the agent writes will follow the guidelines.

## License

BSD 3-Clause License. See [LICENSE](LICENSE) for details.
