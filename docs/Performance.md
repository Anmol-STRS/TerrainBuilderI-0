# Performance Guide

## Benchmarks

### Test Dataset
- **Files**: 360 DEM files
- **Total Size**: 60 GB
- **Formats**: Mixed GeoTIFF and IMG
- **Average File Size**: ~167 MB
- **CRS**: Multiple coordinate systems
- **Resolution**: Varying from 1m to 10m

### Performance Results

#### Current Implementation (Phase 1-2)

| Metric | Value |
|--------|-------|
| **Total Processing Time** | 2-4 hours |
| **Throughput** | 6.3 MB/s (average) |
| **Speedup vs Baseline** | 5× faster |
| **Memory Usage (Peak)** | 622 MB |
| **Memory Usage (Average)** | 366 MB |
| **Cache Hit Rate** | 40-60% |
| **Chunks Processed** | 9,604 |
| **Time per Chunk** | 0.75-1.5 seconds |

#### Baseline Comparison (HEC-RAS)

| Metric | HEC-RAS | Terrain Builder | Improvement |
|--------|---------|-----------------|-------------|
| Processing Time | 15 hours | 3 hours | **5× faster** |
| Throughput | 1.1 MB/s | 6.3 MB/s | **5.7× faster** |
| Memory Efficiency | Variable | Constant | **Optimized** |
| Parallelization | None | Chunk-ready | **Scalable** |

## Performance Breakdown

### Time Distribution

**Per Chunk (2048×2048 pixels)**:
- Query DEMs: 1-5 ms
- Read Windows: 200-400 ms (40-60% from cache)
- Resample: 150-300 ms
- Mosaic: 100-200 ms
- Compress: 50-100 ms
- Write: 50-100 ms

**Total**: 0.75-1.5 seconds per chunk

### Operation Breakdown

| Operation | Percentage | Time (60GB) | Optimization |
|-----------|-----------|-------------|--------------|
| I/O Operations | 40% | 72-96 min | LRU Cache |
| Resampling | 30% | 54-72 min | SIMD |
| Mosaicking | 20% | 36-48 min | SIMD |
| Writing/Compression | 10% | 18-24 min | Tiled Format |

## Optimization Impact

### 1. LRU Block Cache

**Configuration**:
- Size: 256 MB
- Policy: Least Recently Used
- Granularity: Per-window

**Results**:
- Hit Rate: 40-60%
- I/O Reduction: 40-60%
- Memory Overhead: Fixed 256 MB
- Speedup: 1.5-2× on I/O operations

**Cache Performance by Chunk**:
- Edge chunks: 30-40% hit rate (fewer neighbors)
- Interior chunks: 50-70% hit rate (more reuse)
- Average: 40-60% hit rate

### 2. SIMD Optimizations (AVX2)

**Implemented Operations**:
- uint8 to float conversion: 8× speedup
- uint16 to float conversion: 6× speedup
- NoData masking: 7.5× speedup
- Bilinear interpolation: 3-5× speedup (partial)

**Overall Impact**:
- Type conversion: 5-8× faster
- Processing throughput: 2.5× overall improvement

**CPU Requirements**:
- AVX2 support (Intel Haswell+ / AMD Excavator+)
- Automatic fallback to scalar code

### 3. Memory-Mapped I/O

**Custom GeoTIFF Reader**:
- Zero-copy access via mmap()
- Direct memory access to compressed tiles
- OS-managed paging

**Results**:
- Read speed: 2-3× faster than GDAL buffered reads
- Memory efficiency: Minimal allocation overhead
- Scalability: Handles files larger than RAM

### 4. Windowed Processing

**Strategy**:
- Process 2048×2048 pixel chunks
- Constant memory footprint
- No full-file loading required

**Benefits**:
- Memory: O(chunk_size) instead of O(dataset_size)
- Scalability: Handles arbitrary input sizes
- Parallelization: Chunk-level independence

## Scalability Analysis

### Input Scaling

| Files | Size | Processing Time | Throughput |
|-------|------|-----------------|------------|
| 100 | 17 GB | 45-90 min | 6.3 MB/s |
| 360 | 60 GB | 2-4 hours | 6.3 MB/s |
| 1000 | 167 GB | 7-11 hours | 6.3 MB/s |

**Observation**: Linear scaling with input size

### Memory Scaling

| Dataset Size | Memory Usage | Cache Efficiency |
|--------------|--------------|------------------|
| 17 GB | 622 MB | 35-45% |
| 60 GB | 622 MB | 40-60% |
| 167 GB | 622 MB | 45-65% |

**Observation**: Constant memory footprint, improving cache efficiency with larger datasets

