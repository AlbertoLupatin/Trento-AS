# Povo to Assetto Corsa: implementation plan

Reviewed 26 October 2026 against AlbertoLupatin/Povo-CS2, local main.
Target: original Assetto Corsa on PC. This is a source assessment and proposed
workflow, not a playable AC conversion. Preserve the existing FPS project.

## Recommended first deliverable

Build a small, daytime, offline practice/free-drive layout around the Polo Ferrari
road frontage on Via Sommarive, with a pit/start area in the surface parking lot
where access and dimensions are verified. Do not invent a circuit by connecting
roads that do not join. Choose safe turnaround locations from mapping and field
photos; use a short closed route only if the real network supports it. Start with
one car and no traffic. Keep vanilla AC compatibility first; CSP lighting, weather,
traffic and extra visual effects are subsequent optional work.

## What the current project actually supplies

| Source | Reuse in AC | Work needed |
| --- | --- | --- |
| `data/raw/terrain/5h666051035_DTM.asc` | Povo ground context, 500 x 500 m, 0.5 m cells | Road surfaces need independent fitting; ground samples do not establish asphalt accuracy |
| `5h665051035_DTM.asc` | Mesiano context | Separate tile, not continuous coverage to Povo; acquire intervening tiles |
| Povo DBM | Roof elevation clues | Not a mesh; distinguish roof, bridge and ground surfaces |
| `data/reference/povo_ortho_2024.*` | Georeferenced alignment/reference | Existing extract is 0.5 m output spacing, not original 10 cm source resolution |
| `tools/asc_to_obj.py` | Engine-independent metre-scale terrain export with provenance | OBJ has no UV/material/collision pipeline; use as a modelling intermediate |
| `tools/build_povo_blockout.py`, `povo_architecture.py` | Building envelopes and facade rhythm | Remove arena boundaries, cover and staging objects; revise visible proportions |
| `tools/build_povo_exterior.py`, `config/povo_exterior.json` | Road/parking hypotheses, bridge and facade context | Rebuild roads as continuous driving meshes; existing strips and junction fans are blockouts |
| Campus/interior generators | Selected visible entrances and garage context | Drop most classrooms, furniture, stairs and lifts from the track scene |
| Photo manifests and walk archives | Facades, bridge, garage, street furniture | Preserve credits and distinguish observed dimensions from assumptions |
| VMAP/VMDL/VMAT/VPK, navigation, entities and radar | Reference only | Replace with AC meshes, shaders, track metadata, timing/pit objects and collision conventions |

README records an older verified Windows package; docs/TODO.md and the latest
WORKLOG entries explicitly leave newer glass, doors and stairs unverified in CS2.
Do not carry the older engine validation claim onto current main or onto AC.

## Coordinates and geometry

Keep authoritative GIS geometry in EPSG:25832 metres, retaining the documented
ETRF2000/ETRS89 lineage and ITALGEO 2005 height datum. Keep the existing origin
E666250, N5103800, H370 for the initial shared scene. Existing local meshes are
X east, Y north, Z up. Never apply the Hammer import scale 39.37007874 to AC.
Use metre-scale Blender objects and one documented export axis transform. AC's
runtime is Y-up; validate the FBX/exporter conversion using a 10 m reference,
an asymmetric east/north marker, upward-facing normals and an actual car spawn.
Do not compensate twice in both Blender and the exporter.

Build road centreline/edges in GIS, then a controlled spline/surface model in
Blender. Explicitly model width, longitudinal grade, crossfall/camber, corner
transitions, kerbs, junction patches and drainage. Fit elevation to trustworthy
road evidence, smoothing survey noise without erasing real grade changes.
The current exterior config labels its roads, heights, ramp and dimensions as
provisional; its 6–7 m widths and short line segments are starting hypotheses.
The bridge's FPS floor adaptation is not evidence of vehicle clearance.

Separate a continuous physical road mesh from visual overlays and surrounding
terrain. Avoid overlapping driveable faces, tiny collision slivers, abrupt normal
changes, cracks and z-fighting. Add detail based on curvature and grade; a uniform
0.5 m terrain grid alone is neither sufficient nor necessary for vehicle physics.
Place ground collision below the engineered road surface, with bridge/underpass
and garage levels handled explicitly rather than draping one surface over all.

