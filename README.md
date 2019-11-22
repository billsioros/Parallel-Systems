
# MPI & OpenMP Task Automation

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

_Important Note: This project is in early development. Features are sparse and bugs may arise._

## **The Suite**

[setup.sh](setup.sh) defines some aliases, set's up the console prompt and loads **mpiP** and **OpenMP** when sourced.

You really only need loading **mpiP** and **OpenMP**. Everything else is entirely optional.

[compile.sh](compile.sh) is responsible for compiling the MPI, OpenMP or Hybrid supplied source file.

It is expecting at least one arguement, which should be the source file. Any other arguements are grouped and form pairs of keys and values. Each pair represents a macro, named **key** with a value of **value**, that is going to be defined during the compilation of the program.

[schedule.sh](schedule.sh) is responsible for generating a job and submitting it to the PBS queue.

It is expecting at least two arguements, the executable and the number of processes that should be created. A third arguement may be supplied, indicating where the generated files should be saved. The supplied path is relative to **OUTPUT_ROOT** and its default value is *NUMBER_OF_PROCESSES/DATE/TIME*.

[profile.sh](profile.sh) is responsible for compiling our program with various (key, value) macro pairs, running it with different numbers of processes and collecting our measurements.

It is expecting exactly two arguements, the source file and a [profiling description](#profiling-descriptions).

[config.sh](config.sh) contains the [configuration settings](#configuration) and is being sourced by the other files.

[ui.sh](ui.sh) offers some basic user interface components and is being sourced by the other files.

## **Installation**

    git clone https://github.com/billsioros/mpi-openmp-task-automation

## **Configuration**

* **COMPILER** indicates the script that should be used to compile the source file.
* **SCHEDULER** indicates the script that should be used to schedule the executable's run.
* **OUTPUT_ROOT** indicates the directory, under which the files and directories generated by our scheduler and profiler are going to be saved.
* **USER_ID** is being used when querying the job queue to check if a job is done running.
* **TIME_PATTERN** is a regular expression indicating the format of our timer's output.
* **EDITOR** and **EDITOR_ARGS** are optional and are used by the profiler to open files in your favorite editor.

## **Profiling Descriptions**

[profile.sh](./profile.sh) is expecting a profiling description, which **must** define:

* a string named **MACRO**, containing the name of the macro that should be defined by [compile.sh](./compile.sh).
* an array named **VALUES**, containing the values that macro **MACRO** should receive in different runs.

## **Example**

Let' s assume we would like to estimate the integral from a to b of an equation f(x) using the trapezoidal rule.

Let' s also assume that we have already developed an **MPI** program, named **mpi_trap1.c** to do so and this program defines a macro named **nTraps**, which corresponds to the **number of trapezoids** that are going to be used in the calculation of the integral.

Firstly, we need to source [setup.sh](setup.sh) like so

    . ./setup.sh

### **Compiling**

We now need to compile our source file using [compile.sh](compile.sh) as follows

    ./compile.sh mpi_trap1.c nTraps 512

    [compile.sh] enable profiling: y
    [compile.sh] link OpenMP: n

This results in the creation of an executable file named **mpi_trap1.x**, inside which the macro **nTraps** has been assigned the value **512**.

Executing [compile.sh](compile.sh) with the _--clean_ option deletes the executable.

### **Scheduling**

Scheduling the executable can be achieved like so

    ./schedule.sh mpi_trap1.x 16

    [schedule.sh] name='mpi_trap1_16_argo059_job', id='14524.argo', ps=16, ns=2, ppn=8

    Job id            Name             User              Time Use S Queue
    ----------------  ---------------- ----------------  -------- - -----
    12507.argo        myJob            argo081                  0 Q workq
    14524.argo        mpi_trap1_16_ar  argo059           00:00:00 R workq

* **ps** stands for the **number of processes**
* **ns** stands for the **number of nodes**
* **ppn** stands for the **number of processes per node**

The following files and directories are generated

    find ./out

    ./out/
    ./out/16
    ./out/16/21_11_2019
    ./out/16/21_11_2019/15_06_30
    ./out/16/21_11_2019/15_06_30/mpi_trap1_16_argo059_job.stderr
    ./out/16/21_11_2019/15_06_30/mpi_trap1_16_argo059_job.stdout

Executing [schedule.sh](schedule.sh) with the _--clean_ option removes any mpiP and job associated files and removes any **USER_ID** associated job from the queue.

### **Profiling**

We firstly need to define a [profiling description](#profiling-descriptions) like so

```bash
#!/bin/bash

MACRO="nTraps"

VALUES=()

for ((power = 20; power <= 28; power += 2))
do
    VALUES+=( "$(( 2 << ($power - 1) ))" )
done
```

We can compile the source file with different numbers of trapezoids defined and schedule it with different numbers of processes in a single command as follows

    echo -e "y\nn\ny\nn\ny\nn\ny\nn\ny\nn\n" | ./profile.sh ./mpi_trap1.c ./description.sh

The _echo_ command is used so that [compile.sh](compile.sh) runs non-interactively.

The following files and directories are generated

    find ./out

    ./out/
    ./out/21_11_2019
    ./out/21_11_2019/15_13_08
    ./out/21_11_2019/15_13_08/67108864
    ./out/21_11_2019/15_13_08/67108864/4
    ./out/21_11_2019/15_13_08/67108864/4/mpi_trap1_4_argo059_job.stderr
    ./out/21_11_2019/15_13_08/67108864/4/mpi_trap1_4_argo059_job.stdout
    ...
    ./out/21_11_2019/15_13_08/268435456/32
    ./out/21_11_2019/15_13_08/268435456/32/mpi_trap1_32_argo059_job.stderr
    ./out/21_11_2019/15_13_08/268435456/32/mpi_trap1_32_argo059_job.stdout
    ./out/21_11_2019/15_13_08/268435456/16
    ./out/21_11_2019/15_13_08/268435456/16/mpi_trap1_16_argo059_job.stderr
    ./out/21_11_2019/15_13_08/268435456/16/mpi_trap1_16_argo059_job.stdout
    ./out/21_11_2019/15_13_08/results.csv

Let' s now see what our measurements look like

    head -5 ./out/21_11_2019/15_13_08/results.csv

    nTraps   , Processes, Time    , Speed Up       , Εfficiency
    1048576  , 1        , 0.000925, 1.0            , 1.0
    1048576  , 2        , 0.00082 , 1.12804878049  , 0.564024390245
    1048576  , 4        , 0.000841, 1.09988109394  , 0.274970273485
    1048576  , 8        , 0.001022, 0.905088062622 , 0.113136007828

## **Licence**

This project is licensed under the [MIT License](./LICENCE)

