# AWAS Codebase Guide for AI Coding Agents

**AWAS** (Advanced World Airliners Set) is an OpenTTD NewGRF graphics mod written in NML (NewGRF Markup Language). It adds diverse realistic aircraft variants to OpenTTD with multiple airline liveries.

## Architecture Overview

### Core Structure
- **Main entry**: `AWAS.pnml` - Defines GRF metadata, cargo table, and includes all templates and aircraft
- **Build output**: Generated files in `generated/` (AWAS.grf, AWAS.nml, readme.txt)
- **Source organization**: Two-level template system + aircraft-specific files

### Aircraft Definition Pattern
Each aircraft variant follows a consistent structure with exactly 3 files:

1. **Aircraft.pnml** - Main definition file
   - Includes graphics and switch files
   - Contains `item(FEAT_AIRCRAFT, ...)` block with properties: speed, capacity, costs, dates
   - Uses consistent aircraft ID numbering (7001-7098+)
   - Defines graphics switches and cargo subtype handling

2. **Aircraft_graphic.pnml** - Sprite definitions
   - Defines `spriteset()` for base livery
   - Uses `alternative_sprites()` with `ZOOM_LEVEL_NORMAL, BIT_DEPTH_32BPP` for airline liveries
   - Each livery sprite references corresponding PNG file

3. **Aircraft_switch.pnml** - Runtime behavior
   - Maps `cargo_subtype` values (1-13 typical) to specific sprite sets and display names
   - Livery 1 = manufacturer default; 2+ = airlines
   - Each subtype maps to STR_REFIT_LIVERY_* language strings

### Template System
- **Graphics templates** (`src/template/Aircraft_template.pnml`): Sprite coordinate definitions per aircraft type
  - Each aircraft model has unique template (e.g., `tp_B737_800`) with 8 directional frame coordinates
  - Format: `[x, y, width, height, offsetX, offsetY, ANIM]` per direction
- **Purchase templates** (`src/template/purchase.pnml`): Single sprite for buy dialog (always pointing direction)
- **Aircraft template** (`src/template/Aircraft_template.pnml`): Boilerplate for new aircraft files

## Developer Workflows

### Build Process
```bash
make                    # Build GRF (runs nmlc compiler, generates docs)
make clean             # Remove generated/ directory
make bundle_tar        # Create distribution tar
```

**Key dependencies**: `nmlc` (NML compiler), `python3` (for doc generation), `bash`

**Version management**: Version auto-generated from date in Makefile.config (`VERSION = $(shell date +"%Y.%m.%d")`)

### Adding New Aircraft
1. Create folder: `src/Aircraft/{Manufacturer}/{Family}/{Model}/`
2. Create 3 NML files using Aircraft_template.pnml as reference
3. Add `#include "src/Aircraft/.../Model/Model.pnml"` to [Aircraft_list.pnml](src/Aircraft_list.pnml) with next sequential ID comment (e.g., `//7099`)
4. Run `make` to compile and verify syntax

### Adding Airline Liveries
1. Create airline-specific PNG graphic (256-color indexed, TTD palette)
2. Add `spriteset` + `alternative_sprites` pair to Aircraft_graphic.pnml
3. Add cargo_subtype case to switch in Aircraft_switch.pnml (increment case number)
4. Add language string `STR_REFIT_LIVERY_AirlineName` to [lang/english.lng](lang/english.lng) and [lang/korean.lng](lang/korean.lng)

## Key Conventions

### Naming
- Aircraft files: `{Model}.pnml`, `{Model}_graphic.pnml`, `{Model}_switch.pnml`
- Sprite sets: `set_{Model}`, `set_{Model}_{AirlineName}` (all lowercase airline names in set names)
- Language strings: `STR_REFIT_LIVERY_{AirlineName}`, `STR_{Model}_NAME`, `STR_{Family}_FAMILY`
- Switches: `sw_{Model}`, `sw_B{Model}_cargo_subtype_text`, `switch_{Model}_name`

### Aircraft IDs
- Range: 7001-7098+ (sequential assignments in Aircraft_list.pnml)
- Each aircraft gets one unique ID - variants share same ID with different cargo subtypes

### Cargo Subtypes
- 1 = Manufacturer livery (default)
- 2-13+ = Airline liveries (correspond to PNG files and language strings)
- Switch statement handles missing subtypes by returning manufacturer default

### Properties Patterns
- **Speed**: Typically 800-1000 km/h (realistic to aircraft type)
- **Capacity**: e.g., B737-800 = 189 passengers, 19 mail (typical payload)
- **Life**: `model_life: VEHICLE_NEVER_EXPIRES` (common), `vehicle_life: 25-35` years
- **Costs**: `cost_factor: 200-280`, `running_cost_factor: 100-150` (balanced for gameplay)
- **Acceleration**: 20-30 (typical jets)

## Cross-Component Communication

### Language System
- `lang/english.lng` and `lang/korean.lng` contain all user-visible strings
- Strings referenced as `string(STR_NAME)` in NML, indexed by STR_* constants
- Aircraft names: `STR_{Model}_{Variant}_NAME` or `STR_{Model}_NAME`
- Livery labels: `STR_REFIT_LIVERY_{Airline}`

### Sprite Coordinate System
- X-axis: horizontal spacing between directional frames (variable per aircraft)
- Y-axis: vertical stacking of different aircraft models
- PNG files typically contain 8 directions (N, NE, E, SE, S, SW, W, NW)
- Purchase sprite always from specific offset coordinates

## Common Patterns & Anti-patterns

**Do:**
- Follow exact 3-file structure for new aircraft
- Use consistent switch syntax: `switch(FEAT_AIRCRAFT, SELF, switch_name, condition)`
- Keep coordinate precision - test in OpenTTD after changes
- Use `alternative_sprites` only for high-color airline variants (reduces GRF size)
- Comment aircraft IDs inline in Aircraft_list.pnml for tracking

**Don't:**
- Mix graphics and switch logic in same file (separate responsibilities)
- Reuse aircraft IDs (must be unique per variant)
- Hardcode airline names in NML (use language strings for i18n)
- Forget to add language strings for new liveries

## Debugging Tips

- **Compilation errors**: Check NML syntax in Aircraft.pnml, especially brackets and semicolons
- **Sprite not showing**: Verify coordinate offsets in templates match PNG layout; test with `make` → view in OpenTTD
- **Language string missing**: Verify STR_* constant exists in both english.lng and korean.lng (Korean strings currently sparse)
- **Livery not appearing**: Ensure cargo_subtype case matches between switch and graphics, PNG file referenced correctly

## File References

Key files to consult when working on specific tasks:
- Adding aircraft: [Aircraft_template.pnml](src/template/Aircraft_template.pnml), [Aircraft_list.pnml](src/Aircraft_list.pnml)
- Graphics work: [B737_800_graphic.pnml](src/Aircraft/Boeing/B737/B737_800/B737_800_graphic.pnml) (reference example)
- Switches: [B737_800_switch.pnml](src/Aircraft/Boeing/B737/B737_800/B737_800_switch.pnml) (reference example)
- Metadata: [AWAS.pnml](AWAS.pnml), [Makefile.config](Makefile.config)