## Implementation stages and acceptance gates

1. **Geographic route specification.** Draw the first route and scenery buffer in
   QGIS. Inventory every required raster tile and source date/licence. Verify road
   widths, bends, junction topology, bridge clearance and start/turnaround areas.
   Deliver a route GeoPackage and uncertainty register. Acceptance: continuous
   source coverage and a mapped route, without fabricated joins.
2. **Driving greybox.** Export terrain context; create a clean road and parking
   mesh in Blender. Add simple collision barriers only where intended, bridge and
   building shells. Deliver a master blend plus repeatable exports. Acceptance:
   scale/axis checks and no seams or conflicting road collision surfaces.
3. **First native AC package on Windows.** Use a compatible FBX export and Kunos
   ksEditor to assign supported shaders and create KN5. Confirm exporter and SDK
   availability on the user's installed AC/SDK before pinning versions. Start
   with a single `content/tracks/povo/` layout: KN5, `models.ini` as appropriate,
   `ui/ui_track.json`, preview/outline assets and `data/surfaces.ini`. Set physical
   mesh prefixes and surface keys consistently (for example `1ROAD` with a matching
   ROAD surface), retaining separate nonphysical scenery. Add correctly oriented
   `AC_START_0` and `AC_PIT_0`; add timing gates for a timed layout using the SDK's
   conventions. Acceptance: track loads in original AC, car spawns on the road,
   drives, brakes, resets and returns to pits without falling or collision spikes.
4. **Road physics and presentation.** Tune asphalt/kerb/grass/gravel surface
   definitions; validate off-track behaviour and visual/physical alignment. Drive
   both directions where allowed, cross every seam and junction, test the bridge
   and ramp with different wheelbases and ride heights. Add authored tiling road
   materials, markings, roadside props and optimized campus exteriors. Acceptance:
   stable handling, correct scale and a recorded performance baseline at fixed
   settings on a named PC. Do not set an arbitrary FPS guarantee before measuring.
5. **Optional timed/AI layout.** Add supported timing sectors and boundaries,
   record a valid fast lane and pit lane if AI racing is required, then test actual
   AI sessions. A branching road network or two-way traffic is a separate feature;
   vanilla racing AI does not provide a general city traffic simulation.
6. **Expansion.** First expand the contiguous Povo street network around the
   proven route. Next consider a Povo–Mesiano connection, subject to a verified
   public-road route and complete corridor data. Trento descent or wider hill
   roads are later separate milestones. Package route layouts against shared
   scenery, preserve the origin, tile terrain and vegetation, and profile visible
   object count and draw calls. Use lower-detail distant terrain; inspect seams
   and coordinate precision as extent grows. Acceptance: every new segment meets
   the same driving/physics gate before further expansion.

## Asset and distribution policy

Use PAT terrain with CC BY attribution, the municipal orthophoto under its recorded
CC0 declaration, and original/appropriately licensed photography. Maintain an
asset manifest including original hashes, survey dates, transforms and licences.
Keep OSM-derived data separately identifiable and assess ODbL obligations if used.
University floor plans and mappe-povo material have unresolved redistribution
rights; do not package reference PDFs/photos automatically. The repository has no
blanket licence. Do not reuse Valve assets or Google-extracted imagery/geometry.

## Environment and division of work

Cloud Linux: Python generators/tests, GIS preparation, Blender modelling/import
checks, provenance and static package audits. Installed tools include Python
3.12, Blender 4.3.2, QGIS and GDAL, with Pillow/NumPy/Matplotlib available.
Windows AC workstation: ksEditor/KN5 authoring and actual physics, shader, AI and
performance validation. Neither a successful OBJ import nor the FPS tests proves
that an AC package works. No AC SDK, game installation or KN5 runtime was validated
in this cloud environment.

Create the AC implementation in a separate project area/repository when coding
begins, consuming shared geographic inputs rather than importing the compiled FPS
package. Suggested structure: `sources/`, `gis/`, `blender/`, `tools/`, `config/`,
`build/` and `dist/povo/`. Pin export tools after the first Windows round trip.
The next concrete implementation task is the route GeoPackage and short driving
greybox, followed immediately by a Windows AC loading test before detailed art.
