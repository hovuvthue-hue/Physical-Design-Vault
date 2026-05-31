# Floorplanning Concept Map from L5-FP_1 and L6-FP_2

## Source basis

This synthesis is grounded primarily in the two uploaded lecture decks. Taken together, they define floorplanning in the broad physical-design sense: boundary definition, I/O planning, macro placement, standard-cell legal-region definition, PDN construction, and floorplan quality checks before placement. They also show that floorplanning is iterative rather than a one-shot task. fileciteturn0file0 fileciteturn0file1

For double-checking, I relied only on official or academic sources: the LEF/DEF language reference for site semantics, OpenROAD documentation for floorplan initialization, pin placement, chip-level pad-ring handling, tapcell/endcap insertion, PDN generation, and flow stage interfaces, plus UC San Diego and UC Berkeley research reports on the role of floorplanning and macro placement in QoR and practical chip planning. citeturn12view0turn12view2turn12view3turn11view0turn18view0turn18view1turn5view5turn13view0

One important caution from the lectures: the aspect-ratio convention is internally inconsistent. One slide states chip aspect ratio as height/width, while another states width/height and interprets “wide” as values greater than 1. OpenROAD’s `initialize_floorplan` documentation defines aspect ratio as height/width. For your cards, do not store `Aspect Ratio = x/y` without also storing the exact convention and tool context. fileciteturn0file0 citeturn12view0

## Scope map

A clean AI-readable partition is this.

**COMMON_BLOCK_FLOORPLAN_SCOPE**: `[[Floorplanning]]`, `[[PhysicalConstraints]]`, macro-instance planning using `[[HardIP]]`, legal standard-cell region definition using `[[PlacementGrid]]`, `[[Site]]`, `[[Row]]`, routing-grid instantiation using `[[RoutingGrid]]`, `[[Track]]`, `[[Pitch]]`, designer blockages and halos, early PDN construction, and floorplan quality checks. This is the core of both lectures. fileciteturn0file0 fileciteturn0file1

**FULL_CHIP_FLOORPLAN_SCOPE**: die sizing against pad/package constraints, I/O pad-ring construction, corner/filler/power-supply cells, ESD-aware I/O planning, bond-wire or bump-array constraints, seal ring, scribe line, CUP and flip-chip/RDL choices. These appear mainly in Lecture 5 backup/full-chip material and are essential if your scope is chip top, but not if you are only doing block-level floorplanning. fileciteturn0file0 citeturn18view1

**LIBRARY_AND_TECH_INPUT_SCOPE**: `[[GateLevelNetlist]]`, `[[SDC]]`, `[[MMMC]]`, `[[LEF]]`, `[[LIB]]`, `[[MetalStack]]`, `[[ITF]]`, `[[CellAbstract]]`, `[[Obstruction]]`, and LEF `[[Pin]]`. These are not floorplanning steps themselves; they are enabling descriptions and constraints consumed by floorplanning. LEF explicitly carries layer, via, placement-site, and macro-cell definitions, and SITE definitions are used to form DEF rows. fileciteturn0file0 citeturn12view3

**DOWNSTREAM_CONSUMER_SCOPE**: `[[Placement]]`, `[[ClockTreeSynthesis]]`, `[[Routing]]`, `[[ParasiticExtraction]]`, `[[STA]]`, and `[[Signoff]]`. Floorplanning constrains and preconditions all of them. OpenROAD’s flow description makes the coupling explicit: floorplan/PDN generation precedes placement, CTS, routing, and extraction, and each later step consumes placed DEF or floorplan state. citeturn5view3turn5view4turn17search5turn19search6

## Core floorplanning concepts

**`[[Floorplanning]]`**  
Essence: the first major physical-design step that converts logical design intent into a physical plan. In these lectures, it includes boundary definition, I/O placement, macro placement, standard-cell-area planning, PDN construction, and quality checks. This is broader than the narrower industry shorthand where “floorplan” sometimes means only die/core size plus macro placement. Flow position: immediately after design import and before placement. Primary output: a floorplan checkpoint/database plus exported physical state such as DEF; the lecture examples explicitly show a database checkpoint (`write_db place_macros.db`), so “DEF only” is too narrow as an output description. fileciteturn0file0 fileciteturn0file1 citeturn17search8

