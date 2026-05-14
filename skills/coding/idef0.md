---
name: idef0
description: Create, review, and format IDEF0 models using the IDEFy `.idef0` DSL.
shortcuts: ["idef0", "IDEF0"]
trigger: "When the user asks to create, edit, review, format, or lay out IDEF0 diagrams, IDEF0 schemes, IDEF0 models, or `.idef0` files in the IDEFy DSL."
noTrigger: "Do not use for general process analysis or classical IDEF0 theory unless the user wants an artifact in the IDEFy `.idef0` DSL."
---
# Skill: IDEF0

Create valid IDEF0 models in the IDEFy `.idef0` DSL, including file layout, activity decomposition, arrow notation, tunnels, and canonical formatting.

## Quality Criteria

- The generated model is compiler-oriented: paths, filenames, IDs, interfaces, tunnels, and declarations are internally consistent.
- The generated model is readable as IDEF0: every file describes one activity sheet, with clear boundary arrows and functional blocks.
- Complex models are split into a project tree under `src/idef0/`, not squeezed into one oversized file.
- Formatting follows the canonical style so a formatter or human reviewer can preserve the same structure.
- The output is self-contained: if files cannot be written directly, provide the target tree and complete file contents.

## When Generating

Start from the activity being decomposed. One `.idef0` file describes one decomposition sheet:

```idef0
activity A0 "Activity name" {
    I1 "Input";
    O1 "Output";
    C1 "Control";
    M1 "Mechanism";

    A1 "Functional block" : I1, C1, M1 -> X11[O1];
}
```

For a project, create at least two files in the same project root:

```text
src/idef0/<domain>/<model>/
    A-0.idef0
    A0.idef0
```

`A-0` is the project context, not an activity:

```idef0
context A-0 "Usage context" {
    T1 "Tunnel used somewhere in the project";

    ...A0;
}
```

Use `A-0` only for tunnel declarations and the optional `...A0` reference. Do not decompose `A-0`.

## Project Roots and Paths

The DSL is anchored by a `src/idef0/` scan root. That scan root may appear at any level of a repository or workspace:

```text
src/idef0/coffee/brewing/A0.idef0
packages/samples/src/idef0/coffee/brewing/A0.idef0
```

Both are valid. The prefix before `src/idef0/` is not part of the project name.

Project names are computed from the path after the nearest `src/idef0/` scan root and before the project root:

```text
src/idef0/coffee/brewing/A0.idef0
# project name: coffee.brewing

packages/samples/src/idef0/banking/lending/as_is/A0.idef0
# project name: banking.lending.as_is
```

Rules:

- Every `.idef0` file must live under a `src/idef0/` scan root.
- There must be at least one folder between `src/idef0/` and the project root. `src/idef0/A0.idef0` is invalid.
- Folder names on the path from `src/idef0/` to the project root must be Java-package style:
  - first character: lowercase ASCII letter `a..z`;
  - later characters: lowercase ASCII letters, digits, or `_`;
  - no dots, hyphens, spaces, uppercase letters, or non-ASCII letters.
- The project root is the folder containing `A0.*.idef0`.
- The project root must also contain `A-0.*.idef0`.
- Do not place a second `A0.*.idef0` anywhere inside an existing project root; nested projects are invalid.
- Multiple models in one business area are sibling projects, each with its own `A-0` and `A0`.

## File and Folder Names

File format:

```text
<ID>[.<cosmetic>].idef0
```

Examples:

```text
A0.idef0
A0.Order_intake.idef0
A11.Validate-request.idef0
```

Rules:

- The ID before the first dot must match the header ID inside the file.
- Cosmetic text is optional and ignored by the compiler.
- Cosmetic text may contain letters from any alphabet, digits, `_`, and `-`.
- Cosmetic text must not contain spaces, dots, slashes, or quotes.
- Cosmetic names for folders are allowed only inside the project root, for example `A1.Accept/`; folder names inside the project do not affect compilation.

## Activity and Block IDs

- The root activity of every model is always `A0`.
- Child blocks use suffixes `1..9`, then `A..Z`: `A1`, `A2`, ..., `A9`, `AA`, `AB`, ..., `AZ`.
- Deeper blocks concatenate suffixes: `A11`, `A12`, `A1A`, `A1B`, ...
- Use uppercase Latin letters only in activity IDs.
- Prefer no more than 9 functional blocks in one activity. 10..35 blocks are allowed but should be treated as a warning-level smell. More than 36 blocks is invalid.
- Separate models never use `B0`, `C0`, etc.; create sibling projects with their own `A0`.

Only create a separate `.idef0` file for a functional block when that block is decomposed. Atomic activities exist only as blocks in the parent file.

