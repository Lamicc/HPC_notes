# HPC_notes
## Job Scheduling
| Queues        | Hardware description          | Time limits  |
| ------------- |:-------------:| -----|
| HighMemLongterm.q     | 2 x 10 core Intel Haswell E5-2660 v3 2.60GHz with 384GB RAM | Few months |
| HighMemShortterm.q      | 2 x 10 core Intel Haswell E5-2660 v3 2.60GHz with 384GB RAM      |  7 days |
| LowMemLongterm.q | 2 x 10 core Intel Haswell E5-2660 v3 2.60GHz with 192GB RAM     | Few months |
| LowMemShortterm.q| 2 x 10 core Intel Haswell E5-2660 v3 2.60GHz with 192GB RAM| 7 days |
| InterLowMem.q| 2 x 10 core Intel Haswell E5-2660 v3 2.60GHz with 192GB RAM | 24hours |

|Parallel environments| Purpose |
|-------------|-------------|
|mpislots|Used for mpi jobs with filling each node and then going to the next one.|
|mpinodes| Used fro mpi jobs with roudrobin way. Thus putting one thread in each node.|
|smp |Used from smp jobs, so running in a single node.|

### Scripts

Set the console environment

    #!/bin/env bash

Set memory requirement per core
    
    #$ -l h_vmem=2G

Set the parallel environment and number of cores

    #$ -pe smp 60

Set the standard output and error

    #$ -o ~/path/output.file.$JOB_ID
    #$ -e ~/path/error.file.$JOB_ID

Set the name of the queue will like to submit(optional)

    #$ -q HighMemLongterm.q
    
Set the modules you need for running your job

    module load general/R/3.2.1

Set the current working directory

    #$ -cwd

Set the current environment variables
    
    #$ -V

For SMP jobs set the number of threads

    export OMP_NUM_THREADS=20

Set the name of the job

    #$ -N myjobname

For setting the wall time

    #$ -l h_rt=100:00:00
    
Set the project you belong

    #$ project –p projectname

Set the email address

    #$ -M some.person.kcl.ac.uk

For sending the email when job finishes
    
    #$ -m e

For jobs requiring a lot of slots, enables the backfilling

    #$ -R y

For mpi programs

    mpirun –np $NSLOTS /my/input/file my/output/file
    
### Scripts Array jobs

Set the variable range

    #$ -t 1-100

Use the variable in your code or software

    test=$SGE_TASK_ID

### Controlling jobs

Check the status of the jobs of all users

    qstat –u “*”

Check the status of a job and reason for waiting

    qstat –j jobid

Pause and release a job

    qhold or qrel jobid

Check available hosts
    
    qhosts
    
* If your job needs communication at the beginning or at the end of the simulation, ethernet connected
nodes are good enough;
* If your job uses communication all the time then infiniband nodes are needed;

## Software
User Environmental modules
* Using environmental modules different version of
software can be run;
* Resolve any dependencies conflicts;
* Set’s the proper environmental variables for each
software;
* Simple commands for loading/unloading, can be
used in scripts, can users create their own;

Check available software:

    module av
    
    -------------------------------------------------------------------
    /opt/apps/etc/modules//general/quantum-espresso/6.0:    #Path of the module
    module-verbosity on                                     #More descriptive messages
    module load libs/fftw/3.3.5/intel15-double              #Load required environment
    module-prereq libs/fftw/3.3.5/intel15-double                  #Set as prerequisite load
    module-conflict general/quantum-espresso                      #Not allowed to load with other in this folder
    prepend-path PATH /opt/apps/general/quantum-espresso/6.0/bin  #Usually bin folder has the executable binaries
    -------------------------------------------------------------------

List currently loaded modules in your account:

    module list

Load/unload a module:

    module load/unload compilers/gcc/5.3.0

Hint: For load/unload, you can use tab key autocomplete;

Show the “interesting” content of the module:

    module show compilers/gcc/5.3.0
    
Unload all the loaded modules(be carefull):

    module purge
    
### Setting up module file for your software: 

1. Setup the folder that your module files are:

        module use folder/in/your/home/directory/that/you/keep/module/files

