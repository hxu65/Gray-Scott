






# Coeus recup evaluation:
# How to build
Ensure adios2, openmpi installed
```
git clone https://github.com/hxu65/gray-scott.git
cd gray-scott
mkdir build
cd build
cmake ../
make -j8
cp gray-scott/simulation/adios2.xml ./
```
# How to run

### gray-scott configuration
The configuration file, setting-files.json, includes several key parameters:<br>
"L": Specifies the size for each I/O operation, typically set to 128, 64, or 32.<br>
"steps": Defines the number of simulation steps.
Here is the example for setting file:
L: 128 means the problem size.
Steps: 800 means the number of problems
output:gs.bp means the output file location
"adios_config": "adios2.xml", the location for the adios2.xml
```
{
  "L": 128,
  "Du": 0.2,
  "Dv": 0.1,
  "F": 0.01,
  "k": 0.05,
  "dt": 2.0,
  "plotgap": 10,
  "steps": 800,
  "noise": 0.01,
  "output": "gs.bp",
  "checkpoint": true,
  "checkpoint_freq": 70,
  "checkpoint_output": "ckpt.bp",
  "restart": false,
  "restart_input": "ckpt.bp",
  "adios_config": "adios2.xml",
  "adios_span": false,
  "adios_memory_selection": false,
  "mesh_type": "image"
}
```

### Running the Gray-Scott Simulation with ADIOS2 Without Derived Variables


```
cd gray-scott
mpirun -n 4 build/adios2-gray-scott settings-files.json 0
```
Setting the final value to 0 ensures that derived variables will not be used.

### Run the gray scott with adios2 and with derived variables
```
cd gray-scott
mpirun -n 4 build/adios2-gray-scott settings-files.json 1
```
Setting the final value to 1 ensures that derived variables will be used.



### Hashing a bp5 file and generate a same bp5 file with derived variables(hashing)
```
cd gray-scott
mpirun -n 4 build/adios2-hashing gs.bp hashing.bp
```
gs.bp is the original file without hashing variables. <br>
hashing.bp is the file for derived variable with hashing operation

### compare two bp5 file with hashing variables
```
cd gray-scott
mpirun -n 4 build/hashing-comparison gs.bp gs.bp
```

## coeus-recup evaluation case 1
Please Note:  change the parameters in settings-files.json and settings-files2.json <br>
for setting-files.json, the output file should be "output": "gs1.bp", <br>
for setting-files2.json, the output file should be "output": "gs2.bp"  <br>
You can also add use /hdd/gs2.bp for output file location

```
# run the gray-scott to generate a bp5 file without hashing
mpirun -n 4 build/adios2-gray-scott settings-files.json 0
# hashing it
mpirun -n 4 build/adios2-hashing gs1.bp hashing1.bp

# run the second gray-scott to generate a bp5 file without hashing
mpirun -n 4 build/adios2-gray-scott settings-files2.json 0
# hashing it
mpirun -n 4 build/adios2-hashing gs2.bp hashing2.bp

# compare them
mpirun -n 4 build/hashing-comparison hashing1.bp hashing2.bp
```

## coeus-recup evaluation case 2
Please Note:  change the parameters in settings-files.json and settings-files2.json <br>
for setting-files.json, the output file should be "output": "gs1.bp", <br>
for setting-files2.json, the output file should be "output": "gs2.bp"  <br>
You can also add use /hdd/gs2.bp for output file location

```
# run the gray-scott with hashing
mpirun -n 4 build/adios2-gray-scott settings-files.json 1
mpirun -n 4 build/adios2-gray-scott settings-files2.json 1

# compare the hashing value
mpirun -n 4 build/hashing-comparison gs1.bp gs2.bp
```


# The original README for ADIOS2 gray-scott 
## For reference only