**`[[PhysicalConstraints]]`**  
This should be the umbrella card for boundary and geometric constraints. In the lectures, that umbrella includes die area, core area, I/O ring area, core-to-I/O spacing, aspect ratio, pin-side restrictions, fixed macro locations/orientations, halos, placement blockages, routing blockages, and power-grid geometry. A useful card split under this umbrella is to create new child cards such as `[[DieArea]]`, `[[CoreArea]]`, `[[IORingArea]]`, `[[CoreToIORingSpacing]]`, `[[AspectRatio]]`, `[[PlacementBlockage]]`, `[[RoutingBlockage]]`, and `[[Halo]]`. fileciteturn0file0 fileciteturn0file1

**`[[PhysicalConstraints]]` boundary and sizing semantics**  
For full-chip work, the lecture defines die area as the total chip silicon area, core area as the logic region, I/O ring area as the peripheral band for I/O cells, and core-to-ring spacing as the gap between core and I/O ring. For block-level work, the boundary is inherited from the top level, there are no I/O pads, and the designer must honor top-defined I/O pin locations and shapes. That distinction is fundamental and should be explicit in your `[[Floorplanning]]` card. fileciteturn0file0

**`[[HardIP]]` as macro instances inside `[[Floorplanning]]`**  
The lecture defines macros as fixed-function predesigned blocks treated like hard IP during PnR, including SRAM/ROM/register-file memories, analog blocks such as ADC/DAC/PLL, hard PHY/IP blocks, and some special I/O structures. The floorplanning task is not “what is a macro,” but “where do instantiated macros go, in what orientation, with what keep-out, and with what relation to timing, routing, clocking, power, and interfaces.” Berkeley Hammer explicitly notes that large hard macros usually need manual placement information to achieve good efficiency, and UCSD’s RTL-MP work describes floorplanning as essential to good QoR and especially challenging in macro-dominated designs. fileciteturn0file0 citeturn13view0turn5view5

**`[[HardIP]]` macro-placement factors**  
The lectures break macro placement into seven interacting dimensions: timing closure, power consumption, system interface, clock distribution, routing congestion, power delivery with EM/thermal concerns, and foundry/manufacturing requirements. This is the right abstraction level for your card. Do not reduce macro placement to “place connected macros close together”; that is only the timing-and-wirelength slice of the problem. The full concept is multi-objective and constrained. fileciteturn0file0

**Macro placement rules that deserve explicit card links**  
Timing-driven grouping of communicating macros shortens critical paths. Power-aware grouping shortens high-toggle nets and reduces dynamic-power waste. Interface-aware placement keeps interface macros close to relevant I/O or controller logic. Clock-aware placement keeps clocked macros near sensible clock-distribution regions. Congestion-aware placement avoids tightly packed high-pin macros, notches, and chimneys. Power-aware placement leaves room for straps and local PG access and avoids clustering power-hungry macros. Foundry-aware placement requires safe edge distances, fixed orientations where needed, and insertion of supporting cells such as tap cells and boundary/endcap cells. UCSD’s macro-placement literature and OpenROAD’s hierarchical macro-placement documentation are consistent with exploiting hierarchy, connectivity, and RTL dataflow rather than pure geometry alone. fileciteturn0file0 citeturn19search0turn5view5

**`[[PlacementGrid]]`, `[[Site]]`, and `[[Row]]`**  
These three must be separated cleanly. `[[Site]]` is a library/technology primitive defined in LEF; it gives the placement grid unit for a family of macros or core cells. `[[Row]]` is a design-level object created during floorplanning from repeated sites across the core. The lecture describes placement rows as regularly spaced legal horizontal bands made of legal sites and aligned to the technology’s site grid. The standard-cell placement area is the legal row region where the tool may place cells. OpenROAD’s floorplan initialization and LEF/DEF documentation match this: floorplan initialization makes rows from a specified site, and LEF SITE definitions are used in DEF ROW statements. fileciteturn0file1 citeturn12view0turn12view3

