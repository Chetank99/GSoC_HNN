# INCF GSoC'25 - HNN Core Refactoring

<img width="100px" src="https://upload.wikimedia.org/wikipedia/commons/thumb/0/08/GSoC_logo.svg/1200px-GSoC_logo.svg.png" align="left"/>
<img width="100px" src="https://raw.githubusercontent.com/jonescompneurolab/jones-website/master/images/frontpage/logos/logo-hnn-medium.png" align="right"/>

**Project:** Python API for new cell types

**Organization:** [INCF](https://incf.org/) - [Human Neocortical Neurosolver (HNN)](https://jonescompneurolab.github.io/hnn/)

**Repository:** [jonescompneurolab/hnn-core](https://github.com/jonescompneurolab/hnn-core)

## GSoC Contributor

**Name:** Chetan Kandpal

**Email:** [chetankandpal3@gmail.com](mailto:chetankandpal3@gmail.com)

**GitHub Profile:** [chetank99](https://github.com/chetank99)

**University:** Indian Institute of Technology (IIT), Kanpur, India

## Mentors

- Katharina Duecker ([katduecker](https://github.com/katduecker))
- Dylan Daniels ([dylansdaniels](https://github.com/dylansdaniels))
- Austin E. Soplata ([asoplata](https://github.com/asoplata))

## Project Overview

The Human Neocortical Neurosolver (HNN)-core is a user-friendly Python API for biophysically detailed neural modeling, designed specifically for researchers studying EEG and MEG signals without requiring extensive computational backgrounds. The existing codebase was built with hardcoded assumptions about cell types, locations, and connectivity patterns, severely limiting its extensibility.

This project aimed to **refactor the HNN-core codebase to enable dynamic implementation of neural circuits with customizable cell types**, removing hardcoded assumptions and making the system more flexible for researchers developing new network architectures.

### Key Goals

1. **Eliminate hardcoded cell type assumptions** throughout the codebase
2. **Enable dynamic cell creation and positioning** through flexible APIs
3. **Refactor connectivity and drive logic** to work with arbitrary cell types
4. **Maintain backward compatibility** while improving code architecture
5. **Enhance user experience** for researchers implementing custom neural circuits

## A Note Before We Begin

There are two ways to tell the story of a refactoring project.....

The first way is clean, here's what needed doing, here's what got done, here's the result. Nice linear progress from problem to solution. That's the "Work Accomplished" section below, it's what goes in the report, what you'd present at a meeting, what makes sense in hindsight.

The second way is how it actually happened, the code that looked simple until you touched it, the "quick fix" that broke seventeen other things, the bug that made you question your understanding of the entire system. That's the messy bits I mention towards the ends, the debugging, the moments where you realize your mental model was completely wrong. The moments you realize, simple clean approaches often require the most amount of cognitive resources.

Both stories are true. Since this file goes into the GSoC archives and if you're reading this because you're about to refactor some legacy code for GSoC (or anywhere really), the second story might be more useful. Because refactoring isn't just about changing code, it's about repeatedly discovering that your current understanding is insufficient, rebuilding your mental model, and then discovering it's still insufficient at the next “*layer*” down.

The clean story shows where you ended up. The messy story shows how thinking actually works when you're trying to untangle years of accumulated assumptions in a codebase. You start confident, hit a wall, realize the wall is actually part of a larger maze, map the maze, discover the maze has levels, and somewhere along the way you're investigating why changing a cell name makes dipoles disappear!

This is normal. This is the work. My naive 2 cents.

So, here's both versions! the accomplished and the archaeological. May your refactoring adventures be fun and bug-free!

## Work Accomplished

The chunks were implemented in dependency order (work done across multiple branches during the GSoC coding period), starting with the most independent (Chunk 1, cell positioning) and progressing toward the most dependent (Chunk 3, Cell Factory), which relies on all previous chunks being complete. Chunk 3 (Cell Factory) was deferred to post-GSoC scope as it builds upon the architectural foundations established by the completed chunks.

### Chunk 1: Dynamic Cell Positioning (**Merged**)

**PR:** [#1095 - Introducing layer_dict for cell coordinates](https://github.com/jonescompneurolab/hnn-core/pull/1095)

**What was accomplished:**

- **Replaced `pos_dict` with anatomically-based `layer_dict`** system for more structured cell coordinate definition
- **Introduced anatomical layer keys** like 'L5_bottom', 'L2_bottom', 'L5_mid', and 'L2_mid' instead of generic cell type keys
- **Refactored `_create_cell_coords()` in `network.py`** to return the new `layer_dict` structure
- **Updated `Network` class constructor** to accept `pos_dict` and `cell_types` as arguments during initialization
- **Modified `jones_2009_model()` in `network_models.py`** to use the new coordinate system
- **Maintained backward compatibility** while enabling custom cell arrangements

**Main Changes:**

- Moved cell positioning functions like `_calc_pyramidal_coord` and `_calc_basket_coord` into the new framework
- Enabled the `Network` constructor to accept custom `cell_types` and `pos_dict` parameters
- Set up foundation for future PRs to add custom cell types without hardcoded position assumptions

This change was the first step toward flexible network creation, moving away from hardcoded cell positioning to a system that can accommodate custom network architectures.

### Chunk 1.5: Short-to-Long Name Mapping (**Merged**)

**PR:** [# 1103](https://github.com/jonescompneurolab/hnn-core/pull/1103)

**What was accomplished:**

- **Standardized cell naming conventions** across the entire codebase
- **Implemented consistent mapping** between short names (e.g., "L5Pyr") and long names (e.g., "L5_pyramidal")
- **Resolved naming conflicts** that were blocking the metadata refactoring work

This smaller but crucial chunk resolved naming inconsistencies that were preventing the larger architectural changes. It established the foundation for consistent cell type handling throughout the refactoring process.

### Chunk 2: Metadata-Driven Architecture (**Merged**)

**PR:** [#1099 - Chetan's Phase2/Chunk2 upgrades to celltype metadata](https://github.com/jonescompneurolab/hnn-core/pull/1099)

**What was accomplished:**

- **Implemented cell metadata system** to replace hardcoded cell type assumptions
- **Standardized cell naming conventions** by enforcing consistent use of long-form names (e.g., "L5_pyramidal" instead of "L5Pyr")
- **Refactored connection logic** to use metadata-driven approach instead of hardcoded type checks
- **Updated dipole calculation and plotting functions** to work with dynamic cell types
- **Modified test suite** to work with the new metadata-based system
- **Introduced `measure_dipole` metadata attribute** for cells to specify their contribution to dipole measurements

**Main Changes:**

- **Cell Type Metadata Structure:** Each cell type now includes metadata including morphology, connectivity rules, and functional properties
- **Dynamic Connection Establishment:** Connections are now established based on cell metadata rather than string matching on cell type names
- **Flexible Dipole Measurement:** Cells specify whether they contribute to dipole measurements through metadata rather than hardcoded layer assumptions
- **Extensible Parameter System:** New cell types can be added by defining their metadata without modifying core connection or measurement code

**Areas Refactored:**

- `network.py`: Connection establishment logic
- `cell_response.py`: Dynamic cell type handling
- `viz.py`: Plotting functions for cell morphology
- Test modules: Updated to work with metadata-driven system

**Impact:** This enables HNN-core to support arbitrary cell types through metadata configuration rather than requiring code changes for each new cell type.

### Chunk 3: Unified Cell Factory (Future Work)

The `CellTypeBuilder` is like a cell factory that lets you design and build neurons with specific properties

Originally, HNN-Core only supported four hardcoded cell types:

- L2_pyramidal (Layer 2 excitatory neurons)
- L5_pyramidal (Layer 5 excitatory neurons)
- L2_basket (Layer 2 inhibitory neurons)
- L5_basket (Layer 5 inhibitory neurons)

[CellTypeBuilder](https://github.com/jonescompneurolab/hnn-core/blob/fb5a70c9d7c42d9f4fcde63a2e61e4291fb1fa81/hnn_core/cells_default.py#L6) allows to

- Create new cell types without modifying core code
- Define custom properties for each cell type
- Reuse cell definitions across different models
- Share cell types with other researchers

### Basic Structure

```python
class CellTypeBuilder:
    @staticmethod
    def create_custom_cell(name, pos=(0, 0, 0), gid=None):
        # 1. Define the shape
        # 2. Define the synapses
        # 3. Define the electrical properties
        # 4. Assemble everything
        # 5. Return a Cell object

```

1. **Create Sections** (the physical stuff)
2. **Define Synapses** (where connections (magic) happen)
3. **Set Electrical Properties**
4. **Define Connectivity**

and to be used via [custom_cell_factory](https://github.com/jonescompneurolab/hnn-core/blob/fb5a70c9d7c42d9f4fcde63a2e61e4291fb1fa81/hnn_core/cells_default.py#L92)

This chunk represents the final step toward a fully flexible cell type system and is planned for post-GSoC development, building on the foundations established in the completed chunks.

## Code Quality and Testing

### Testing Strategy

- **Maintained 100% backward compatibility** through comprehensive testing
- **Added unit tests** for all new functionality

### Post-GSoC Development Plan

- **Complete Chunk 3:** Implement unified cell factory pattern
- **Documentation Enhancement:** Create comprehensive tutorials for custom cell type creation
- **Performance Optimization:** Current code might be regression the code optimization, need to optimize metadata-driven architecture for large-scale simulations (mpi timeout issue)

## Repository Links

- **Main Repository:** [jonescompneurolab/hnn-core](https://github.com/jonescompneurolab/hnn-core)
- **Project Documentation:** [HNN-core API Documentation](https://jonescompneurolab.github.io/hnn-core/)
- **Contributor Fork:** [chetank99/hnn-core](https://github.com/chetank99/hnn-core)

## Pull Requests Summary

| Chunk | Status | PR Link | Description |
| --- | --- | --- | --- |
| **Chunk 1** | ✅ Merged | [#1095](https://github.com/jonescompneurolab/hnn-core/pull/1095) | Introducing layer_dict for cell coordinates |
| **Chunk 1.5** | ✅ Merged | [#1103](https://github.com/jonescompneurolab/hnn-core/pull/1103) | Short-to-long name mapping standardization |
| **Chunk 2** | ✅ Merged | [#1099](https://github.com/jonescompneurolab/hnn-core/pull/1099) | Metadata-driven connections and drives |
| **Chunk 3** | 🚀 Post-GSoC | [*Proof of concept*](https://github.com/jonescompneurolab/hnn-core/blob/fb5a70c9d7c42d9f4fcde63a2e61e4291fb1fa81/hnn_core/cells_default.py#L6https://github.com/jonescompneurolab/hnn-core/blob/fb5a70c9d7c42d9f4fcde63a2e61e4291fb1fa81/hnn_core/cells_default.py#L6) | Unified cell factory pattern |

---

## ***Now comes the really messy(read: fun) bits!!!***

---

## What I thought to be a Simple Refactoring exercise became a Deep Surgery

### The Initial Plan vs Reality

Started out thinking this would be straightforward. The task: make cell types flexible instead of hardcoded.

For context, HNN simulates the electrical activity of brain cells to help researchers understand EEG and MEG signals. These signals come from the "dipole moments", essentially the summed electrical activity of pyramidal neurons in the cortex. The software models four types of brain cells: excitatory pyramidal neurons and inhibitory basket cells, each in two cortical layers (L2 and L5).

A quick search showed these four cell types mentioned in various places. Should be simple to make this flexible, right? Just add the ability to define new cell types?

Turns out the entire codebase assumed there would always be exactly these four cell types with exactly these names: `L2_pyramidal`, `L5_pyramidal`, `L2_basket`, and `L5_basket`.

### The Parameter Dictionary Maze

When I tried adding a test cell type to verify flexibility, it crashed immediately. The function that creates pyramidal neurons wasn't extensible, it was just a hardcoded if-else statement that only accepted two specific names.

Then came the naming consistency challenge. The codebase used both short names (`L2Pyr`) and long names (`L2_pyramidal`) inconsistently. When I tried standardizing to long names, the code started looking for parameters that didn't exist. The parameter system constructed keys by concatenating cell names, so changing the names broke all parameter lookups.

Along with it, NEURON (the underlying neural simulation engine HNN uses) had already registered certain names as identifiers for cell compartments. These couldn't be changed without breaking the entire simulation backend.

### The Test Suite Cascade

After implementing what seemed like a clean refactoring for flexible cell positioning, the test suite exploded with failures. The cryptic error messages gave no hint about the real problem, a single test fixture using the old API was failing, and every test depending on it cascaded into incomprehensible errors.

### The Silent Dipole Bug

This was the most intriguing bug. After refactoring, everything appeared to work, simulations ran, tests passed, plots generated. Except the dipole plots (the main scientific output showing simulated brain signals) were completely flat. No errors, just scientifically meaningless results.

Investigation revealed a naming mismatch: the initialization code stored dipole recordings under one set of names, while the aggregation code looked for them under different names. They never matched, so dipoles were created but never collected.

Even after fixing the names, dipoles remained flat. Deeper investigation showed that pyramidal cells (the only cells that contribute to the dipole signal in HNN) weren't being recognized due to a hardcoded check using the old naming convention. Without proper recognition, these cells weren't getting the biophysical machinery needed to generate dipole signals. The simulation ran perfectly, just without the one thing it was supposed to measure.

### The Cascading Dependencies

Each fix revealed new layers of assumptions. Fixing cell positioning broke initialization. Fixing initialization broke the constructor. Fixing the constructor broke tests. Fixing tests broke parameter lookups. Fixing parameters broke dipole calculations. Fixing dipoles broke visualization.

It was archaeological, every layer excavated revealed more hardcoded assumptions underneath.

The breakthrough came with implementing a metadata system. Instead of checking if a cell type's name contained certain strings, the code could now check properties:

```python
# Old way: if 'L2' in cell_type and 'pyramidal' in cell_type:
# New way: if metadata['layer'] == '2' and metadata['morpho_type'] == 'pyramidal':

```

This meant adding new cell types became about defining properties, not hoping your naming convention matched all the hidden assumptions in the code. But even this required updating every module that touched cell types, the surgery went deep into the codebase's architecture.

### The Lessons

What I overestimated as a simple refactoring exercise (hellooo [Mr David Dunning and Mr Justin Kruger](https://thedecisionlab.com/biases/dunning-kruger-effect)) took a lot of effort because:

1. Cell type names weren't just labels, they were keys into parameter dictionaries, NEURON section identifiers, and test fixtures
2. The four hardcoded cell types were load-bearing assumptions throughout the codebase
3. Silent failures (flat dipoles) were way worse than crashes
4. One broken fixture could cascade into dozens of unrelated-looking errors
5. You can't refactor a tightly coupled system all at once, need incremental steps (CHUNKS!) with validation (TESTS!) at each stage

The "simple" task of making cell types flexible required reverse-engineering the entire simulation pipeline, building a parallel testing framework, and essentially reimplementing core parts of the system while keeping them backward compatible. Every week revealed new layers of complexity that the previous week's solution had broken. I loved each and every single bit!

*This project was completed as part of Google Summer of Code 2025 with the International Neuroinformatics Coordinating Facility (INCF) and the Human Neocortical Neurosolver (HNN) project.*
