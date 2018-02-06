# Getting started with Intel Advisor 2018 roofline model

Instructions on how to prepare a roofline model with Intel advisor 2018 on Cray-xc40

For this test case, I will use NAS Benchmarks (LU). Moreover, I use Shaheen II supercomputer, a Cray-XC40 at KAUST Supercomputing Laboratory. Adjust the paths and the executable name accordingly.

1. Connect to the system with X11

```
ssh -X ...
```

2. We load the appropriate modules (it depends on the system) 

```
module swap PrgEnv-cray/5.2.82 PrgEnv-intel
module load advisor/2018.1.1.535164 
module swap intel/15.0.2.164 intel/17.4.4.196
```
3. We need to compile our application with debug mode and dynamic compilation

For example 
ftn -g -dynamic ...

All the submission and config files are included in the [roofline](https://github.com/gmarkomanolis/roofline)

4. Using the Intel advisor

### MPI application

With Cray MPI is better to use Intel advisor on one process, we will use the [multi-prog](https://slurm.schedmd.com/srun.html) feature

* The executable is called for example LU.C.16, we need 16 MPI processes, create a file called [config_initial.txt](https://github.com/gmarkomanolis/roofline/blob/master/config_initial.txt) with the following:

```
0 advixe-cl -v -collect survey -project-dir=/path_to_project/ -- ./executable
1-15 ./executable
```
This means that the Intel advisor will be used on the first rank only, declare the appropriate path and the name of the executable

 * If the execution is too slow then follow some advices:

Change default program tree processing mode (especially for Fortran code):
```
0 advixe-cl -v -collect survey –stackwalk-mode=online –no-stack-stitching -project-dir=/path_to_project/ -- ./executable
1-15 ./executable
```

Disable system and non interesting modules, for example for a module called demo.so:
```
0 advixe-cl -v -collect survey -module-filter-mode=include -module-filter=demo.so -project-dir=/path_to_project/ -- ./executable
1-15 ./executable
```
See: [Intel Advisor overhead](https://software.intel.com/en-us/articles/managing-overhead-of-intel-advisor-analyses)

* Execute:
 
```
sbatch submit_initial.sh
```
On our system, there are some errors at the end, but be sure that the execution of the application is finished without issues, then the errors are coming from some libraries on our system not related to the studied application.

* In order to gather information for the flops, execute:

```
sbatch submit_flops.sh
```

where the [config_flops.txt](https://github.com/gmarkomanolis/roofline/blob/master/config_flops.txt) contains this:

```
0 advixe-cl -collect tripcounts -flop -project-dir=/path_to_project/ -- ./lu.C.16
1-15 ./lu.C.16
```
If everything worked as expected, you have a folder called e000

If the execution time is too slow, you could disable the tricount or apply some techniques:

Disable tripcount:
```
0 advixe-cl -collect -flop -no-trip-counts -project-dir=/path_to_project/ -- ./lu.C.16
1-15 ./lu.C.16
```

Select loops to profile:
```
0 advixe-cl -collect tripcounts -flop -mark-up-list=<id1> -project-dir=/path_to_project/ -- ./lu.C.16
1-15 ./lu.C.16
```
or
```
0 advixe-cl -collect tripcounts -flop -loops=scalar,loop-height=0 -project-dir=/path_to_project/ -- ./lu.C.16
1-15 ./lu.C.16
```



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

* This is the initial GUI while you execute the above command

![alt text](/tutorial/roofline_initial.png)

* If you click on the project e000 then you will see on your right the following data with generic overview of the efficiency

![alt text](/tutorial/summary.png)

* If you click on the tab "Survey & Roofline" you can see data per loop and identify potential bottlenecks.

![alt text](/tutorial/survey_roofline.png)

* If you click on the Roofline menu (left arrow on the above screenshot), you get the roofline model below. We select the checkbox that the arrows points as we use only one MPI process for the experiments and the initial roofline peak results are about full node. The data points represent different loops. The colors and the size of the data points indicate the percentage of the time, larger the circle, it consumes more time. The red color shows that the performance is not efficient. If you click on one of the data points the window below will show the corresponding code. We can see multiple peak lines related to memory bandwidth and operations. Through this approach we can get familiar when part of the code fits to the memory and if the code does not achieve quite good performance.

![alt text](/tutorial/roofline_model.png)

* If you click the tab "Why No Vectorization?" below the roofline model, you can find some tips to improve the code

![alt text](/tutorial/recommendations.png)

