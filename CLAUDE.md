# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.


## Using Gemini CLI for Large Codebase Analysis

When analyzing large codebases or multiple files that might exceed context limits, use the Gemini CLI with its massive
context window. Use `gemini -p` to leverage Google Gemini's large context capacity.

### File and Directory Inclusion Syntax

Use the `@` syntax to include files and directories in your Gemini prompts. The paths should be relative to WHERE you run the
  gemini command:

#### Examples:

**Single file analysis:**
gemini -p "@src/main.py Explain this file's purpose and structure"

Multiple files:
gemini -p "@package.json @src/index.js Analyze the dependencies used in the code"

Entire directory:
gemini -p "@src/ Summarize the architecture of this codebase"

Multiple directories:
gemini -p "@src/ @tests/ Analyze test coverage for the source code"

Current directory and subdirectories:
gemini -p "@./ Give me an overview of this entire project"

### Or use --all_files flag:
gemini --all_files -p "Analyze the project structure and dependencies"

### Implementation Verification Examples

Check if a feature is implemented:
gemini -p "@src/ @lib/ Has dark mode been implemented in this codebase? Show me the relevant files and functions"

Verify authentication implementation:
gemini -p "@src/ @middleware/ Is JWT authentication implemented? List all auth-related endpoints and middleware"

Check for specific patterns:
gemini -p "@src/ Are there any React hooks that handle WebSocket connections? List them with file paths"

Verify error handling:
gemini -p "@src/ @api/ Is proper error handling implemented for all API endpoints? Show examples of try-catch blocks"

Check for rate limiting:
gemini -p "@backend/ @middleware/ Is rate limiting implemented for the API? Show the implementation details"

Verify caching strategy:
gemini -p "@src/ @lib/ @services/ Is Redis caching implemented? List all cache-related functions and their usage"

Check for specific security measures:
gemini -p "@src/ @api/ Are SQL injection protections implemented? Show how user inputs are sanitized"

Verify test coverage for features:
gemini -p "@src/payment/ @tests/ Is the payment processing module fully tested? List all test cases"

When to Use Gemini CLI

Use gemini -p when:
- Analyzing entire codebases or large directories
- Comparing multiple large files
- Need to understand project-wide patterns or architecture
- Current context window is insufficient for the task
- Working with files totaling more than 100KB
- Verifying if specific features, patterns, or security measures are implemented
- Checking for the presence of certain coding patterns across the entire codebase

Important Notes

- Paths in @ syntax are relative to your current working directory when invoking gemini
- The CLI will include file contents directly in the context
- No need for --yolo flag for read-only analysis
- Use --yolo flag for any requests that need file updates
- Gemini's context window can handle entire codebases that would overflow Claude's context
- When checking implementations, be specific about what you're looking for to get accurate results


## Liquid DSP Library Overview

Liquid DSP is a software-defined radio digital signal processing library written in C. This version has been modified to generate LLVM IR and instrumented bitcode for use in the ISAEGG hardware optimization research framework. The library provides DSP components for software-defined radios with a focus on embedded platforms.

## Development Commands

### Building with LLVM IR Generation

This modified version generates both regular binaries and LLVM IR for hardware optimization research.

**Prerequisites:**
- Set environment variables: `PASS_PATH` (path to instrumentation pass) and `LLVM_DIR` (LLVM installation directory)
- clang-18 compiler
- CMake 3.10+

```bash
# Configure build with LLVM IR generation
export PASS_PATH=/path/to/instrumentation/pass
export LLVM_DIR=/path/to/llvm/installation
mkdir build && cd build
cmake ..

# Build library and generate LLVM IR files
make

# The build generates:
# - Regular object files and library
# - LLVM IR files (.ll) for each module  
# - Bitcode files (.bc) linking module IR files
# - Instrumented versions for profiling
```

### Traditional Build (for reference)

```bash
# Standard CMake build without LLVM IR
mkdir build && cd build
cmake -DBUILD_EXAMPLES=ON -DBUILD_AUTOTESTS=ON -DBUILD_BENCHMARKS=ON ..
make
sudo make install  # Linux only, run sudo ldconfig after install
```

### Testing and Validation