**Row orientation semantics**  
The lecture shows alternating `R0` and `MY` row orientations for standard-cell flipping, while LEF/DEF conventions express legal site/macro matching with orientation sets such as `N`, `FS`, `FN`, and `S` depending on symmetry. For your vault, the key rule is that “row orientation” is not a cosmetic detail; it is part of legal placement because it must match power-rail and well-structure assumptions of the cell library. Store an alias note between tool conventions such as `R0/MY` and LEF/DEF conventions such as `N/FS` where appropriate. fileciteturn0file1 citeturn12view3turn12view1

**`[[RoutingGrid]]`, `[[Track]]`, and `[[Pitch]]`**  
The lecture defines wiring tracks as regularly spaced routing-grid lines, with preferred direction by layer and pitch derived from width-plus-spacing design rules. This is different from the placement grid. The placement grid legalizes instances; the routing grid legalizes wires. OpenROAD also separates these concerns: `initialize_floorplan` / `make_rows` define rows, while `make_tracks` defines routing tracks and allows layer-specific pitch/offset overrides. fileciteturn0file1 citeturn12view1

**`[[PlacementBlockage]]`, `[[RoutingBlockage]]`, and `[[Halo]]`**  
Placement blockages govern where cells or macros may legally go; routing blockages govern where signal routing may not go or must be limited; halos are macro-associated keep-outs that move with the macro. The lecture distinguishes hard, partial, soft, and macro-only placement blockages, and shows routing blockages as layer-specific restricted regions. These are floorplanning-native constraints, not routing-afterthoughts. They are how you enforce routability and margin before placement begins. fileciteturn0file0 fileciteturn0file1

**New card candidates for power planning inside floorplanning**  
Your current card set does not include first-class PDN cards, but the lectures treat them as central. The minimum useful additions are `[[PDN]]`, `[[PowerNets]]`, `[[CoreRing]]`, `[[BlockRing]]`, `[[PowerStripe]]`, `[[IRDrop]]`, and `[[Electromigration]]`. The lecture defines power planning as distributing power from the I/O area to all cells in the core while minimizing IR drop, EM risk, and overdesign. It also defines a hierarchical PG structure: standard-cell rails, core rings, block rings, and stripes/straps across multiple layers. OpenROAD’s PDN generator documentation matches that model: the designer specifies grid policy, stripe widths and spacing, rings, voltage domains, and macro-aware grids, then the tool generates the actual PDN structures. fileciteturn0file1 citeturn11view0

**`[[PowerNets]]` and PG connectivity**  
The lectures explicitly say power/ground nets do not exist in the incoming synthesized netlist and must be created and logically connected to standard-cell PG pins, macro PG pins, and tie cells. This is conceptually important: PG topology is partly “outside the synthesized logic netlist” and is injected during physical design. OpenROAD’s floorplan stage also includes tie-cell insertion and later tapcell/endcap insertion as separate physical-design operations. Do not merge `[[TieCell]]` and `[[TapCell]]`; they solve different problems. fileciteturn0file1 citeturn5view0turn18view0

## Full-chip floorplanning extensions

**Full-chip I/O planning versus block I/O planning**  
The lectures separate full-chip I/O pads from block-level I/O pins. Full-chip I/O pads connect to package/PCB and must obey packaging and ESD rules; block-level I/O pins connect to other blocks and are judged more by timing, congestion, and top-level integration. OpenROAD mirrors this split: the pin placer handles die-boundary pin placement on track grids, while the chip-level pad ring flow handles I/O rings, pads, corner cells, fillers, bumps, and wirebond pads. This should become an explicit distinction in your card graph: a block pin is not the same concept as a pad cell. fileciteturn0file0 citeturn12view2turn18view1

**`[[Pin]]` ambiguity you should resolve in your vault**  
Your existing `[[Pin]]` card under `LEF_Geometry/Cell` is a library-abstract concept: where a cell or macro exposes terminals in LEF. The lectures also use “pins” to mean placed block I/O terminals at the block boundary. Those are related but not identical concepts. A robust split is to keep `[[Pin]]` for LEF/cell geometry and add a new `[[BlockIOPin]]` or `[[IOPinPlacement]]` card for floorplanning-time boundary-pin planning. fileciteturn0file0 citeturn12view2turn12view3

