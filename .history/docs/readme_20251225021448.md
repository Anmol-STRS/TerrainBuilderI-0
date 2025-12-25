# Terrain Builder Documentation

High-performance C++ terrain preprocessing pipeline for large-scale DEM processing.

## Overview

Terrain Builder is a specialized tool for processing and merging Digital Elevation Model (DEM) files into unified terrain grids. Built for GIS analysts and hydraulic modelers working with HEC-RAS, it delivers **5× faster** processing compared to traditional methods.

### Key Features

- **High Performance**: Process 60GB of terrain data in 2-4 hours (vs. 15 hours baseline)
- **Smart Caching**: LRU block cache with 40-60% hit rate reduces disk I/O
- **Memory-Mapped I/O**: Custom GeoTIFF reader with SIMD optimizations
- **Scalable Architecture**: Handles 360+ files with automatic spatial indexing
- **Multi-Format Support**: GeoTIFF, ERDAS IMG, HFA, ENVI, and more

### Performance Metrics

| Metric | HEC-RAS Baseline | Terrain Builder | Improvement |
|--------|------------------|-----------------|-------------|
| Processing Time | 15 hours | 3 hours | **5× faster** |
| Throughput | 1.1 MB/s | 6.3 MB/s | **5.7× faster** |
| Memory Usage | Variable | ~622 MB peak | Optimized |
| Cache Hit Rate | N/A | 40-60% | Reduced I/O |

## Documentation

- **[Architecture Overview](ARCHITECTURE.md)** - System design and components
- **[Architecture Diagrams](ARCHITECTURE_DIAGRAMS.md)** - Visual system documentation
- **[Performance Guide](PERFORMANCE.md)** - Benchmarks and optimization details
- **[Technical Specifications](TECHNICAL_SPECS.md)** - Detailed technical information

## Quick Example

```cpp
// Process 360 DEM files into a unified terrain grid
TerrainInputCatalog catalog;
catalog.scan_files("/path/to/dems");

TargetGrid grid;
grid.build_from_catalog(catalog);

TerrainPipeline pipeline;
pipeline.run(catalog, config);
// Output: terrain_unified.tif (60GB processed in ~3 hours)
```

## Use Cases

- **Hydraulic Modeling**: Preprocessing terrain for HEC-RAS simulations
- **GIS Analysis**: Creating unified terrain grids from multiple sources
- **Large-Scale Processing**: Handling hundreds of DEM files efficiently
- **Research**: Fast terrain analysis for environmental studies

## Project Stats

- **60,000+** lines of production C++ code
- **5,302** lines of core processing logic
- **11** core modules with clean separation
- **9,604** chunks processed in parallel
- **256 MB** intelligent cache system

## Technology Stack

- **Language**: Modern C++ (C++11/14/17)
- **Geospatial**: GDAL for format support
- **Optimization**: AVX2 SIMD instructions, memory-mapped I/O
- **Architecture**: Modular pipeline with windowed processing

## Roadmap

### Phase 3: Advanced I/O (In Progress)
- Custom memory-mapped I/O for all formats
- Additional SIMD optimizations
- **Target**: 12.5× speedup (1.2 hours for 60GB)

### Phase 4: GPU Acceleration (Planned)
- CUDA kernels for resampling and mosaicking
- Asynchronous I/O pipeline
- **Target**: 27× speedup (33 minutes for 60GB)

## License

See LICENSE file for details.

## Contact

For questions, issues, or contributions, please use the GitHub issue tracker.

---

**Built for performance. Designed for scale.**