## Arrow IDs and Roles

Arrow ID prefixes define arrow type:

| Prefix | Meaning |
|---|---|
| `I*` | Input boundary arrow |
| `O*` | Output boundary arrow |
| `C*` | Control boundary arrow |
| `M*` | Mechanism boundary arrow |
| `X*` | Internal arrow on one activity sheet |
| `T*` | Tunnel declared in `A-0` |

Boundary arrows are declared in the activity header, one per line:

```idef0
activity A0 "Brew coffee" {
    I1 "Roasted coffee beans";
    O1 "Cup of coffee";
    C1 "Recipe";
    M1 "Barista";

    A1 "Grind beans" : I1, C1, M1 -> X11 "Ground coffee";
}
```

Every declaration ends with `;`. Use double-quoted strings. Escape inner double quotes as `\"`.

## Functional Blocks

Each functional block is one logical declaration:

```idef0
A1 "Block name" : <consumed arrows> -> <produced arrows>;
```

Consumed arrows on the left of `->`:

- Parent boundary inputs, controls, and mechanisms are consumed by bare ID: `I1`, `C1`, `M1`.
- Internal arrows and tunnels must declare the role they play for this block:
  - `I[X11]`, `C[X11]`, `M[X11]`
  - `I[T1]`, `C[T1]`, `M[T1]`
- Do not consume `O*` directly. A parent output is reached by producing an internal `X*` mapped to `O*`.

Produced arrows on the right of `->` always use an `X*` ID:

- `X11 "Description"` creates an internal arrow on this sheet.
- `X11[O1]` maps the internal arrow to parent output `O1`; name and description come from `O1`.
- `X11[T1]` tunnels the internal arrow through project tunnel `T1`.

Always produce an `X*` arrow, even if no block consumes it yet. This keeps later refactoring local.

## Interface Consistency Between Levels

When block `A1` in parent activity `A0` is decomposed into `A1.*.idef0`, the child file header must match the parent block interface:

- Inputs consumed by `A1` become `I*` boundary arrows in the child file.
- Controls consumed by `A1` become `C*` boundary arrows in the child file.
- Mechanisms consumed by `A1` become `M*` boundary arrows in the child file.
- Parent outputs produced by `A1` through `X*[O*]` become `O*` boundary arrows in the child file.
- The child file has its own internal `X*` namespace.

Before writing a child file, derive its boundary arrows from the parent declaration and check that the IDs and meanings agree.

## Tunnels

- Declare every tunnel exactly once in `A-0`.
- Every declared tunnel must be used somewhere in the project.
- On one activity sheet, the same tunnel cannot be both produced and consumed; use an internal `X*` arrow for same-sheet flow.
- A tunnel may be consumed by multiple blocks on the same sheet, potentially in different roles.
- A tunnel may play different roles on different sheets.
- The role is not fixed in the `A-0` declaration.

## Canonical Formatting

Boundary arrows:

- One arrow declaration per line.
- Exactly one blank line between boundary arrows and functional blocks.

Functional blocks:

- Sort blocks by ID inside each activity.
- Sort consumed arrows by role: inputs first, then controls, then mechanisms.
- Sort arrows inside each role. Bare IDs come before bracketed references:
  `I1, I2, I[T1], I[X11], I[X22]`.
- Never split an activity ID, arrow ID, or string literal across lines.

### Mode 1: One Line

Use for short declarations. Adjacent Mode 1 blocks have no blank lines between them.

Align `:` and `->` across Mode 1 blocks in the file, and keep visual columns for `I`, `C`, and `M` groups:

```idef0
    A1 "Search"     : I1,     C2, C3, C[T2], M1, M3 -> X11 "Candidates";
    A2 "Interview"  : I[X11], C3,            M2, M4 -> X21 "Assessments";
    A3 "Offer"      : I[X21], C4, C[T1],     M1, M2 -> X31 "Offer", X32[O2];
    A4 "Onboarding" : I[X31], C1, C2,        M1, M2 -> X41[O1];
```

### Mode 2: Three Lines

Use when the Mode 1 line would exceed 120 characters, but both sides are still reasonably short.

Put one blank line between Mode 2 or Mode 3 blocks and neighboring blocks:

```idef0
    A13 "Define process inputs and outputs"
        : I[X112], I[X113], I[X114], C1
        -> X131 "Input/output specification";
```

### Mode 3: Multiline Produced Arrows

Use Mode 3 when:

- Mode 2 is already needed because the declaration is long; or
- the produced side contains more than one new described arrow such as `X111 "..."`.