**I/O library content that belongs to full-chip floorplanning**  
The lecture’s full-chip I/O material shows digital I/O buffers, analog I/O cells, power-supply cells, corner cells, filler cells, and special interface standards such as SSTL, HSTL, and LVDS. It also states that power-supply cells support power distribution and ESD protection, and that core and I/O supplies are often separated because I/Os draw large transient currents and may operate at different voltage levels. These are top-level floorplanning concerns, not block-level standard-cell-placement concerns. fileciteturn0file0

**`[[ESDProtection]]`, `[[SealRing]]`, and `[[ScribeLine]]`**  
These are not central to a standard digital block floorplan, but they are full-chip physical-planning concepts that strongly affect pad-ring design and boundary decisions. The lecture describes ESD as a major reliability problem and shows diode clamps and current-limiting resistors as protection paths. It also describes the seal ring as a protective structure around the active die area and the scribe line as the saw street between dies. These deserve separate cards if your scope includes chip-top integration. fileciteturn0file0

**Packaging and pad-configuration constraints**  
The full-chip backup material adds packaging constraints such as no bond-wire crossing, minimum spacing, maximum angle, and maximum wire length. It also introduces CUP and flip-chip with RDL as alternative pad configurations that affect die-size efficiency and signal/power delivery. OpenROAD’s chip-level connection flow similarly includes I/O rows, corner cells, pads, bumps, and redistribution-layer related placement. This is why die sizing is not only an internal core-area problem. fileciteturn0file0 citeturn18view1

**Die-size limitation modes**  
A very useful lecture classification is that die size can be core-limited, pad-limited, block-limited, or package-limited. This is more practically useful than a single die-area formula because it tells you what is actually dominant in your design. It should be stored as a trade-off/diagnosis concept, not buried as an example. fileciteturn0file0

## Adjacent concepts outside floorplanning

**`[[DesignImport]]`, `[[GateLevelNetlist]]`, `[[SDC]]`, and `[[MMMC]]`**  
The lecture’s design-import slide shows the expected upstream state: read netlist, read MMMC, read physical data, initialize power/ground context, and run early analysis checks such as analysis coverage, timing, slack histogram, and timing reports. These are upstream-enabling concepts. `[[GateLevelNetlist]]` provides cell count, connectivity, hierarchy, and macro instances. `[[SDC]]` and `[[MMMC]]` tell floorplanning which paths and clocks matter across modes/corners. They are not floorplanning actions, but without them floorplanning becomes blind geometry. fileciteturn0file0

**`[[LEF]]`, `[[CellAbstract]]`, `[[Obstruction]]`, `[[Pin]]`, `[[LIB]]`, `[[MetalStack]]`, and `[[ITF]]`**  
These are technology/library abstractions that floorplanning consumes. LEF defines layer/via/site/macro abstract information; macro LEFs expose sizes, pins, and obstructions; SITE defines the legal placement primitive; routing layers and pitches emerge from technology information. `[[LIB]]` anchors timing/power characterization, while `[[MetalStack]]` and extraction-rule inputs such as `[[ITF]]` ultimately govern resistance, capacitance, and PDN choices. They belong outside `[[Floorplanning]]`, but the linkage is direct and strong. citeturn12view3turn12view1turn19search6

**`[[DEF]]` as both input and output carrier**  
DEF is not “a floorplanning concept” in the same sense as rows or macro placement, but it is the main design-state exchange carrier across floorplanning, placement, routing, and extraction. The lectures show reading DEF in some contexts and writing floorplan checkpoints/databases in others; OpenROAD’s downstream tools also explicitly consume placed DEF. For your vault, treat `[[DEF]]` as a transport format that carries floorplanning decisions, not as a synonym for floorplanning itself. fileciteturn0file0 fileciteturn0file1 citeturn5view4

