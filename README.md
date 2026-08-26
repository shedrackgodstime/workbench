# workbench

Technical workspace for Shedrack Godstime — projects, explorations, research notes, and the narrative behind the work.

## Structure

```
projects/         Built things: software, tools, crates, prototypes
explorations/     Investigated things: research, notes, protocol studies
```

Each entry is a folder with a `README.md` that carries the full public narrative.

## Portfolio Integration

The portfolio site reads `README.md` files from this repository at build time. See `portfolio-contract.md` for the frontmatter schema and writing guidelines.

```
projects/irosh/README.md          → /workbench/irosh
explorations/nat-traversal/README.md → /workbench/nat-traversal
```

## Writing

Entries should stand on their own. Write the summary, context, technical details, and lessons learned in the `README.md`. Supporting files (notes, references, diagrams) can exist alongside it but are not read by the portfolio.
