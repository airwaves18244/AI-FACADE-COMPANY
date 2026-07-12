# KLAMMER: The Facade Compiler

An advanced compilation engine for **НВФ (navesnye ventiliruemye fasady — ventilated facades)** targeting Autodesk Revit 2024. 

**KLAMMER** automates the generation of facade substructure (POK), cladding layouts, thermal layer composition, and material takeoffs from architectural LOD-200 surfaces.

## Project Structure

```
RevitNvf.sln
 src/
   RevitNvf.Core/        netstandard2.0  — Pure domain logic & algorithms
   RevitNvf.Revit/       net48           — Revit adapter & commands
   RevitNvf.UI/          net48 (WPF)     — Ribbon UI & dockable panels
 tests/
   RevitNvf.Core.Tests/  net8.0          — Unit tests (xUnit)
 docs/
   SPECIFICATION.md      — Technical specification & domain model
   ROADMAP.md           — Development roadmap
   REVIT_MCP_SETUP.md   — Revit MCP configuration
```

## Key Features

- **Deterministic Layout Algorithms**: Pure-function bracket/guide/panel placement with idempotence guarantee
- **Multi-System Support**: Generic `FacadeSystem` config for any cladding type (ceramogranite, metal cassettes, composite)
- **Layer Stackup**: Automatic thermal layer composition (insulation, wind barrier, air gap)
- **Material Takeoff**: Automated ведомости (BOM) for procurement
- **CAD-Agnostic Core**: Domain logic in netstandard2.0 supports Revit, IFC, and other adapters

## Build & Test

**Cross-platform (Core + tests):**
```bash
dotnet test tests/RevitNvf.Core.Tests
```

**Full solution (requires Revit 2024 SDK):**
```
Open RevitNvf.sln in Visual Studio 2022 and build
```

## Architecture

See `docs/SPECIFICATION.md` for the complete technical specification.

See `KLAMMER_*.md` documents for product strategy, roadmap, and architectural decisions.

## References

- **NVF Domain**: `docs/` and `.claude/skills/nvf-domain/`
- **Revit API 2024**: `.claude/skills/revit-api-2024/`
- **Family Generation**: `.claude/skills/revit-family-generation/`

---

**KLAMMER** is the facade compiler for the next generation of architectural automation.
