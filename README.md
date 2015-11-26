# Persistence Learning

This code implements the **kernel(s) for persistence diagrams** proposed in the following
two publications. Please use the provided BibTeX entries when citing our work.

```
@inproceedings{Reininghaus14a,
    author    = {R.~Reininghaus and U.~Bauer and S.~Huber and R.~Kwitt},
    title     = {A Stable Multi-scale Kernel for Topological Machine Learning},
    booktitle = {CVPR},
    year      = {2015}}
```
```
@inproceedings{Kwitt15a,
    author    = {R.~Kwitt and S.~Huber and M.~Niethammer and W.~Lin and U.~Bauer},
    title     = {Statistical Topological Data Analysis - A Kernel Perspective},
    booktitle = {NIPS},
    year      = {2015}}
```

# Compilation

The core of the code ```diagram_distance``` depends on DIPHA which is included as
a submodule. After you have checked out the repository to your local harddisk
you can checkout the submodule via

```
git update submodule --recursive
```

Once this has finished, change into the ```code/diagram_distance``` directory
and create a ```build``` directory, then use ```cmake``` or ```ccmake``` to
configure the build process, e.g.,

```bash
cd code/dipha-pss
mkdir build
cd build
cmake ...
make
```

If you want to build ```diagram_distance``` *without* support for the
Fast-Gauss-Transform, simply take the standard settings as they are. In
case you do *not* have MPI support on your machine, you can install, e.g.,
OpenMPI. On MacOS (using *homebrew*) this can simply be done via

```bash
brew install open-mpi
```

### Compiling support for the Fast-Gauss-Transform

To enable support for kernel computation via the (improved) Fast-Gauss-Transform
we need the ```figtree``` library. This can be obtained from the original author
Vlad I. Morariu from [here](ttps://github.com/vmorariu/figtree) or you use our
fork (which contains a few small fixes to eliminate compiler warnings) using

```bash
git clone https://github.com/rkwitt/figtree.git
```

For the sake of argument, lets assume this has been checked out to ```/tmp/figtree```.
Next, compile ```figtree``` (not covered here) and make sure that you have set
the environment paths such that the libraries can be loaded. On MacOS X, using
our setting with ```figtree``` in ```/tmp/figtree```, we would have to set (or
add to our ```.bahsrc``` or ```.zshrc``` depending on what type of shell you
use)

```bash
export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:/tmp/figtree/lib
```

Once this is done, we can
configure ```dipha-pss``` to use the library. This is done by entering the
cmake GUI, enabling the ```USE_FGT``` flag and then setting the correct paths
for the ```FIGTREE_INCLUDE``` and ```FIGTREE_LIB``` variable. In our example,
using the command line this would look like (from scratch)

```bash
cd dipha-pss
mkdir build
cd build
cmake .. -DUSE_FGT=ON \
  -DFIGTREE_LIB=/tmp/figtree/lib \
  -DFIGTREE_INCLUDE=/tmp/figtree/include
make
```

This will compile ```diagram_distance``` and enable the ```--use_fgt```
option.

### Compiling DIPHA

You will need to compile DIPHA (i.e., the submodule that we checked out
earlier) in case you want to compute your own persistence diagrams. DIPHA
also uses ```cmake```, so the process should be fairly simple. The
standard process would look like

```bash
cd code/dipha
mkdir build
cd build
cmake ..
make
```
## Examples

In the following, we show some examples which reproduce some of the results
in the *CVPR* and *NIPS* paper. *Note that these examples use MATLAB code
(also contained in the repository).*

Tbd.