**`[[Placement]]`, `[[ClockTreeSynthesis]]`, `[[Routing]]`, `[[ParasiticExtraction]]`, `[[STA]]`, and `[[Signoff]]`**  
These are downstream consumers of the floorplan. Placement needs legal rows, fixed macros, halos, and blockages. CTS inherits the geometric relationship between clock source regions and sinks, so poor macro placement worsens skew and insertion delay risk. Routing inherits channels, tracks, blockages, and PDN occupancy. Extraction inherits final wire shapes and technology context; OpenRCX then extracts resistance and capacitance, which feed timing/power signoff. Floorplanning therefore does not “solve timing,” but it sets the physical possibility space in which timing closure later happens. fileciteturn0file0 fileciteturn0file1 citeturn5view4turn17search5turn19search2turn19search6turn19search7

## Metrics formulas and trade-offs

The lectures provide several sizing formulas and heuristics that are useful as cards, but they should be labeled as heuristics rather than universal laws. For full-chip sizing, one lecture formula is: `Die Area = I/O Ring Area + Core-to-I/O Spacing Area + Core Area`. A more detailed backup heuristic expresses die area in terms of gate count, added CTS/timing/ECO overhead, gate density, I/O area, macro area, and target utilization. For block/core sizing, the central heuristic is `Core Area = Estimated Cell Area / Target Utilization`, where estimated cell area includes overhead for CTS, timing optimization, spare cells, congestion relief, plus losses due to blockages/irregularity. fileciteturn0file0 citeturn12view0

A critical metric distinction in the lecture is between raw core size and meaningful standard-cell utilization. The slide explicitly defines standard-cell utilization as `Total Standard Cell Area / Placeable Standard Cell Area`, and shows that “placeable” space is reduced by macros, padding, blockages, unusable regions, and whitespace policy. This is the right mental model: high core utilization on paper is not the same as healthy placeable utilization after floorplan constraints are applied. fileciteturn0file0

For power integrity, the lecture gives a simple average-power and static-IR-drop model: `Pavg = Pleakage + Pinternal + Pswitching`, `Pswitching = 0.5 × C × V^2 × f × toggle_rate`, and `Static IR Drop = Iavg × R = (Pavg / Vdd) × R`. It separately defines dynamic IR drop as the extra voltage drop caused when many cells switch simultaneously, and visually shows how IR drop in the data path can slow data and worsen setup, while IR drop on a clock branch can delay clock arrival and worsen hold. These are exactly the floorplan-to-timing bridges that should be cross-linked to `[[CellDelay]]`, `[[NetDelay]]`, `[[SetupTime]]`, `[[HoldTime]]`, `[[ClockLatency]]`, and `[[ClockSkew]]`. fileciteturn0file1

The dominant trade-offs in these lectures are stable and worth storing explicitly. Higher target utilization reduces area cost but worsens congestion, detours, and timing margin. Tighter macro clustering can reduce wirelength but may create pin-density hotspots, narrow channels, and worse IR/EM/thermal behavior. A denser PDN reduces IR drop and EM risk but consumes routing resources and can increase die pressure. More generous keep-outs and blockages improve routability and timing robustness but reduce effective placeable area. Full-chip I/O planning trades pad count, package rules, bond-wire or bump geometry, ESD robustness, and die size against each other. fileciteturn0file0 fileciteturn0file1 citeturn11view0

## Debug and report hooks

The lecture-provided report and check hooks are useful enough to deserve explicit linkage in your cards. Before or at floorplan entry, `[[DesignImport]]` is followed by `report_analysis_coverage`, `check_timing`, `report_slack_histogram`, and `report_timing`; that means even early floorplanning is already judged against timing sanity, not just geometry. fileciteturn0file0

For general floorplan validation, the lecture uses `check_floorplan` and classifies checks into six groups: power-grid integrity, I/O pads/pins and boundary cells, macro placement, placement rows and core utilization, blockage planning, and general sanity checks. This taxonomy is strong enough to become the backbone of a `[[FloorplanChecks]]` card. fileciteturn0file1

