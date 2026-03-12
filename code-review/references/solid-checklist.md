# SOLID & Code Smells Checklist

## Table of Contents
- [SOLID Principles](#solid-principles)
- [Code Smells](#code-smells)
- [Refactor Heuristics](#refactor-heuristics)

## SOLID Principles

### SRP (Single Responsibility)
- File owns unrelated concerns (e.g., HTTP + DB + domain rules in one file)
- Large class/module with low cohesion or multiple reasons to change
- Functions that orchestrate many unrelated steps
- God objects that know too much about the system
- **Ask**: "What is the single reason this module would change?"

### OCP (Open/Closed)
- Adding new behavior requires editing many switch/if blocks
- Feature growth requires modifying core logic rather than extending
- No plugin/strategy/hook points for variation
- **Ask**: "Can I add a new variant without touching existing code?"

### LSP (Liskov Substitution)
- Subclass checks for concrete type or throws for base method
- Overridden methods weaken preconditions or strengthen postconditions
- Subclass ignores or no-ops parent behavior
- **Ask**: "Can I substitute any subclass without the caller knowing?"

### ISP (Interface Segregation)
- Interfaces with many methods, most unused by implementers
- Callers depend on broad interfaces for narrow needs
- Empty/stub implementations of interface methods
- **Ask**: "Do all implementers use all methods?"

### DIP (Dependency Inversion)
- High-level logic depends on concrete I/O, storage, or network types
- Hard-coded implementations instead of abstractions or injection
- Import chains that couple business logic to infrastructure
- **Ask**: "Can I swap the implementation without changing business logic?"

## Code Smells

| Smell | Signs |
|-------|-------|
| **Long method** | Function > 30 lines, multiple nesting levels |
| **Feature envy** | Method uses more data from another class than its own |
| **Data clumps** | Same group of parameters passed together repeatedly |
| **Primitive obsession** | Using strings/numbers instead of domain types |
| **Shotgun surgery** | One change requires edits across many files |
| **Divergent change** | One file changes for many unrelated reasons |
| **Dead code** | Unreachable or never-called code |
| **Speculative generality** | Abstractions for hypothetical future needs |
| **Magic numbers/strings** | Hardcoded values without named constants |
| **Middle man** | Class that only delegates to another without adding value |

## Refactor Heuristics

1. **Split by responsibility, not by size** — A small file can still violate SRP.
2. **Introduce abstraction only when needed** — Wait for the second use case before extracting.
3. **Keep refactors incremental** — Isolate behavior before moving it.
4. **Preserve behavior first** — Add tests before restructuring.
5. **Name things by intent** — If naming is hard, the abstraction might be wrong.
6. **Prefer composition over inheritance** — Inheritance creates tight coupling.
7. **Make illegal states unrepresentable** — Use types to enforce invariants.
