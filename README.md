# Roofline model
Instructions on how to prepare a roofline model with Intel advisor 2018 on Cray-xc40

For this test case I will use NAS Benchmarks (LU). Moreover, I use Shaheen II supercomputer, a Cray-XC40 at KAUST Supercomputing Laboratory.

1) We load the appropriate modules (it depends on the system)

module swap PrgEnv-cray/5.2.82 PrgEnv-intel

module load advisor/2018.1.1.535164 

module swap intel/15.0.2.164 intel/17.4.4.196

2) We need to compile our application with debug mode and dynamic compilation

For example 
ftn -g -dynamic ...

3) Using the Intel advisor

## MPI application

With Cray MPI is better to use Intel advisor on one process, we will use the multi-prog feature

The executable is called for example LU.C.16, create a file called config_initial.txt with the following:


