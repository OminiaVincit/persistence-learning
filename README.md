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

# Overview

- **[Compilation](#compilation)**
  - [Compiling support for the Fast-Gauss-Transform](#compiling-support-for-the-fast-gauss-transform)
  - [Compiling DIPHA](#compiling-dipha)
- **[Examples](#examples)**
  - [Timing](#timing)
  - [Averaging PSS feature maps](#averaging-pss-feature-maps)
  - [Simple classification with SVMs](#simple-classification-with-svms)

# Compilation

The core of the code is in ```diagram_distance.cpp``` and depends on
DIPHA which is included as a submodule. After you have checked out the
repository via

```bash
git clone https://github.com/rkwitt/persistence-learning.git
```

you can checkout the submodule(s) via

```bash
git init
git update submodule --recursive
```

In case the submodules do not get checked-out properly (or nothing happens),
execute the helper script

```bash
./git-submodule-sync.rb
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
fork (which contains a few small fixes to eliminate compiler warnings) which was
checked-out during the ```git submodule update``` into ```code/figtree```.

Next, compile ```figtree``` to build a *static* library. This is done, since
we will call (in our experiments) ```diagram_distance``` from MATLAB and we
want to avoid having to set the library path. To compile ```figtree``` simply
type

```bash
cd code/figtree
make FIGTREE_LIB_TYPE=static
```

Once this is done, we can configure ```dipha-pss``` to use the library. This is
done by entering the cmake GUI, enabling the ```USE_FGT``` flag and then setting
the correct paths for the ```FIGTREE_INCLUDE``` and ```FIGTREE_LIB``` variable.
In our example, using the command line this would look like (from scratch)

```bash
cd dipha-pss
mkdir build
cd build
cmake .. -DUSE_FGT=ON \
  -DFIGTREE_LIB=../../figtree/lib \
  -DFIGTREE_INCLUDE=../../figtree/include
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

### Timing

We start with a simple timing experiment, where we do not use persistence
diagrams computed from data, but simple create random persistence diagrams
for measuring performance of the kernel.

```MATLAB
cd 'code/matlab';
pl_setup
stat = pl_test_timing('/tmp/test', 50:50:200, 20, 0, 1);
disp(stat);
```

This will run ```diagram_distance``` (1) with and (2) without support for FGT
and output timing results (in seconds) for both variants. In particular, we
start with diagrams of 50 random points and increase this to 200 in steps of
50 points. In every run, 20 such diagrams are created, resulting in 20x20 Gram
matrices. The output is written to ```/tmp/test``` which is created in case it
does not exist.

### Averaging PSS feature maps

In this example, we take a large (random) sample of points from a double annulus,
then draw a couple of *small* random samples from that collection, compute persistence
diagrams (from the small samples) and eventually average the corresponding PSS
feature maps. The full collection of points and three exemplary random samples are
shown in the figure below:

![Input](https://github.com/rkwitt/persistence-learning/blob/master/common/pss_averaging_input.png "Input")

This is a good example to illustrate how to compute persistence diagrams from distance
matrices using DIPHA. In particular, for each random sample, the distance matrix simply is
the *pairwise Euclidean distance* between all the points in each random sample.

The full functionality is implemented in the MATLAB function ```pl_experiment_pss_average.m```.
To produce the results from the *NIPS 2015* paper (see reference above), we additionally provide a
```.mat``` file ```pl_experiment_pss_average_NIPS15.mat``` which you can load
and pass to the script. This sets the configuration (e.g., radii of annuli, seed,
  etc.) we used in the paper. We run the script as follows:

```matlab
cd 'code/matlab';
pl_setup;
out_dir = '/tmp/out';
load ../../data/pl_experiment_pss_average_NIPS15.mat
result = pl_experiment_pss_average(pl_experiment_pss_average_NIPS15, out_dir);
```

This writes all output files to ```/tmp/out``` including (1) the persistence
diagrams, (2) the distance matrices, (3) plots of all the samples, (4) PSS
feature maps and (5) the average PSS feature map. The computed persistence
diagrams are also available as part of the ```result``` structure that is
returned by the script.

### Simple classification with SVMs

In this demonstration, we will create toy data from the annuli as before, however,
this time the objective is to distinguish samples drawn from a single annulus
and samples drawn from a double-annulus based on their persistence diagrams.

First, we create the sample data. We will use ```/tmp/``` as our output
directory.

```matlab
out_dir = '/tmp';
cd code/matlab;
pl_setup;

cnt=1;
for i=1:10
    % create filename
    filename = fullfile(out_dir, sprintf('dmat_%.3d.dipha', cnt));
    % Draw 100 points, center [0, 0], inner radius = 1, outer radius = 2
    points = pl_sample_annulus([0,0], 1, 2, 100, -1)';
    D = squareform(pdist(points));
    save_distance_matrix(D, filename);
    cnt=cnt+1;
end
for i=1:10
    filename = fullfile(out_dir, sprintf('dmat_%.3d.dipha', cnt));
    % Draw 100 points, two centers [0, 1] and [0, -1]
    points = pl_sample_linked_annuli( ...
        100, [0 1], 1, 1.5, [0 -1], 0.5, 1, -1);
    D = squareform(pdist(points));
    save_distance_matrix(D, filename);
    cnt=cnt+1;
end
```

Next, we compute persistence diagrams (by hand), using DIPHA. The inputs are
(as before) the distance matrices we just created from the point samples. We
use a simple bash script (e.g., ```compute.sh```) to do this job. Just set the variable
```DIPHA_BINARY``` to the correct path to the ```dipha``` binary.

```bash
#!/bin/bash
DIPHA_BINARY=<ADD PATH TO DIPHA BINARY HERE>
for i in `seq 1 20`; do
    SRC_FILE=`printf "dmat_%.3d.dipha" ${i}`
    DST_FILE=`printf "dmat_%.3d.pd" ${i}`
    CMD="${DIPHA_BINARY} --upper_dim 2 ${SRC_FILE} ${DST_FILE}"
    ${CMD}
done
```
Move this script to ```/tmp/``` (since this was the output directory in the
MATLAB code) and execute it:

```bash
cd /tmp/
chmod +x compute.sh
./compute.#!/bin/sh
```
We can now use the PSS kernel to compute the Gram matrix, i.e., the matrix
of pairwise kernel evaluations between all created persistence diagrams.
This can be done very easily, since ```diagram_distance``` accepts a ASCII
file as input, where all persistence diagrams are listed (one per line).
In ```/tmp/``` we create such a list via

```bash
cd /tmp/
find . -name 'dmat*.pd' > diagrams.list
```
Finally, we execute ```diagram_distance``` and compute features up
to dimension two (set via ```---dim```). In our example, we compute
the kernel for 1-dimensional features and set the
time parameter of the kernel (set via ```--time```) to 0.1.


```bash
cd code/dipha-pss/build/bin
./diagram_distance --inner_product --time 0.1 --dim 1 /tmp/diagrams.list > /tmp/kernel.txt
```

The kernel matrix, saved as ```/tmp/kernel.txt``` can now be used, e.g., to
train a SVM classifier. We will use libsvm for that purpose, in particular,
the MATLAB interface to libsvm (see the libsvm documentation on how to compile
the MATLAB interface).

```matlab
labels = [ones(10,1);ones(10,1)*2]; % Create labels for training
pos = randsample(1:20,15); % Indices of diagrams used for training
neg = setdiff(1:20,pos);   % Indices of diagrams used for testing
model = svmtrain(labels(pos),[(1:length(pos))' kernel(pos,pos)], '-t 4 -c 1');
[pred,acc,~] = svmpredict(labels(neg), [(1:5)' kernel(neg,pos)], model);
disp(acc);
```
Ideally, we get an accuracy of 100%, simply because the problem is also
very easy. In the demonstrations that follows, we will use more realistic
data. This example just illustrates the *basic* pipeline when we want to
use the kernel in a classification setup.

### Using 3D shapes as input data

In both the *CVPR* and the *NIPS* paper, we experiment with persistence
diagrams, computed from functions on surfaces of 3D shapes. In the following
steps, we demonstrate the full pipeline from 3D shapes, represented as
surface meshes, to persistence diagrams. We also provide the full dataset
of 3D corpus callosum shapes from the *NIPS* paper for research purposes.

#### Additional 3rd party code

For the full pipeline to work, we will need some additional MATLAB
code which will make our life easier when dealing with meshes. In
particular, we will need:

1. [STLRead]((http://www.mathworks.com/matlabcentral/fileexchange/22409-stl-file-reader/content/STLRead/html/stldemo.html})
2. [iso2mesh](http://iso2mesh.sourceforge.net/cgi-bin/index.cgi)



#### Input data

The input data is available in [STL](https://en.wikipedia.org/wiki/STL_(file_format)
format. We will use Eric Johnson's STL file reader to load the files; see STLRead code.

*Importantly, the meshes already are a valid simplicial complex, since they
represent a triangulation of the object's surface. Next, we will compute a
function value for each face of the complex.*

#### Compute function








1. Read STL files
2. Compute HKS
3. Triangulation to DIPHA-compatible complex
4. Run DIPHA
5. Perform classification/hypothesis testing
