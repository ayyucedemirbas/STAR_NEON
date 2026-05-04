STAR 2.7.11b
==========
Spliced Transcripts Alignment to a Reference
© Alexander Dobin, 2009-2024
https://www.ncbi.nlm.nih.gov/pubmed/23104886

AUTHOR/SUPPORT
==============
Alex Dobin, dobin@cshl.edu </br>
https://github.com/alexdobin/STAR/issues </br>
https://groups.google.com/d/forum/rna-star

Ayyuce Demirbas, AyseAyyuceDemirbas [at] my [dot] unt [dot] edu

APPLE SILICON NEON FORK
=======================
This version includes native Apple Silicon/ARM64 NEON support for Opal and is maintained at:
https://github.com/ayyucedemirbas/STAR_NEON

HARDWARE/SOFTWARE REQUIREMENTS
==============================
  * x86-64 compatible processors or ARM64 processors with NEON, including Apple Silicon
  * 64 bit Linux or Mac OS X


DIRECTORY CONTENTS
==================
  * source: all source files required for compilation
  * bin: pre-compiled executables for Linux and Mac OS X
  * doc: documentation
  * extras: miscellaneous files and scripts

COMPILING FROM SOURCE
=====================



```bash
# Get latest STAR source from releases
wget https://github.com/alexdobin/STAR/archive/2.7.11b.tar.gz
tar -xzf 2.7.11b.tar.gz
cd STAR-2.7.11b

# Alternatively, get STAR source using git
git clone https://github.com/alexdobin/STAR.git

# Or get this Apple Silicon NEON-enabled fork
git clone https://github.com/ayyucedemirbas/STAR_NEON.git
```

Compile under Linux
-------------------

```bash
# Compile
cd STAR/source
make STAR
```
For processors that do not support AVX extensions, specify the target SIMD architecture, e.g.
```
make STAR CXXFLAGS_SIMD=-msse4.1
```
On ARM64/aarch64 systems such as Apple Silicon, STAR uses a native 128-bit NEON backend for Opal and `CXXFLAGS_SIMD` defaults to an empty value.


Compile under Mac OS X
----------------------

```bash
# 1. Install brew (http://brew.sh/)
# 2. Install gcc with brew:
$ brew install gcc
# 3. Build STAR:
# run 'make' in the source directory
# note that the path to c++ executable has to be adjusted to its current version
$cd source
$make STARforMacStatic CXX=/usr/local/Cellar/gcc/8.2.0/bin/g++-8
# 4. Make it availible through the terminal
$cp STAR /usr/local/bin
```

For Apple Silicon, use the ARM64 Homebrew prefix and build without x86 SIMD flags:
```bash
brew install gcc
cd source
make STARforMacStatic CXX=/opt/homebrew/bin/g++-15 -j4
```

The Apple Silicon Opal backend is native NEON, but it is 128-bit wide, so it processes fewer lanes per vector than AVX2's 256-bit path. It should avoid SIMDe's 256-bit emulation overhead on Apple Silicon, but the real speedup should be benchmarked on your workload.

To verify that the build is using the native ARM64/NEON path:
```bash
./STAR --version
file STAR opal/opal.o
/opt/homebrew/bin/g++-15 -dM -E -x c++ /dev/null | grep __ARM_NEON
otool -tvV opal/opal.o | grep -E 'sqadd|sqsub|smin|smax|umin|umax|cmgt' | head
```

Expected output should include the STAR version, `Mach-O 64-bit executable arm64`
for `STAR`, `Mach-O 64-bit object arm64` for `opal/opal.o`, `#define __ARM_NEON 1`,
and NEON instructions such as `sqadd.16b`, `sqsub.16b`, `smin.8h`, `smax.4s`,
or `cmgt.16b`.

All platforms - non-standard gcc
--------------------------------

If g++ compiler (true g++, not Clang sym-link) is not on the path, you will need to tell `make` where to find it:
```bash
cd source
make STARforMacStatic CXX=/path/to/gcc
```

If employing STAR only on a single machine or a homogeneously setup cluster, you may aim at helping the compiler to optimize in way that is tailored to your platform. The flags LDFLAGSextra and CXXFLAGSextra are appended to the default optimizations specified in source/Makefile.
```
# platform-specific optimization for gcc/g++
make CXXFLAGSextra=-march=native
# together with link-time optimization
make LDFLAGSextra=-flto CXXFLAGSextra="-flto -march=native"
```

FreeBSD ports
=============

STAR can be installed on FreeBSD via the FreeBSD ports system.
To install via the binary package, simply run:
```
pkg install star
```

LIMITATIONS
===========
This release was tested with the default parameters for human and mouse genomes.
Mammal genomes require at least 16GB of RAM, ideally 32GB.
Please contact the author for a list of recommended parameters for much larger or much smaller genomes.


FUNDING
=======
The development of STAR is supported by the National Human Genome Research Institute of
the National Institutes of Health under Award Number R01HG009318.
The content is solely the responsibility of the authors and does not necessarily represent the official views of the National Institutes of Health.