1. Setup your module file accordingly:

        #%Module
        #
        # @name: NAMD
        # @version: 2.11
        #
        # Customize the output of `module help` command
        # ---------------------------------------------
        proc ModulesHelp { } {
        puts stderr "\tAdds GNU Cross Compilers to your environment variables"
        puts stderr "\t\t\$PATH, \$MANPATH"
        }
        # Customize the output of `module whatis` command
        # -----------------------------------------------
        module-whatis "loads the [module-info name] environment"
        # Define internal modulefile variables (Tcl script use only)
        # ----------------------------------------------------------
        set name NAMD
        set version 2.11
        set prefix /opt/apps/general/${name}/${version}
        # Check if the path exists before modifying environment
        # -----------------------------------------------------
        if {![file exists $prefix]} {
        puts stderr "\t[module-info name] Load Error: $prefix does not exist"
        break
        exit 1
        }
        # Update common variables in the environment
        # ------------------------------------------
        conflict general/NAMD
        module load compilers/gcc/5.3.0
        prereq compilers/gcc/5.3.0
        module load libs/openmpi/1.10.0/gcc5.3.0
        prereq libs/openmpi/1.10.0/gcc5.3.0
        module load libs/fftw/2.1.5/gcc5.3.0
        prereq libs/fftw/2.1.5/gcc5.3.0
        prepend-path PATH $prefix
        prepend-path INCLUDE $prefix/inc
        
### Compilation of scientific software and code

Start an interactive session:

    qrsh –pe mpislots 20

Most of the software are based in autoconf and cmake for managing the building process:

    cmake .. –PARAMETERS or configure –PARAMETERS

Read the documentation in order to find and load the proper dependencies using the module infrastructure;
This will scan your system and create a make parameter file;
Parameters for cmake and autoconf can be found in documentation. Themost common is the destination path of the build:        

       -prefix_parameter=installation/folder/in/your/home/directory

Another important parameter for choosing the CPU architecture is:

     -SIMD_parameter=AVX2_256 or AVX_256
where the first on is for Haskwell CPUs(Rosalind nodes) and the second one is
for Ivy Bridge CPUs(ADA nodes)

You can start the compilation using 20 cores:

    make -j 20
Note: Sometimes too many cores can cause the compilation to
fail, so reduce this number.

After the successful compilation you can move the binaries to the final destination:

     make install

Set up in the module file the environmental variables and paths
according to the documentation of the software. If not
compiled as a static, you need to add the dependencies;

### Testing of scientific software
Scientific software usually comes with a testing feature called regression tests. The most
common command for running them are:

    make tests or make check

That’s not enough! Regression tests cover only a small proportion of the
testing of functionality of a software. Scientific software consists of many subroutines or
code units used in different combinations depending the input of the user . So, only a
small proportion of these combinations are covered by these tests;

Each user must test the scientific software against his/her simulation input;

“Sanity checks” can be e.g. checking of conservation of energy or summing of possibilities
to be unit or comparison with a similar problem with analytical solution;

Unfortunately, very hard to predict which will work so in general it is “trial and error”
procedure. If the scientific software do not pass regressions tests or your sanity checks, you
need to try with different flags or different versions of library dependencies;

Scientific software that is installed by default in HPC cluster, is tested only with regression
tests. If not passing any user testing, it is responsibility of the user to compile a version that
pass his/her tests. Furthermore, it is responsibility of the user to compile software with
different features, plugins, optimizations etc. other than already installed by default;

You can check error in the queue system:
    
    qstat –j jobid
   
## Rosalind
| Compute nodes           |||||
| ------------- |-------------| -------------| ------------- | ------------- |
|64|Dell PowerEdge R630 |2 x 10 core Intel Haswell E5-2660 v3 2.60GHz |192GB ram |Infiniband QRD40Gb/s networking|
|32|Dell PowerEdge R630|2 x 10 core Intel Haswell E5-2660 v3 2.60GHz|384GB ram|Infiniband QRD 40Gb/s networking|
|32|Dell PowerEdge R630|2 x 10 core Intel Haswell E5-2660 v3 2.60GHz|384GB ram|10Gb/s Ethernet networking|
|16|HP ProLiant BL460c blade|2 x 8 core Intel Ivybridge 2.7GHz| 264GB ram| 10GbE connectivity to Dell S6000 40GbE core|

|Storage||
| ------------- |-------------|
|Lustre scratch storage| 524TB|
|Ceph storage |2352TB raw storage|
|Ceph backup storage |1176TB raw storage|
|Openstack block storage| 100TB storage|
