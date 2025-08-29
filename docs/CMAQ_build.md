## Building the CMAQ model (on Aire)

This document walks you through the steps to build the CMAQ model and its dependencies. These instructions are written for building the model on Aire, but could be adapted for any HPC system.

### What is CMAQ?
CMAQ stands for Community Multiscale Air Quality. It is a regional chemical transport model developed and maintained by the US Environmental Protection Agency (EPA), but many updates and other contributions come from the community of users (hence, the Community aspect of the name). 

### Model requirements
There are a few requirements for building CMAQ. Assuming that you are building the model on Aire, these requirements are met. However, if you are building on a different HPC, you will want to check that your system meets these requirements before getting started. The basic requirements are:
1. **Operating system: Unix or Unix-like.**
   - CMAQ needs a Unix or Unix-like system. Any Linux distrubtions will work, as will MacOS.
2. **Space: at least 7 GB.**
   - The amount of space for your build (not the model inputs and outputs) depends on the version of the model and the availability of the model dependencies. On Aire, most of the libraries are already installed so these are not a space issue.
   - The base CMAQ source code is relatively small, about 250 MB and the final build is about 50 MB. Other versions of the model, such as with the integrated source apportionment method turned on or the coupled WRF-CMAQ version, will take more space.
   - The required I/O API software is about 6 GB of space.
   - The additional libraries can take up to another 2-3 GB of space, depending on what needs to be locally installed versus what is already available on the HPC you are using.
3. **Software: Fortran and C compilers, MPI, and netcdf libraries.**
   - CMAQ requires several libraries and software be available. These are:
       - Fortran and C compilers. On Aire, there is the OneAPI Intel compiler.
       - Message Passing interface (MPI) compatible with the Fortran and C compilers. On Aire, this is the IntelMPI that is included in the OneAPI install.
       - netcdf, netcdf-Fortran, and pnetcdf libraries. These are installed on Aire and available as modules. If these libraries are not available on your HPC system, you will need to build them. There are [instructions for building the necessary software](https://github.com/USEPA/CMAQ/blob/main/DOCS/Users_Guide/Tutorials/CMAQ_UG_tutorial_configure_linux_environment.md#tutorials-and-scripts-for-building-netcdf-and-io-api-libraries-for-cmaq) from the CMAQ management team.
    
### Building the model
There are two required builds: a program called IOAPI that handles all of the input and output of files and the CMAQ model itself. Start with IOAPI (and be aware that this is the more painful build of the two).
#### Building I/O API
1. Identify the path for netcdf, netcdf-f, pnetcdf and MPI. Also identify your compiler for C, C++, and Fortran. On Aire, to do this, you will first need to load the modules through the CEMAC interface.
     1. Source the CEMAC shell script that will allow you to load the modules. `. /users/cemac/cemac.sh`
     2. Load the necessary modules `module load intel/2025.2.0 intelmpi/2025.2.0 hdf5 netcdf/4.9.3`. This will load the modules from the CEMAC builds, and you will be able to see the location of where each is built.
     3. Type `module avail` and the available modules, including those you have loaded will show. The paths for each build are listed above each, but you can also check using certain commands. For example, for netcdf, netcdf-f, and pnetcdf, load the modules then use the following commands to find the path to these libraries.
     ```
     nc-config --libs
     nf-config --flibs
     pnetcdf-config --libdir
     ```
     4. Finally, identify the compilers. On Aire with Intel OneAPI, these are 
      ```
     CC   = icx
     CXX  = icpx
     FC   = ifx
     ```
2. Download I/O API
  ```
  git clone https://github.com/cjcoats/ioapi-3.2
  cd ioapi-3.2
  git checkout -b 20200828
  ```
3. Modify the I/O API top-level Makefile with the path and compiler information you identified in Step 1.
  ```
  cp Makefile.template Makefile
  vi Makefile
  ```
    1. Starting on line 138, uncomment (delete the # sign) the code until line 146.
    2. Modify lines 138-146 with the information relevant to your system.
      1. For the BIN, this is your compiler version. On Aire, use `Linux2_x86_64ifx`.
      2. Change `INSTALL = ${HOME}` to `INSTALL = ${BASEDIR}`.
      3. Add `"-DIOAPI_NCF4"` to `IOAPIDEFS`.
      4. Change `NCFLIBS` to `NCFLIBS = -L/path/to/netcdf -lnetcdf -L/path/to/netcdf-f -lnetcdff -L/path/to/pnetcdf -lpnetcdf -L/path/to/mpi -lmpi`.
    3. Comment out lines 192 and 193 (type # in front of text).
    4. Save the changes to the Makefile and exit the document `:wq`.
4. Move to the ioapi directory and modify the Makeinclude.BIN file corresponding to the BIN you chose in the Makefile
   1. `cd ioapi`
   2. On Aire, `vi Makeinclude.Linux2_x86_64ifx`
   3. Modify the compilers to match what you found in step 1. (Also add the FC flags shown below).
      ```
      CC   = icx
      CXX  = icpx
      FC   = ifx -auto -warn notruncated_source -Bstatic -static-intel
      ```
   4. On lines 23 and 24, add or modify the existing flag to `-qopenmp`
5. 