This is a 3D 7-point stencil code to simulate the following [Gray-Scott
reaction diffusion model](https://doi.org/10.1126/science.261.5118.189):

```
u_t = Du * (u_xx + u_yy + u_zz) - u * v^2 + F * (1 - u)  + noise * randn(-1,1)
v_t = Dv * (v_xx + v_yy + v_zz) + u * v^2 - (F + k) * v
```

## How to run

Make sure MPI and ADIOS2 are installed and that the `PYTHONPATH` includes the ADIOS2 package.
Make sure the adios2-examples/bin installation directory is in the `PATH` (conda and spack installations should take care of this aspect).

From a scratch directory copy the config files from your installation of adios2-examples:

```
$ cp -r <adios2-examples-install-prefix>/share/adios2-examples/gray-scott .
$ cd gray-scott
$ mpirun -n 4 adios2-gray-scott settings-files.json
========================================
grid:             64x64x64
steps:            1000
plotgap:          10
F:                0.01
k:                0.05
dt:               2
Du:               0.2
Dv:               0.1
noise:            1e-07
output:           gs.bp
adios_config:     adios2.xml
process layout:   2x2x1
local grid size:  32x32x64
========================================
Simulation at step 10 writing output step     1
Simulation at step 20 writing output step     2
Simulation at step 30 writing output step     3
Simulation at step 40 writing output step     4
...


$ bpls -l gs.bp
  double   U     100*{64, 64, 64} = 0.0907758 / 1
  double   V     100*{64, 64, 64} = 0 / 0.674811
  int32_t  step  100*scalar = 10 / 1000


$ python3 gsplot.py -i gs.bp

```

## Analysis example how to run

```
$ mpirun -n 4 adios2-gray-scott settings-files.json
$ mpirun -n 2 adios2-pdf-calc gs.bp pdf.bp 100
$ bpls -l pdf.bp
  double   U/bins  100*{100} = 0.0907758 / 0.991742
  double   U/pdf   100*{64, 100} = 0 / 4096
  double   V/bins  100*{100} = 0 / 0.668056
  double   V/pdf   100*{64, 100} = 0 / 4096
  int32_t  step    100*scalar = 10 / 1000

$ python3 pdfplot.py -i pdf.bp
OR
$ mpirun -n 8 python3 pdfplot.py -i pdf.bp -o u
This is a parallel script, each process plots one PDF.
Each process plots the middle slice of their subarray U/pdf[x:y,:]

```

## How to change the parameters

Edit settings.json to change the parameters for the simulation.

| Key           | Description                           |
| ------------- | ------------------------------------- |
| L             | Size of global array (L x L x L cube) |
| Du            | Diffusion coefficient of U            |
| Dv            | Diffusion coefficient of V            |
| F             | Feed rate of U                        |
| k             | Kill rate of V                        |
| dt            | Timestep                              |
| steps         | Total number of steps to simulate     |
| plotgap       | Number of steps between output        |
| noise         | Amount of noise to inject             |
| output        | Output file/stream name               |
| adios_config  | ADIOS2 XML file name                  |

Decomposition is automatically determined by MPI_Dims_create.

## Examples

| D_u | D_v | F    | k      | Output
| ----|-----|------|------- | -------------------------- |
| 0.2 | 0.1 | 0.02 | 0.048  | ![](img/example1.jpg?raw=true) |
| 0.2 | 0.1 | 0.03 | 0.0545 | ![](img/example2.jpg?raw=true) |
| 0.2 | 0.1 | 0.03 | 0.06   | ![](img/example3.jpg?raw=true) |
| 0.2 | 0.1 | 0.01 | 0.05   | ![](img/example4.jpg?raw=true) |
| 0.2 | 0.1 | 0.02 | 0.06   | ![](img/example5.jpg?raw=true) |


## In situ pipeline example

In adios2.xml, change all IO groups' engine to SST.

      <engine type="SST"

Launch the pipeline in 4 separate terminals:
```
$ mpirun -n 4 adios2-gray-scott settings-staging.json
$ mpirun -n 1 adios2-pdf-calc gs.bp pdf.bp 100
$ mpirun -n 1 python3 pdfplot.py -i pdf.bp
$ mpirun -n 1 python3 gsplot.py -i gs.bp
```

MPMD mode run in a single terminal:
```
$ mpirun -n 4 adios2-gray-scott settings-staging.json : \
         -n 1 adios2-pdf-calc gs.bp pdf.bp 100 :           \
         -n 1 python3 pdfplot.py -i pdf.bp :         \
         -n 1 python3 gsplot.py -i gs.bp
```

## In situ batch and interactive visualization with ParaView Catalyst

This requires ADIOS 2.9.0 or later, due to the use of `ParaViewADIOSInSituEngine` plugin for ADIOS.
Internally, this plugin uses the ADIOS inline engine to pass data pointers to ParaView's
[Fides](https://fides.readthedocs.io/en/latest/) reader
and uses ParaView [Catalyst](https://catalyst-in-situ.readthedocs.io/en/latest/index.html)
to process a user python script that contains a ParaView pipeline.
Fides is a library that provides a schema for reading ADIOS data into visualization services
such as ParaView. By integrating it with ParaView Catalyst, it is now possible to perform
in situ visualization with ADIOS2-enabled codes without writing adaptors. All that is needed
from the user is a simple JSON file describing the data.


`simulation/settings-inline.json` uses the `adios2-inline-plugin.xml` configuration file.
It sets the engine type to `plugin` and provides the `PluginName` and `PluginLibrary`
parameters required when using engine plugins. In addition, you will need to set the
environment variable `ADIOS2_PLUGIN_PATH` to contain the path to the `libParaViewADIOSInSituPlugin.so`
shared library built by ADIOS.

In the `catalyst` dir, there is a `gs-fides.json`, which is the data model Fides uses to read the data.
The `gs-pipeline.py` contains the pipeline Catalyst will execute on each step.
These files are passed as parameters to the engine plugin (see parameters `DataModel` and `Script` in
the `adios2-inline-plugin.xml` file).


### Build and Run

This example is built as normal (making sure you are using ADIOS v2.9.0 or later)
and does not have build dependencies on ParaView.

This example requires ParaView 5.11, which is currently in release phase.
In order to perform in situ visualization, you'll need to build from source.
First build the [Catalyst](https://gitlab.kitware.com/paraview/catalyst) stub library.

```
$ git clone https://gitlab.kitware.com/paraview/catalyst.git
$ mkdir catalyst-build
$ cd catalyst
$ git checkout v2.0.0-rc3
$ cd ../catalyst-build
$ cmake -GNinja -DCATALYST_USE_MPI=ON -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/path/to/install/catalyst ../catalyst
$ ninja && ninja install
```

Now you'll need to build ParaView using the following instructions:

```
$ git clone --recursive https://gitlab.kitware.com/paraview/paraview.git
$ mkdir paraview-build
$ cd paraview
$ git checkout v5.11.0-RC2
$ git submodule update --init --recursive
$ cd ../paraview-build
$ cmake -GNinja -DPARAVIEW_USE_PYTHON=ON -DPARAVIEW_USE_MPI=ON -DPARAVIEW_ENABLE_FIDES=ON -DPARAVIEW_ENABLE_CATALYST=ON \
-DADIOS2_DIR=/path/to/adios2.8.0 -Dcatalyst_DIR=/path/to/catalyst -DCMAKE_BUILD_TYPE=Release ../paraview
$ ninja
```

The ADIOS2 lib directory should contain `libParaViewADIOSInSituEngine.so`.
Set the following env variables.
```
$ export ADIOS2_PLUGIN_PATH=/path/to/adios2-build/lib
$ export CATALYST_IMPLEMENTATION_NAME=paraview
$ export CATALYST_IMPLEMENTATION_PATHS=/path/to/paraview-build/lib/catalyst
```

To run:
```
$ mpirun -n 4 build/gray-scott simulation/settings-inline.json
```

### Interactive visualization

Open the ParaView GUI. `/path/to/paraview/build/bin/paraview`
On the `Catalyst` menu, click `Connect`. You can leave the default port of 22222.
Hit Ok.
Then in the `Catalyst` click `Pause Simulation`.
Now you can run the simulation, same as above.
The simulation will start and Catalyst will connect to ParaView.
At this point you can click the gray buttons beside the extracts. This will pull the data to
ParaView, allowing you to interact with it. When you're ready for the simulation to resume,
in the `Catalyst` menu, click `Continue`. You will see the visualizations update as the simulation runs.
You can make edits to your pipeline and pause/continue the simulation.
