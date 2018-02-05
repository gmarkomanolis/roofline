# Getting started with Intel Advisor 2018 roofline model
Instructions on how to prepare a roofline model with Intel advisor 2018 on Cray-xc40

For this test case I will use NAS Benchmarks (LU). Moreover, I use Shaheen II supercomputer, a Cray-XC40 at KAUST Supercomputing Laboratory. Adjust the paths and the executable name accordingly.

1. We load the appropriate modules (it depends on the system) 

```
module swap PrgEnv-cray/5.2.82 PrgEnv-intel
module load advisor/2018.1.1.535164 
module swap intel/15.0.2.164 intel/17.4.4.196
```
2. We need to compile our application with debug mode and dynamic compilation

For example 
ftn -g -dynamic ...

All the submission and config files are included in the [roofline](https://github.com/gmarkomanolis/roofline)

3. Using the Intel advisor

### MPI application

With Cray MPI is better to use Intel advisor on one process, we will use the [multi-prog](https://slurm.schedmd.com/srun.html) feature

The executable is called for example LU.C.16, we need 16 MPI processes, create a file called [config_initial.txt](https://github.com/gmarkomanolis/roofline/blob/master/config_initial.txt) with the following:

```
0 advixe-cl -v -collect survey -project-dir=/path_to_project/ -- ./executable
1-15 ./executable
```

This means that the Intel advisor will be used on the first rank only, declare the appropriate path and the name of the executable

* Execute:
 
```
sbatch submit_initial.sh
```
On our system there are some errors at the end, but be sure that the execution of the application is finished without issues, then the errors are coming from some libraries on our system not related to the studied application.

* In order to gather information for the flops, execute:

```
sbatch submit_flops.sh
```

where the [config_flops.txt](https://github.com/gmarkomanolis/roofline/blob/master/config_flops.txt) is:

```
0 advixe-cl -collect tripcounts -flop -project-dir=/path_to_project/ -- ./lu.C.16
1-15 ./lu.C.16
```
If everything worked as expected, you have a folder called e000


* Optional step, Use Intel advisor to gather information about data dependencies for the loops that are not vectorized because of data dependencies. Be careful this phase take significant time to finish the execution
```
sbatch submit_dependencies.sh
```
where the [config_dependencies.txt](https://github.com/gmarkomanolis/roofline/blob/master/config_dependencies.txt) is:

```
0 advixe-cl -collect dependencies -track-stack-variables -no-filter-reductions -no-filter-by-scope -stop-after=0 -- ./lu.C.16
1-15 ./lu.C.16
```

* Now open the GUI
```
advixe-gui /path_to_project/ &
```

### GUI

* This is the initial GUY while you execute the above command

![alt text](/tutorial/roofline_initial.png)

* If you click on the project e000 then you will see on your right the following data with generic overview of the efficiency

![alt text](/tutorial/summary.png)

* If you click on the tab "Survey & Roofline" you can see data per loop and identify potential bottlenecks.

![alt text](/tutorial/survey_roofline.png)

* If you click on the Roofline menu (left arrow on the above screenshot), you get the roofline model below. We select the checkbox that the arrows points as we use only one MPI process for the epxeriments and the initial roofline peak results are about full node. 

![alt text](/tutorial/roofline_model.png)

* If you click the tab "Why No Vectorization?" below the roofline model, you can find some tips to improve the code

![alt text](/tutorial/recommendations.png)