```bash
# Run comprehensive test suite (700,000+ checks)
make xautotest
./xautotest

# Run specific test modules
./xautotest -s fft      # Test FFT functionality
./xautotest -v          # Verbose output
./xautotest -q          # Quiet mode
./xautotest -o results.json  # JSON output

# Run benchmarks
make benchmark
./benchmark -s dotprod_rrrf  # Benchmark specific function
```

### Code Coverage Analysis

```bash
# Build with coverage instrumentation
cmake -DBUILD_AUTOTESTS=ON -DCOVERAGE=ON ..
make xautotest
./xautotest -q -o autotest.json
cd ..
gcovr --filter="src/.*/src/.*.c" --print-summary
```

## Architecture Overview

### Core Modules (src/ directory)

The library is organized into functional modules, each containing:
- `src/`: Implementation files
- `tests/`: Automated test cases  
- `bench/`: Performance benchmarks

**Primary Modules:**
- **dotprod**: Vector dot products with SIMD optimizations (SSE, AVX, NEON)
- **filter**: Digital filters (FIR, IIR, polyphase, resampling)
- **fft**: Fast Fourier Transforms (arbitrary length, mixed-radix)
- **modem**: Digital modulation schemes (PSK, QAM, FSK, GMSK)
- **fec**: Forward Error Correction (Hamming, Reed-Solomon, convolutional)
- **framing**: Packet framing and synchronization
- **agc**: Automatic Gain Control
- **nco**: Numerically Controlled Oscillators

**Supporting Modules:**
- **math**: Mathematical functions (Bessel, gamma, polynomial operations)
- **matrix**: Linear algebra operations
- **random**: Random number generators and distributions
- **utility**: Bit manipulation and array operations
- **vector**: Basic vector operations

### SIMD Architecture Support

The library automatically detects and uses SIMD extensions:
- **x86/x64**: SSE2, SSE3, SSE4, AVX, AVX2, AVX-512
- **ARM**: NEON (ARMv7, ARMv8, Apple Silicon)
- **PowerPC**: AltiVec
- Portable C fallback for unsupported architectures

### Build System Architecture

**CMake Configuration:**
- Modular object libraries for each DSP module
- Automatic SIMD detection and selection
- LLVM IR generation with instrumentation support
- Cross-platform compilation support

**Key Build Options:**
- `BUILD_EXAMPLES`: Compile example programs (200+ examples)
- `BUILD_AUTOTESTS`: Build comprehensive test suite  
- `BUILD_BENCHMARKS`: Performance benchmarking tools
- `ENABLE_SIMD`: Enable SIMD optimizations (default ON)
- `COVERAGE`: Enable code coverage analysis

## LLVM IR Generation Workflow

This modified version adds custom LLVM IR generation:

1. **Module Compilation**: Each DSP module is compiled to LLVM IR using clang-18
2. **Instrumentation**: IR files are instrumented with basic block profiling 
3. **Linking**: Module IR files are linked into single bitcode files
4. **Library Generation**: All modules linked into final library bitcode

**Generated Artifacts:**
```
build/
├── <module>.bc              # Non-instrumented bitcode
├── <module>_instrumented.bc # Instrumented bitcode  
├── <module>_ll/            # Individual .ll files
├── <module>_ll_instrumented/ # Instrumented .ll files
├── liquid.bc               # Complete library bitcode
└── liquid_instrumented.bc  # Complete instrumented library
```

## Examples and Usage Patterns

The `examples/` directory contains 200+ demonstration programs covering:
- Basic DSP operations (filters, transforms, modulation)
- Advanced algorithms (equalization, synchronization)
- Complete communication systems (OFDM, packet radio)

**Running Examples:**
```bash
# Build examples
cmake -DBUILD_EXAMPLES=ON ..
make

# Run specific examples
./examples/fir_filter_example
./examples/modem_example -m qpsk
./examples/fft_example
```

## Key Dependencies

- **Required**: libc, libm (standard C library and math library)
- **Optional**: FFTW3 (faster FFT implementation)
- **Optional**: libfec (additional FEC codes)
- **Development**: CMake, LLVM/Clang toolchain

## Performance Considerations

- SIMD optimizations provide 2-4x speedup for vector operations
- FFTW integration improves FFT performance for large transforms
- Modular design allows selective compilation of required components
- Architecture-specific optimizations for ARM, x86, and PowerPC