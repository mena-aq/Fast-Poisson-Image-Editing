# Poisson Image Editing - A Parallel Implementation - Adding Block Red Black and MultiSweeps RedBlack Solver


> Original: Jiayi Weng (jiayiwen), Zixu Chen (zixuc)<br>
> Forked by Menahil Ahmad (23I-0546) and Fatima Sohail (23I-0633) for our Parallel & Distributed Computing Course

[Poisson Image Editing](https://www.cs.jhu.edu/~misha/Fall07/Papers/Perez03.pdf) is a technique that can fuse two images together without producing artifacts. Given a source image and its corresponding mask, as well as a coordination on the target image, the algorithm always yields amazing result.

This project aims to provide a fast poisson image editing algorithm (based on [Jacobi Method](https://en.wikipedia.org/wiki/Jacobi_method)) that can utilize multi-core CPU or GPU to handle a high-resolution image input.

## Installation

### Linux/macOS/Windows

```bash
$ pip install fpie

# or install from source
$ pip install .
```

Supported Python versions: `3.10` to `3.13`.

### Extensions

| Backend                                        | EquSolver          | GridSolver         | Documentation                                                | Dependency for installation                                  |
| ---------------------------------------------- | ------------------ | ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| NumPy                                          | :heavy_check_mark: | :heavy_check_mark: | [docs](https://fpie.readthedocs.io/en/main/backend.html#numpy) | `pip install numpy`                                          |
| [Numba](https://github.com/numba/numba)        | :heavy_check_mark: | :heavy_check_mark: | [docs](https://fpie.readthedocs.io/en/main/backend.html#numba) | `pip install numba`                                          |
| GCC                                            | :heavy_check_mark: | :heavy_check_mark: | [docs](https://fpie.readthedocs.io/en/main/backend.html#gcc) | cmake, gcc                                                   |
| OpenMP                                         | :heavy_check_mark: | :heavy_check_mark: | [docs](https://fpie.readthedocs.io/en/main/backend.html#openmp) | cmake, gcc (on macOS you need to change clang to gcc-11)     |
| CUDA                                           | :heavy_check_mark: | :heavy_check_mark: | [docs](https://fpie.readthedocs.io/en/main/backend.html#cuda) | nvcc                                                         |
| MPI                                            | :heavy_check_mark: | :heavy_check_mark: | [docs](https://fpie.readthedocs.io/en/main/backend.html#mpi) | `pip install mpi4py` and mpicc (on macOS: `brew install open-mpi`) |
| [Taichi](https://github.com/taichi-dev/taichi) | :heavy_check_mark: | :heavy_check_mark: | [docs](https://fpie.readthedocs.io/en/main/backend.html#taichi) | `pip install taichi`                                         |

After installation, you can use `--check-backend` option to verify:

```bash
$ fpie --check-backend
['numpy', 'numba', 'taichi-cpu', 'taichi-gpu', 'gcc', 'openmp', 'mpi', 'cuda']
```

The above output shows all extensions have successfully installed.

## Usage

We have prepared the test suite to run:

```bash
$ cd tests && ./data.py
```

This script will download 8 tests from GitHub, and create 10 images for benchmarking (5 circle, 5 square). To run:

```bash
$ fpie -s test1_src.jpg -m test1_mask.jpg -t test1_tgt.jpg -o result1.jpg -h1 -150 -w1 -50 -n 5000 -g max
$ fpie -s test2_src.png -m test2_mask.png -t test2_tgt.png -o result2.jpg -h1 130 -w1 130 -n 5000 -g src
$ fpie -s test3_src.jpg -m test3_mask.jpg -t test3_tgt.jpg -o result3.jpg -h1 100 -w1 100 -n 5000 -g max
$ fpie -s test4_src.jpg -m test4_mask.jpg -t test4_tgt.jpg -o result4.jpg -h1 100 -w1 100 -n 5000 -g max
$ fpie -s test5_src.jpg -m test5_mask.png -t test5_tgt.jpg -o result5.jpg -h0 -70 -w0 0 -h1 50 -w1 0 -n 5000 -g max
$ fpie -s test6_src.png -m test6_mask.png -t test6_tgt.png -o result6.jpg -h1 50 -w1 0 -n 5000 -g max
$ fpie -s test7_src.jpg -t test7_tgt.jpg -o result7.jpg -h1 50 -w1 30 -n 5000 -g max
$ fpie -s test8_src.jpg -t test8_tgt.jpg -o result8.jpg -h1 90 -w1 90 -n 10000 -g max
```

Here are the results:

| #    | Source image                                                 | Mask image                                                   | Target image                                                 | Result image                                                 |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | ![](https://github.com/Trinkle23897/DIP2018/raw/master/1/image_fusion/test1_src.jpg) | ![](https://github.com/Trinkle23897/DIP2018/raw/master/1/image_fusion/test1_mask.jpg) | ![](https://github.com/Trinkle23897/DIP2018/raw/master/1/image_fusion/test1_target.jpg) | ![](https://fpie.readthedocs.io/en/main/_images/result1.jpg) |
| 2    | ![](https://github.com/Trinkle23897/DIP2018/raw/master/1/image_fusion/test2_src.png) | ![](https://github.com/Trinkle23897/DIP2018/raw/master/1/image_fusion/test2_mask.png) | ![](https://github.com/Trinkle23897/DIP2018/raw/master/1/image_fusion/test2_target.png) | ![](https://fpie.readthedocs.io/en/main/_images/result2.jpg) |
| 3    | ![](https://github.com/cheind/poisson-image-editing/raw/master/etc/images/1/fg.jpg) | ![](https://github.com/cheind/poisson-image-editing/raw/master/etc/images/1/mask.jpg) | ![](https://github.com/cheind/poisson-image-editing/raw/master/etc/images/1/bg.jpg) | ![](https://fpie.readthedocs.io/en/main/_images/result3.jpg) |
| 4    | ![](https://github.com/cheind/poisson-image-editing/raw/master/etc/images/2/fg.jpg) | ![](https://github.com/cheind/poisson-image-editing/raw/master/etc/images/2/mask.jpg) | ![](https://github.com/cheind/poisson-image-editing/raw/master/etc/images/2/bg.jpg) | ![](https://fpie.readthedocs.io/en/main/_images/result4.jpg) |
| 5    | ![](https://github.com/PPPW/poisson-image-editing/raw/master/figs/example1/source1.jpg) | ![](https://github.com/PPPW/poisson-image-editing/raw/master/figs/example1/mask1.png) | ![](https://github.com/PPPW/poisson-image-editing/raw/master/figs/example1/target1.jpg) | ![](https://fpie.readthedocs.io/en/main/_images/result5.jpg) |
| 6    | ![](https://github.com/willemmanuel/poisson-image-editing/raw/master/input/1/source.png) | ![](https://github.com/willemmanuel/poisson-image-editing/raw/master/input/1/mask.png) | ![](https://github.com/willemmanuel/poisson-image-editing/raw/master/input/1/target.png) | ![](https://fpie.readthedocs.io/en/main/_images/result6.jpg) |
| 7    | ![](https://github.com/peihaowang/PoissonImageEditing/raw/master/showcases/case0/src.jpg) | /                                                            | ![](https://github.com/peihaowang/PoissonImageEditing/raw/master/showcases/case0/dst.jpg) | ![](https://fpie.readthedocs.io/en/main/_images/result7.jpg) |
| 8    | ![](https://github.com/peihaowang/PoissonImageEditing/raw/master/showcases/case3/src.jpg) | /                                                            | ![](https://github.com/peihaowang/PoissonImageEditing/raw/master/showcases/case3/dst.jpg) | ![](https://fpie.readthedocs.io/en/main/_images/result8.jpg) |

### GUI

Brb and MsrbSolvers are also supported in GUI

![](https://fpie.readthedocs.io/en/main/_images/gui.png)


### Backend and Solver

We extend upon EquSolver and GridSolver and add:
* BrbSolver
* MsrbSolver

For other usage, please run `fpie -h` or `fpie-gui -h` to see the hint.

## Benchmark Result

### Original Authors:
See [benchmark result](https://fpie.readthedocs.io/en/main/benchmark.html) and [report](https://fpie.readthedocs.io/en/main/report.html#result-and-analysis).

### Brb & Msrb
Check the main directory for reports & slides in brb_msrb_docs/

## Algorithm Detail

We build upon the original repository by implementing 2 solvers:

* BrbSolver: Divides workload into red and black blocks; essentially a hybrid of Equ and Grid Solver
* MsrbSolver: Extends BrbSolver to use multiple (alpha) sweeps per iteration

### EquSolver vs GridSolver

Usage: `--method {equ,grid}`

EquSolver directly constructs the equations ![](https://latex.codecogs.com/svg.latex?(4-A)\vec{x}=\vec{b}) by re-labeling the pixel, and use Jacobi method to get the solution via ![](https://latex.codecogs.com/svg.latex?{\vec{x}'=(A\vec{x}+\vec{b})/4).

GridSolver uses the same Jacobi iteration, however, it keeps the 2D structure of the original image instead of re-labeling the pixel in the mask. It may take some advantage when the mask region covers all of the image, because in this case GridSolver can save 4 read instructions by directly calculating the neighborhood's coordinate.

If the GridSolver's parameter is carefully tuned (`--grid-x` and `--grid-y`), it can always perform better than EquSolver with different backend configuration.

## BrbSolver

Usage: `--method {brb}`

e.g.
```
fpie -s test2_src.png -m test2_mask.png -t test2_tgt.png -o result2.jpg -h1 130 -w1 130 -n 11000 -g src -b openmp --method brb --tile 8 -c 6
```

## MsrbSolver

Usage: `--method {msrb}`

e.g.
```
fpie -s test2_src.png -m test2_mask.png -t test2_tgt.png -o result2.jpg -h1 130 -w1 130 -n 5000 -g src -b openmp --method msrb --tile 8 -c 6
```