```idef0
    A11 "Analyze process project"
        : I1, C1
        ->
            X111 "Set process name",
            X112 "Set input",
            X113 "Set output",
            X114 "Kind";
```

If a Mode 2 or Mode 3 consumed-arrow line would exceed 120 characters, keep `:` on its own line and put each role group on a separate line:

```idef0
    A15 "Coordinate complex activity"
        :
            I1, I2, I[X111], I[X112], I[X113], I[X114],
            C1, C2, C[X121], C[X131],
            M1, M2, M3
        -> X151[O1];
```

## Layout for Complex Models

Inside a project root, subfolders are for humans; the compiler recursively scans all `.idef0` files and ignores folder structure.

Choose the simplest readable layout:

```text
# Flat layout for small projects
src/idef0/coffee/brewing/
    A-0.idef0
    A0.idef0
    A1.idef0
    A11.idef0
    A2.idef0
```

```text
# Decomposition-oriented layout for larger projects
src/idef0/coffee/brewing/
    A-0.idef0
    A0.idef0
    A0/
        A1.idef0
        A1/
            A11.idef0
        A2.idef0
```

```text
# Business-phase layout when it helps navigation
src/idef0/coffee/brewing/
    A-0.idef0
    A0.idef0
    intake/
        A1.idef0
        A11.idef0
    processing/
        A2.idef0
        A3.idef0
```

All three layouts compile the same if file IDs and contents are consistent.

## Generation Workflow

1. Identify the target `src/idef0/` scan root. If the user gives a nested path like `packages/samples/src/idef0`, use it as valid.
2. Choose a Java-package-style project path under that scan root.
3. Create `A-0.*.idef0` and `A0.*.idef0` in the project root.
4. Decompose `A0` into 3..9 clear functional blocks when possible.
5. For each block that needs decomposition, create one matching child `.idef0` file.
6. Keep atomic blocks fileless.
7. Use tunnels only for cross-sheet/contextual flows that should not be parent outputs.
8. Run the consistency checklist before presenting the result.

## Consistency Checklist

Before finalizing:

- Every generated file is under a `src/idef0/` scan root.
- The project root is at least one folder below `src/idef0/`.
- The project root contains both `A-0.*.idef0` and `A0.*.idef0`.
- There is no nested `A0.*.idef0` inside the project.
- Every filename ID matches the `activity` or `context` ID inside the file.
- Every functional block ends with `;`.
- Boundary declarations are one per line and end with `;`.
- Blocks are sorted by ID.
- Consumed `X*` and `T*` arrows have explicit role prefixes.
- Produced arrows always use `X*`.
- Child activity interfaces match their parent block declarations.
- Every `T*` is declared once in `A-0` and used somewhere.
- No tunnel is both consumed and produced on the same sheet.
- No duplicate activity file ID exists inside the project.
- No orphan `.idef0` file exists outside a project root.

## Complete Minimal Example

```text
packages/samples/src/idef0/coffee/brewing/
    A-0.idef0
    A0.idef0
```

```idef0
# packages/samples/src/idef0/coffee/brewing/A-0.idef0

context A-0 "Coffee brewing context" {
    T1 "Office noise";

    ...A0;
}
```

```idef0
# packages/samples/src/idef0/coffee/brewing/A0.idef0

activity A0 "Brew coffee" {
    I1 "Roasted coffee beans";
    I2 "Filtered water";
    O1 "Cup of coffee";
    O2 "Spent grounds";
    C1 "Recipe";
    C2 "Target TDS";
    M1 "Barista";
    M2 "Grinder";
    M3 "Temperature-controlled kettle";
    M4 "Dripper and filter";

    A1 "Grind beans"   : I1,     C1,     M1, M2 -> X11 "Ground coffee", X12[T1];
    A2 "Prepare water" : I2,     C1,     M3     -> X21 "Water at target temperature";
    A3 "Extract coffee": I[X11], C1, C2, M1, M4 -> X31[O1], X32[O2];
}
```

## Anti-patterns

- Treating any folder named `idef0` as valid without the `src/idef0/` anchor.
- Including the path prefix before `src/idef0/` in the project name.
- Putting `A0.idef0` directly in `src/idef0/`.
- Using uppercase, hyphenated, dotted, or non-ASCII folders in the project-name path.
- Creating `B0` or `C0` for additional models.
- Creating files for atomic activities that have no decomposition.
- Consuming `X*` or `T*` without `I[...]`, `C[...]`, or `M[...]`.
- Producing `O*` or `T*` directly instead of mapping from an `X*`.
- Declaring a tunnel in an activity file instead of `A-0`.
- Using subfolder names inside the project as semantic input to the compiler.
- Letting a child file invent an interface that does not match the parent block.
