
# MPI & OpenMP Task Automation

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

_Important Note: This project is in early development. Features are sparse and bugs may arise._

Let' s assume we would like to estimate the integral from a to b of an equation f(x) using the trapezoidal rule.

Let' s also assume that we have already developed an MPI / OpenMP / Hybrid program, named **mpi_trap.c** to do so and this program defines a macro named **nTraps**, which corresponds to the number of trapezoids that are going to be used in the calculation of the integral.

## **The Suite**

[setup.sh](setup.sh) defines some aliases, set's up the console prompt and loads mpiP and OpenMP when sourced. You really only need loading mpiP and OpenMP. Everything else is entirely optional.

    . ./setup.sh

[compile.sh](compile.sh) is responsible for compiling the MPI, OpenMP or Hybrid supplied source file.

It is expecting at least one arguement, which should be the source file. Any other arguements are grouped and form pairs of keys and values. Each pair represents a macro, named **key** with a value of **value**, that must be defined during the compilation of the program.

    ./compile.sh mpi_trap.c nTraps 512

    [compile.sh] enable profiling: y
    [compile.sh] link OpenMP: n

Executing it with the --clean option deletes the executable.

[schedule.sh](schedule.sh) is responsible for generating a job and submitting it to the PBS queue.

It is expecting exactly two arguements, the executable and the number of processes that should be created.

    ./schedule.sh mpi_trap.x 16

    [schedule.sh] name='mpi_trap1_16_argo059_job', id='14524.argo', ps=16, ns=2, ppn=8

    Job id            Name             User              Time Use S Queue
    ----------------  ---------------- ----------------  -------- - -----
    12507.argo        myJob            argo081                  0 Q workq
    14524.argo        mpi_trap1_16_ar  argo059           00:00:00 R workq

Executing it with the --clean option removes any mpiP associated file, any job file generated by it and removes any associated job from the queue.

[profile.sh](profile.sh) is responsible for compiling our program with different (key, value) macro pairs, running it with different number of processes and collecting our measurements.

It is expecting exactly two arguements, the source file and a [profiling description](#profiling-descriptions).

    echo -e "y\nn\ny\nn\ny\nn\ny\nn\ny\nn\n" | ./profile.sh ./mpi_trap.c ./description.sh

    head ./out/20_11_2019/23_47_53/results.csv

    nTraps   , Processes, Time    , Speed Up       , Εfficiency
    1048576  , 1        , 0.000827, 1.0            , 1.0
    1048576  , 2        , 0.000982, 0.84215885947  , 0.421079429735

[ui.sh](ui.sh) offers some basic user interface components and is being sourced by the other files.

[config.sh](config.sh) contains the configuration settings and is being sourced by the other files.

* **COMPILER** indicates the script that should be used to compile the source file.
* **SCHEDULER** indicates the script that should be used to schedule the executable's run.
* **OUTPUT_ROOT** indicates the root directory. Our scheduling and profiling script create some files and directories. These files and directories are going to be saved under the root directory.
* **USER_ID** is being used when querying the job queue to check if a job has finished.
* **TIME_PATTERN** is a regular expression indicating the format of our timer's output.
* **EDITOR** and **EDITOR_ARGS** are optional and are used by the profiler to open files in your favorite editor.

## **Profiling Descriptions**

The profiler is expecting a profiling description, which must define:

* a variable name **MACRO**, containing the name of the macro to be defined when running the profiler.
* an array named **VALUES**, containing the values that the macro should receive.

```bash
#!/bin/bash

MACRO="nTraps"

VALUES=()

for ((power = 20; power <= 28; power += 2))
do
    VALUES+=( "$(( 2 << ($power - 1) ))" )
done
```

