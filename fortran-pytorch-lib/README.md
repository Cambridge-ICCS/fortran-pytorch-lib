We want to be able to run ML models directly in Fortran. Initially let's assume that the model has been trained in some other language (say Python). We want to run inference on this model without having to call the Python executable. This should be possible by using the existing ML C/C++ interfaces.

# PyTorch

PyTorch provides a C++ API (aka LibTorch) that enables inference and online training of pre-trained models in a production environment with no Python dependency. A model's journey from Python to C++ is enabled by TorchScript, an intermediate representation of a PyTorch model. A model trained in Python is first converted to a TorchScript module via tracing or annotation and then serialised to a file. The serialised model can then be loaded in C++ using the C++ API. For more information, see [here](https://pytorch.org/tutorials/advanced/cpp_export.html).

We have created a minimum C wrapper interface of LibTorch (c_torch.h) that allows us to load and infer a TorchScript model. A Fortran interface (mod_torch) has subsequently been built on top of the C interface. For examples using the C++, C and Fortran interfaces to do inference, see the corresponding ts_inference.* file. A Python script is also provided (p2ts.py) that converts a pre-trained ResNet model to TorchScript and serialises it to a file. This model can be used as the input of the ts_infer_* executables for testing.

## Building

These example programs have the following pre-requisites:

* [Python 3](https://www.python.org/downloads/)
* [PyTorch](https://pytorch.org/)
* [libtorch](https://pytorch.org/cppdocs/installing.html)

To build, do the following in this directory:

    mkdir build
    cd build
    cmake ../.
    make

This will build the separate examples. Next, from this directory, you need
to run the python code to generate the saved model:

    python3 ../pt2ts.py

This will give you some options about how you want to train the model, and
will output a saved model as a .pt file. You should then be able to run the test program (written in Fortran and now compiled) in this directory:

    ./ts_infer_fortran

which will return a result of querying the model, e.g:

    $ ./ts_infer_fortran
    -1.0526 -0.4629 -0.4567 -1.0881 -0.7655
    [ CPUFloatType{1,5} ]

Note that the Fortran example have the model filename hard-coded (e.g.
"annotated_cpu.pt" as the default). Change this and re-compile to address
the other model outputs.

### Summary of files

The starting point for understand the examples is
one of the ts_inference.* files, depending on which
language you wish

* ts_inference.c   - Example of doing inference from C
* ts_inference.cpp - Example of doing inference from C++
* ts_inference.f90 - Example of doing inference from Fortran
* ts_inference.py  - Example of doing inference from Python

* pt2ts.py - Python code to train model

* ctorch.cpp - Wrapper onto PyTorch for C++
* ctorch.h   - Header file for C++ wrapper
* ftorch.f90 - Wrapper onto PyTorch for Fortran

* CMakeLists.txt - Provides the cmake build system configuration


### Troubleshooting

- If `cmake` has a hard time finding your libtorch install you
 can add a line to `CMakeLists.txt` to give a direct location, e.g.,
 add the following before `find_package(Torch REQUIRED)`

     set(Torch_DIR /usr/local/lib/libtorch/share/cmake/Torch)

 where the above assumes `libtorch` has been installed in `/usr/local/lib`.