# Flatiron Institute Nonuniform Fast Fourier Transform libraries: FINUFFT

Version 0.7  (3/8/2017)

### Alex H. Barnett and Jeremy F. Magland

### Purpose

This is a lightweight library to compute the nonuniform FFT to a specified precision, in one, two, or three dimensions.
This task is to approximate various exponential sums involving large numbers of terms and output indices, in close to linear time.
The speedup over naive evaluation of the sums is similar to that achieved by the FFT. For instance, for _N_ terms and _N_ output indices, the computation time is _O_(_N_ log _N_) as opposed to the naive _O_(_N_<sup>2</sup>).
For convenience, we conform to the simple existing interfaces of the
[CMCL NUFFT libraries of Greengard--Lee from 2004](http://www.cims.nyu.edu/cmcl/nufft/nufft.html).
Our main innovations are: speed (enhanced by a new functional form for the spreading kernel), computation via a single call (there is no "plan" or pre-storing of kernel matrices), the efficient use of multi-core architectures, and simplicity of the codes, installation, and interface.
In particular, in the single-core setting we are approximately 8x faster than the (single-core) CMCL library when requesting many digits in 3D.
Preliminary tests suggest that in the multi-core setting we are around 20% faster than the run time of the [Chemnitz NFFT](https://www-user.tu-chemnitz.de/~potts/nfft/) for the 3D type 1 transform, at comparable accuracy, except that our code does not require an additional plan or precomputation phase.

See the manual for more information.

### Dependencies

For the basic libraries

- C\++ compiler such as g\++
- GNU make
- FFTW3
- Optionally, OpenMP (however, the makefile can be adjusted for single-threaded operation)

For the Fortran wrappers

- Fortran compiler such as gfortran (see settings in the makefile)

On a Fedora/CentOS linux system, these dependencies can be installed as follows:
```bash
sudo yum install make gcc gcc-c++ gcc-gfortran fftw3 fftw3-devel libgomp
```
On Ubuntu linux:
```bash
sudo apt-get install make build-essential libfftw3-dev gfortran
```

### Installation

- Clone using git (or checkout using svn, or download as a zip -- see green button above)
- Copy makefile.dist to makefile, then possibly edit the latter for your system
- Compile the library using:

```bash
make
```
This will compile the static library `src/libfinufft.a` which you may now link to from C/C\++, or Fortran. In your C/C\++ code you will need to include the header `src/finufft.h`.
To run a suite of tests and make sure your installation worked:

```bash
make test
```
To run multi-threaded and single-threaded performance tests:

```bash
make perftest
```

Other useful tasks include:

```bash
make test1d # small accuracy test for components in 1D. Analogously for 2D, 3D  
make spreadtestnd # benchmark the spreader routines, all dimensions  
make examples/testutils # test various low-level utilities  
make fortran # compile and test the fortran interfaces  
```

### Contents of this package

- `src` : main library source and headers.  
- `examples` : test codes (drivers) which verify libaries are working correctly, perform speed tests, and show how to call them. 
- `examples/nuffttestnd.sh` : benchmark and display accuracy for all types and dimensions (3x3 = 9 in total) of NUFFT at fixed requested tolerance  
- `examples/checkallaccs.sh [dim]` : sweep over all tolerances checking the spreader and NUFFT at a single dimension;  [dim] is 1, 2, or 3
- `examples/results` : accuracy and timing outputs.  
- `contrib` : 3rd-party code.  
- `fortran` : wrappers and drivers for Fortran.
- `matlab` : wrappers and examples for MATLAB. (Not yet working)  
- `devel` : various obsolete or in-development codes (experts only)  
- `doc` : the manual (not yet there)  
- `README.md`
- `LICENSE`
- `makefile.dist` : GNU makefile (user should first copy to `makefile`)  

### Notes

C\++ is used for all main libraries, although without much object-oriented code. C\++ complex<double> ("dcomplex") and FFTW complex types are mixed within the library, since it is a glorified driver for FFTW, but has dcomplex interfaces and test codes. FFTW was considered universal and essential enough to be a dependency for the whole package.

As a spreading kernel function, we use an unpublished simplification of the Kaiser--Bessel kernel, which at high requested precisions achieves roughly half the kernel width achievable by a truncated Gaussian. Our kernel is of the form exp(-beta.sqrt(1-(2x/W)^2)), where W = nspread is the full kernel width in grid units. This (and Kaiser--Bessel) are good approximations to the prolate spheroidal wavefunction of order zero (PSWF), being the functions of given support [-W/2,W/2] whose Fourier transform has minimal L2 norm outside a symmetric interval. The PSWF frequency parameter (see [ORZ]) is c = pi.(1-1/2R).W where R is the upsampling parameter (currently R=2.0).

References for this include:

[ORZ] Prolate Spheroidal Wave Functions of Order Zero: Mathematical Tools for Bandlimited Approximation.  A. Osipov, V. Rokhlin, and H. Xiao. Springer (2013).

[KK] Chapter 7. System Analysis By Digital Computer. F. Kuo and J. F. Kaiser. Wiley (1967).

[FS] Nonuniform fast Fourier transforms using min-max interpolation.
J. A. Fessler and B. P. Sutton. IEEE Trans. Sig. Proc., 51(2):560-74, (Feb. 2003)

This code builds upon the CMCL NUFFT, and the Fortran wrappers duplicate its interfaces. For this the following are references:

[GL] Accelerating the Nonuniform Fast Fourier Transform. L. Greengard and J.-Y. Lee. SIAM Review 46, 443 (2004).

[LG] The type 3 nonuniform FFT and its applications. J.-Y. Lee and L. Greengard. J. Comput. Phys. 206, 1 (2005).

The original NUFFT analysis using truncated Gaussians is:

[DR] Fast Fourier Transforms for Nonequispaced data. A. Dutt and V. Rokhlin. SIAM J. Sci. Comput. 14, 1368 (1993). 

Our distribution includes code by:

Nick Hale and John Burkardt - Gauss-Legendre nodes and weights  
Leslie Greengard and June-Yub Lee - some fortran test codes from CMCL  



### To do

* installation instructions on various linux flavors
* MAC OSX test, put in makefile
* nf1 (etc) size check before alloc, exit gracefully if exceeds RAM?
* test non-openmp compile
* make common.cpp shuffle routines dcomplex interface and native dcomplex arith (remove a bunch of 2* in indexing, and have no fftw_complex refs in them. However, need first to make sure using complex divide isn't slower than real divide used now). Fix the calling from finufft?d?
* theory work on exp(sqrt) being close to PSWF
* figure out why bottom out ~ 1e-10 err for big arrays in 1d. unavoidable roundoff? small arrays get to 1e-14.
* Checkerboard per-thread grid cuboids, compare speed in 2d and 3d against current 1d slicing.
* decide to cut down intermediate copies of input data eg xj -> xp -> xjscal -> xk2 to save RAM in large problems?
* single-prec compile option for RAM-intensive problems?
* test BIGINT -> long long slows any array access down, or spreading? allows I/O sizes (M, N1*N2*N3) > 2^31. Note June-Yub int*8 in nufft-1.3.x slowed things by factor 2-3.
* matlab wrappers, mcwrap issue w/ openmp, mex, and subdirs. Ship mex executables for linux, osx, etc.
* matlab wrappers need ier output?
* python wrappers
* license file
* outreach, alert Dan Foreman-Mackey re https://github.com/dfm/python-nufft
* doc/manual
* boilerplate stuff as in CMCL page
* clean up tree, remove devel and unused contrib
* Flatiron logo

### Done

* efficient modulo in spreader, done by conditionals
* removed data-zeroing bug in t-II spreader, slowness of large arrays in t-I.
* clean dir tree
* spreader dir=1,2 math tests in 3d, then nd.
* Jeremy's request re only computing kernel vals needed (actually was vital for efficiency in dir=1 openmp version), Ie fix KB ker eval in spreader so doesn't wdo 3d fill when 1 or 2 will do.
* spreader removed modulo altogether in favor of ifs
* OpenMP spreader, all dims
* multidim spreader test, command line args and bash driver
* cnufft->finufft names, except spreader still called cnufft
* make ier report accuracy out of range, malloc size errors, etc
* moved wrappers to own directories so the basic lib is clean
* fortran wrapper added ier argument
* types 1,2 in all dims, using 1d kernel for all dims.
* fix twopispread so doesn't create dummy ky,kz, and fix sort so doesn't ever access unused ky,kz dims.
* cleaner spread and nufft test scripts
* build universal ndim Fourier coeff copiers in C and use for finufft
* makefile opts and compiler directives to link against FFTW.
* t-I, t-II convergence params test: R=M/N and KB params
* overall scale factor understand in KB
* check J's bessel10 approx is ok.
* meas speed of I_0 for KB kernel eval
* understand origin of dfftpack (netlib fftpack is real*4)
* [spreader: make compute_sort_indices sensible for 1d and 2d. not needed]
* next235even for nf's
* switched pre/post-amp correction from DFT of kernel to F series (FT) of kernel, more accurate
* Gauss-Legendre quadrature for direct eval of kernel FT, openmp since cexp slow
* optimize q (# G-L nodes) for kernel FT eval on reg and irreg grids (common.cpp). Needs q a bit bigger than like (2-3x the PTR, when 1.57x is expected). Why?
* type 3 segfault in dumb case of nj=1 (SX product = 0). By keeping gam>1/S
* optimize that phi(z) kernel support is only +-(nspread-1)/2, so w/ prob 1 you only use nspread-1 pts in the support. Could gain several % speed for same acc.
* new simpler kernel entirely
* cleaned up set_nf calls and removed params from within core libs
* test isign=-1
* type 3 in 2d, 3d
* style: headers should only include other headers needed to compile the .h; all other headers go in .cpp, even if that involves repetition I guess.
* changed library interface and twopispread to dcomplex
* fortran wrappers (rmdir greengard_work, merge needed into fortran)
