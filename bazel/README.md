# Bazel Rules Development Skills

Skills for developing custom Bazel rules, toolchains, providers, and aspects.

## Skills Included

### developing-bazel-rules

Use when creating custom Bazel rules for new languages, build systems, or tools.

Covers:
- Rule structure and implementation patterns
- Provider design for dependency propagation
- Efficient use of depsets for transitive data
- Action registration and execution
- Toolchain development for cross-platform support
- Testing Bazel rules with analysistest
- Patterns from production rulesets (rules_go)

## Installation

Add to your Claude Code plugins:

```bash
claude plugins add bazel@dev-skills
```

## Reference Documentation

The skill includes detailed reference files:
- `api-reference.md` - Starlark API quick reference
- `providers.md` - Provider design patterns
- `toolchains.md` - Toolchain development guide
- `testing.md` - Testing Bazel rules
- `go-patterns.md` - Patterns from rules_go

## Resources

- [Bazel Rules API](https://bazel.build/rules/lib)
- [rules_go source](https://github.com/bazel-contrib/rules_go)
- [Bazel examples](https://github.com/bazelbuild/examples/tree/main/rules)