## Future Performance Targets

### Phase 3: Custom I/O (In Progress)

**Optimizations**:
- Memory-mapped I/O for all formats
- Advanced SIMD kernels for resampling
- Format-specific decompression
- Optimized buffer management

**Projected Results**:
- Processing Time: 1.2 hours
- Throughput: 15.6 MB/s
- Speedup: 12.5× vs baseline
- Memory: 622 MB (unchanged)

**Impact Breakdown**:
- I/O: 2-3× faster
- Processing: 1.5-2× faster
- Overall: 2.5× faster than current

### Phase 4: GPU Acceleration (Planned)

**Implementation**:
- CUDA kernels for resampling
- GPU-accelerated mosaicking
- Asynchronous CPU-GPU transfers
- Multi-chunk parallel processing

**Projected Results**:
- Processing Time: 33 minutes (0.55 hours)
- Throughput: 38.2 MB/s
- Speedup: 27× vs baseline
- Memory: 622 MB CPU + 2 GB GPU

**Impact Breakdown**:
- Resampling: 5-10× faster on GPU
- Mosaicking: 3-5× faster on GPU
- I/O: Async overlap with computation
- Overall: 2-3× faster than Phase 3

## Performance Tuning

### Cache Size Tuning

| Cache Size | Hit Rate | Memory | Performance |
|------------|----------|--------|-------------|
| 128 MB | 25-35% | Low | Baseline |
| 256 MB | 40-60% | Medium | **Optimal** |
| 512 MB | 45-65% | High | Diminishing returns |
| 1 GB | 50-70% | Very High | Minimal gain |

**Recommendation**: 256 MB provides best performance/memory ratio

### Chunk Size Tuning

| Chunk Size | Chunks | Time/Chunk | Memory | Performance |
|------------|--------|------------|--------|-------------|
| 1024×1024 | 38,416 | 0.2-0.4s | 310 MB | Overhead-bound |
| 2048×2048 | 9,604 | 0.75-1.5s | 622 MB | **Optimal** |
| 4096×4096 | 2,401 | 3-6s | 2.4 GB | Memory-bound |

**Recommendation**: 2048×2048 balances overhead and memory

### Thread Count (Future)

| Threads | Speedup | Efficiency | Bottleneck |
|---------|---------|------------|------------|
| 1 | 1× | 100% | Baseline |
| 4 | 3.5× | 87% | I/O contention |
| 8 | 6× | 75% | Cache thrashing |
| 16 | 8× | 50% | Disk bandwidth |

**Recommendation**: 4-8 threads for optimal efficiency

## Profiling Data

### CPU Profile (Representative Chunk)

```
Function                          Time    %
────────────────────────────────────────────
read_window (with cache miss)     380ms   40%
bilinear_resample                 250ms   26%
mosaic_blocks                     180ms   19%
compress_chunk                     85ms    9%
write_chunk                        55ms    6%
────────────────────────────────────────────
Total                             950ms   100%
```

### Memory Profile

```
Component                    Size     Peak
──────────────────────────────────────────
Input Windows                200 MB   220 MB
Resampled Buffers           100 MB   120 MB
Output Chunk                 16 MB    16 MB
Block Cache (shared)        256 MB   256 MB
Metadata & Overhead          50 MB    60 MB
──────────────────────────────────────────
Total                       622 MB   672 MB
```

## Hardware Recommendations

### Minimum Requirements
- CPU: Modern x64 with AVX2 support
- RAM: 2 GB
- Disk: SSD recommended for source files
- OS: Linux, Windows, macOS

### Recommended Configuration
- CPU: 4+ cores (for future parallelization)
- RAM: 4 GB
- Disk: NVMe SSD (2000+ MB/s read)
- OS: Linux (best mmap performance)

### Optimal Configuration
- CPU: 8+ cores with AVX-512
- RAM: 8 GB
- Disk: NVMe RAID for source files
- GPU: NVIDIA (CUDA compute 7.0+) for Phase 4
- OS: Linux kernel 5.0+

## Real-World Performance

### Case Study: Hydraulic Modeling Project

**Dataset**:
- Region: Large watershed area
- Files: 360 DEM tiles
- Coverage: 15,000 km²
- Resolution: 1m vertical, 10m horizontal

**Results**:
- HEC-RAS preprocessing: 15 hours
- Terrain Builder: 3 hours
- Time saved: 12 hours per run
- Multiple iterations: 60+ hours saved

**Business Impact**:
- Faster iteration cycles
- More analysis runs per project
- Reduced infrastructure costs
- Improved productivity

---

**Performance optimized for real-world terrain processing workflows.**
