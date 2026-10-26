# Trento-AS

A real-world Trento map for the original **Assetto Corsa**, starting around
Polo Ferrari in Povo and expanding into the surrounding road network.

## Status

Planning and geographic source assessment. No playable AC track is available yet.
See [the implementation plan](docs/ASSETTO_CORSA_PLAN.md) for source reuse,
road reconstruction, export steps and validation gates.

The first milestone is a short drivable Povo section with continuous road
collision, campus scenery, a start/pit area and validation in Assetto Corsa.

## Development

- Python, QGIS and GDAL for geographic preparation and provenance.
- Blender for metre-scale road and scenery modelling.
- Windows with Assetto Corsa and ksEditor for KN5 export and driving validation.

The related [Povo-CS2](https://github.com/AlbertoLupatin/Povo-CS2) project provides
geographic research and reference generators. This repository maintains the AC
implementation separately. Its assets have not been copied here.

## Sources and licensing

Preserve per-source attribution, licences, coordinates and transformations.
Third-party assets retain their original licences. No blanket project licence
has been selected. This is an unofficial fan project.