For PG checks, the lecture explicitly names `check_pg_connectivity`, `check_pg_drc`, `analyze_rail`, and `analyze_power_plan`, and shows early-rail-analysis flows covering missing vias/disconnects, power estimation, static IR drop and EM analysis, dynamic analysis, and iterative optimization. OpenROAD’s PDN generator documentation also exposes `-check_only`, failed-via reporting, ring/stripe definition, and PDN via repair, which matches the same debug intent even if exact command syntax differs by tool. fileciteturn0file1 citeturn11view0

For routability and density, the lecture shows `report_utilization`, `place_design`, `route_early_global`, and `report_congestion -rerun_global_router`, plus visual global-route congestion maps. That is the correct debug philosophy for floorplanning: do not wait for detailed routing and signoff extraction before testing whether rows, channels, macro spacing, and blockages are sane. OpenROAD’s placement documentation is consistent with timing-driven and routability-driven placement behavior being sensitive to estimated RC and congestion. fileciteturn0file1 citeturn19search2turn19search3

## Obsidian-ready link index and limitations

A clean relationship backbone for your vault is this.

`[[DesignImport]] -> [[GateLevelNetlist]], [[MMMC]], [[LEF]], init power/ground context`  
`[[GateLevelNetlist]] + [[SDC]] + [[MMMC]] -> [[Floorplanning]]`  
`[[LEF]] -> [[CellAbstract]], [[Pin]], [[Obstruction]], [[Site]]`  
`[[Site]] -> [[Row]]`  
`[[MetalStack]] -> [[RoutingGrid]], [[Track]], [[Pitch]], [[PDN]]`  
`[[Floorplanning]] -> [[PhysicalConstraints]]`  
`[[PhysicalConstraints]] -> [[DieArea]], [[CoreArea]], [[AspectRatio]], [[PlacementBlockage]], [[RoutingBlockage]], [[Halo]]`  
`[[HardIP]] instance placement -> [[Floorplanning]]`  
`[[Floorplanning]] -> [[Placement]] -> [[ClockTreeSynthesis]] -> [[Routing]] -> [[ParasiticExtraction]] -> [[SPEF]] -> [[STA]] -> [[Signoff]]`  
`[[Floorplanning]] -> [[PDN]] -> [[IRDrop]] -> [[CellDelay]] / [[NetDelay]] / [[Slack]]`  
`[[Floorplanning]] -> [[BlockIOPin]] / [[PadRing]] depending on block versus full-chip scope`  
`[[TapCell]] != [[TieCell]]`  
`[[BoundaryCell]] / [[EndCap]] -> row-edge physical correctness`  
`[[SealRing]] / [[ScribeLine]] -> full-chip reliability and dicing boundary`  fileciteturn0file0 fileciteturn0file1 citeturn12view3turn18view0turn18view1turn5view4

The highest-value new-card candidates not currently visible in your list are: `[[DieArea]]`, `[[CoreArea]]`, `[[IORingArea]]`, `[[CoreToIORingSpacing]]`, `[[AspectRatio]]`, `[[BlockIOPin]]`, `[[IOPad]]`, `[[PadRing]]`, `[[PlacementBlockage]]`, `[[RoutingBlockage]]`, `[[Halo]]`, `[[PDN]]`, `[[PowerNets]]`, `[[CoreRing]]`, `[[BlockRing]]`, `[[PowerStripe]]`, `[[IRDrop]]`, `[[Electromigration]]`, `[[TapCell]]`, `[[TieCell]]`, `[[BoundaryCell]]`, `[[ESDProtection]]`, `[[SealRing]]`, `[[ScribeLine]]`, and, if you care about packaging-aware top-level work, `[[RDL]]` and `[[PackagingConstraints]]`. fileciteturn0file0 fileciteturn0file1

Open questions and limitations: the lectures are strong on conceptual structure but not on exhaustive vendor-specific command semantics, MMMC detail, or final signoff EM modeling. The lecture itself explicitly warns that tool command syntax can vary by version. I also cannot verify proprietary-tool behavior beyond what is shown in the lectures and the cited official/public documentation. For cards that mention commands, store the concept separately from any tool-specific syntax block. fileciteturn0file0 citeturn12view0turn11view0