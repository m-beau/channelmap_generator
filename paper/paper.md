---
title: 'PixelMap: An Application for Flexible Electrode Selection on Neuropixels Probes'
tags:
  - Python
  - neuroscience
  - electrophysiology
  - neuropixels
authors:
  - name: Maxime Beau
    orcid: 0000-0002-8907-6612
    corresponding: true
    affiliation: "1, 2"
  - name: Christian Tabedzki
    orcid: 0000-0001-8409-6094
    affiliation: 1
  - name: Carlos D. Brody
    orcid: 0000-0002-4201-561X
    affiliation: "1, 2"
affiliations:
  - name: Princeton Neuroscience Institute, Princeton University, USA
    index: 1
  - name: Howard Hughes Medical Institute, USA
    index: 2
date: 8 November 2024
bibliography: paper.bib
---

# Abstract

PixelMap is a browser-based application for creating custom channelmaps for Neuropixels probes that respects electrode wiring constraints. Neuropixels probes, widely used for high-density neural recordings, have more physical electrodes than can be used for simultaneous recording because they contain fewer analogue-to-digital converters (ADCs) than data lines. Each ADC is hard-wired to several electrodes, creating complex interdependencies where selecting one electrode makes others unavailable. PixelMap provides an installation-free, browser-based interface for researchers to design arbitrary recording configurations that meet their experimental requirements while satisfying these hardware constraints. The tool generates IMRO (IMec Read Out) files compatible with SpikeGLX, the most common data acquisition software for Neuropixels recordings.

# Statement of need

Neuropixels probes have revolutionised systems neuroscience by enabling simultaneous recordings from hundreds of neurons at spike resolution across multiple brain regions at any depth [@jun2017; @beau2021; @steinmetz2021; @bondy2024; @ye2025; @beau2025]. However, configuring these probes for successful recording of neural data presents challenges. Neuropixels probes fit 960 to 5120 electrodes but can only record from 384 or 1536 channels simultaneously (see Table 1 for hardware specifics), limited by the number of integrated analogue-to-digital converters (ADCs). The electrode-to-ADC wiring map follows complex patterns that vary with each Neuropixels version, making manual channel selection error-prone and time-consuming.


While existing tools like SpikeGLX and Open Ephys provide ways to edit channelmaps as `.imro` files, they require desktop apps and do not easily allow selection of fully arbitrary electrode geometries. Researchers often need custom channel configurations to target specific brain regions or optimise spatial sampling, but creating these configurations manually requires a deep understanding of the probe's wiring architecture and careful verification to avoid wiring violations.

PixelMap addresses these needs by:

1. **Being available on any machine installation-free**: The tool is available as a web application at [https://pixelmap.pni.princeton.edu](https://pixelmap.pni.princeton.edu) as a Python package.
2. **Visualising wiring constraints interactively**: When users select electrodes, the interface immediately shows which other electrodes become unavailable (marked in black) due to shared ADC lines, preventing invalid configurations.
3. **Supporting arbitrary electrode geometries**: Users can select electrodes through (i) picking from presets for common geometries, (ii) textually entering electrode ranges, enabling repeatable selection, (iii) dragging selection boxes and clicking on the probe visualisation itself, and (iv) loading pre-existing `.imro` files. These four selection methods are intercompatible so that they can be used together. For instance, a SpikeGLX `.imro` file can be loaded as a starting point, and selection boxes used to further refine the channelmap geometry.

| Probe Version | Physical Channels | Simultaneously Recordable Channels |
|---------------|-------------------|-------------------------------------|
| Neuropixels 1.0 | 960 | 384 |
| Neuropixels 2.0 (single shank) | 1,280 | 384 |
| Neuropixels 2.0 (4-shank) | 5,120 (1,280 per shank) | 384 |
| Neuropixels 2.0 Quad Base | 5,120 (1,280 per shank) | 1,536 |

**Table 1**: Number of physical and simultaneously addressable electrodes across Neuropixels probe versions.

# Implementation

PixelMap is implemented in Python using HoloViz Panel [@yang2022] for the web interface, providing an interactive and responsive user experience. The software architecture consists of three main components.

First, the **wiring maps** at `./wiring_maps/*.csv` are custom CSV files describing the electrode-to-ADC mappings for each supported probe type. They were built from files provided by IMEC (Neuropixels manufacturer).

Second, the **core logic** at `./backend.py` implements the constraint-checking algorithms that validate electrode selections against probe-specific wiring maps. This handles the complex mapping between physical electrodes and ADC channels for different probe types (Neuropixels 1.0, 2.0 single-shank, and 2.0 four-shank so far). Hash tables (Python dictionaries) are used to query incompatible electrode pairs with O(1) complexity and improve performance.

Finally, the **graphical user interface** at `./gui/gui.py` was built with HoloViz Panel. The interface provides real-time visualisation of the probe layout with electrode colour-coded based on their selection state (available in grey, selected in red, or unavailable in black). The interface supports the abovementioned four selection modes, including Bokeh-based interactive click-selection and box-selection to select or deselect electrodes. User interactions trigger immediate recalculation of available electrodes based on the current selection state. This design ensures users receive instant feedback about constraint violations, preventing invalid configurations before file generation.

![PixelMap's browser-based graphical user interface.\
**Center:** Main panel featuring the probe's physical layout with one or four shanks that exhibit the 960 (Neuropixels 1.0) or 1,280 (Neuropixels 2.0) physical electrodes/shank to be selected. Electrodes available for selection are light grey, selected electrodes turn red, and electrodes that become unavailable due to hardware wiring constraints turn black. In this example, 384 electrodes have been selected (matching the maximum simultaneous recording capacity), with a distributed pattern across multiple banks, illustrating that PixelMap allows selection of arbitrary channelmap geometries.\
**Left:** panel to input probe metadata (also part of `.imro` files) as well as three methods of electrode selection: preset geometries, manual textual input of electrode ranges, and pre-loading an existing `.imro` file. These three methods of electrode selection can be mixed together with an interactive click-and-drag box selector and deselector.\
**Right:** electrode status indicator that turns green to confirm the selection is complete and is ready for IMRO file generation. Users can export their configuration via the "Download IMRO" button for direct use in SpikeGLX or save a PDF visualisation to easily remember the geometry of the corresponding `.imro` file in the future. Below the status indicator are PixelMap's instructions.](Figure1.png)

# Installation and Usage

PixelMap can be used through:

1. **Web application**: Available at [https://pixelmap.pni.princeton.edu](https://pixelmap.pni.princeton.edu) for immediate use without installation.
2. **Local installation**: Via pip (`pip install .`) or uv (`uv run cmap_gui`) from the cloned GitHub repository.
3. **Docker container**: Users can download the image used for the website and run the container locally.
4. **Programmatic API**: Python scripts can directly call `generate_imro_channelmap()` for batch processing or integration into analysis pipelines.

For more details, see the project repository at [https://github.com/m-beau/channelmap_generator](https://github.com/m-beau/channelmap_generator).

The software includes an automated test suite with 41 tests covering hardware constraint validation, all preset configurations, IMRO file generation for all supported probe types, and end-to-end workflows. Tests run automatically via GitHub Actions continuous integration on every code change, ensuring software reliability. See the repository's `tests/` directory for details.

# Author Contributions

|                          | Maxime Beau | Christian Tabedzki | Carlos D. Brody |
|--------------------------|:-----------:|:------------------:|:------------:|
| Conceptualisation        |      X      |                    |              |
| Backend and GUI          |      X      |                    |              |
| App hosting              |             |         X          |              |
| Supervision and funding  |             |                    |      X       |


# Acknowledgements

We thank Julie Fabre for discussions and designing PixelMap's logo. We also thank Jesse C. Kaminsky, Jorge Yanar, and members of the Brody laboratory for testing and feedback during development, and PNI IT members Garrett McGrath and Gary Lyons for their advice concerning hosting. Finally, we thank the Princeton Neuroscience Institute for hosting the web application. This work was supported by Howard Hughes Medical Institute and the National Institutes of Health.

# References
