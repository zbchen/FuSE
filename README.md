# Pre install docker environment

We have deployed the experimental environment on docker. Please pre install docker on your host

# Download

Download docker image:
```sh
$ sudo docker pull dockeryangxu/fpse:2.0
```

If the image is pulled successfully, please check there is an image named apsecpaper/apsecpaper exists.
```sh
$ sudo docker images
REPOSITORY          TAG       IMAGE ID       CREATED       SIZE
dockeryangxu/fpse   2.0       5b15cd41514d   2 hours ago   28.1GB
```


Start to run the container in interactive mode.
```sh
$ sudo docker run -it dockeryangxu/fpse:2.0 bash
$ su aaa
```

# Run a example

Enter the directory `/home/aaa/analysis` and list all contents.

```sh
$ cd /home/aaa/analysis
```

The following code is the example mentioned in our paper. The source code for the example is in `test/simple.c`.

```c
#include <klee/klee.h>

int main() {
float a,b,c;
klee_make_symbolic(&a, sizeof(a), "a");
klee_make_symbolic(&b, sizeof(b), "b");
klee_make_symbolic(&c, sizeof(c), "c");

if (cos(a) > log(b)){
  if (sin(a) < log(b)){
    c = c -1.0;
    if (c == 1.1)
      printf("Neve reach here !\n");
    }
  }
  return 0;
}

```

Before you can analyze this code, you need to set the parameters to run the script `run_solver.sh`: `./run_solver.sh [work_path] [file_name] [solver_type] [search_type]`, where

- work_path: The path of program file location.
- file_name: The program file name.
- solver_type: The solving modes, e.g. BVFP(`smt`, `bitwuzla`, `mathsat5`), RSO(`cvc5-real`, `dreal-is`), ISC(`fp2int`), FUZZ(`jfs`), Search(`gosat`), and Synergy(`smt-dreal`). The bold fields are setting parameters.
- search_type: The search modes, e.g. `bfs` and `dfs`.

If you want to run script with the `bfs + smt-dreal` configuration to analyze the example code,
```sh
$ ./benchmark/run_solver.sh test simple smt-dreal bfs
```

Less than 1 minutes, we can see the output at the terminal. The pathes are explored completely.

```sh
$ cat test/simple\&smt-dreal\&bfs.runlog
```

```
KLEE: KLEE: WATCHDOG: watching 177

KLEE: output directory is "/home/aaa/fp-solver/analysis/test/test&simple&smt-dreal&bfs_output"
KLEE: Using Z3 solver backend
KLEE: Replacing function "__isnanf" with "klee_internal_isnanf"
KLEE: Replacing function "__isnan" with "klee_internal_isnan"
KLEE: Replacing function "__isnanl" with "klee_internal_isnanl"
KLEE: Replacing function "__isinff" with "klee_internal_isinff"
KLEE: Replacing function "__isinf" with "klee_internal_isinf"
KLEE: Replacing function "__isinfl" with "klee_internal_isinfl"
KLEE: WARNING ONCE: function "gsl_ieee_set_mode" has inline asm
KLEE: ERROR: (location information missing) FloatPointCheck: FP Invalid found !
KLEE: NOTE: now ignoring this error at this location
>>>Synergy-Z3 exec time: 1.702099e+02 ms
KLEE: WARNING: SMT-DREAL: Z3 solving SAT and evaluate SUCCESS !
>>>Synergy-Z3 exec time: 5.062000e-03 ms
>>>Synergy-dreal exec time: 1.830823e+00 ms
KLEE: WARNING: SMT-DREAL: DReal solving UNKNOWN with all support and remove this state !
>>>Synergy-Z3 exec time: 6.328000e-03 ms
>>>Synergy-dreal exec time: 8.808838e+00 ms
KLEE: WARNING: SMT-DReal: DReal solving SAT and evaluate SUCCESS !
>>>Synergy-Z3 exec time: 5.023000e-03 ms
>>>Synergy-dreal exec time: 2.091033e+00 ms
KLEE: WARNING: SMT-DReal: DReal solving SAT and evaluate FAILURE and using JFS with seeds to solve !
>>>Fuzz with seed exec time: 7.092697e+02 ms
KLEE: WARNING: FUZZ with seed: solving SAT and evaluate SUCCESS !
>>>Synergy-Z3 exec time: 5.237000e-03 ms
>>>Synergy-dreal exec time: 1.521205e+00 ms
KLEE: WARNING: SMT-DREAL: DReal solving UNKNOWN with all support and remove this state !
>>>Synergy-Z3 exec time: 5.451000e-03 ms
>>>Synergy-dreal exec time: 1.598952e+00 ms
KLEE: WARNING: SMT-DReal: DReal solving SAT and evaluate FAILURE and using JFS with seeds to solve !
>>>Fuzz with seed exec time: 1.141540e+04 ms
KLEE: WARNING: FUZZ with seed: solving UNKNOWN evalute FAILURE and remove the state !
>>>Synergy-Z3 exec time: 8.463000e-03 ms
>>>Synergy-dreal exec time: 2.648600e-02 ms
KLEE: WARNING: SMT-DREAL: DReal solving UNKNOWN with all support and remove this state !
>>>Synergy-Z3 exec time: 2.075294e+02 ms
KLEE: WARNING: SMT-DREAL: Z3 solving UNSAT and remove the state !

KLEE: done: total instructions = 39
KLEE: done: completed paths = 3
KLEE: done: partially completed paths = 1
KLEE: done: generated tests = 4
Total exec time: 2.424297e+04 ms
```

If you want to analyze this program using the BVFP solver, please modify the `solver_type` parameter to `smt`，`bitwuzla` or `mathsat5`.  KLEE will take a long time (more than 30 min) to finish the program (you can CTRL+C to intrupt this program), because the source code of `sin/cos/log` are not avalible.  it can not explore all pathes fastly:

```sh
$ ./benchmark/run_solver.sh example simple smt bfs
...... 
# take a long time
```

# Obtain experimental results

Our experiments were performed on a server with Intel(R) Xeon(R) Platinum 8269CY CPU (2.50GHz) and the operating system is Ubuntu 18.04 LTS. 

To obtain the results, a machine with similar CPUs is required. Moreover, our experiments were run in 50 parallel.

## Analyze a program in Benchmark

Navigate to `/home/aaa/analysis/benchmark` and list the contents.

```sh
$ cd /home/aaa/analysis/benchmark
```

If you want to obtain experimental results for a single test program, e.g., `sf/gsl_sf_airy_Ai_deriv_e.c`. For example, obtain the experimental results of `jfs+dfs`.

```sh
$ ./run_solver.sh algorithm gsl_sf_ellint_Kcomp_e jfs bfs
```

After running, log and test cases are generated in the corresponding directory, you can read the log as follow:

```sh
$ cat "algorithm/gsl_sf_ellint_Kcomp_e&jfs&bfs.runlog"
```

```sh
......
>>>JFS exec time: 5.533964e+02 ms
KLEE: WARNING: FUZZ: JFS solving SAT and evaluate SUCCESS !
KLEE: ERROR: ellint.c:502: FloatPointCheck: Common Overflow found !
KLEE: NOTE: now ignoring this error at this location
>>>JFS exec time: 5.370376e+02 ms
KLEE: WARNING: FUZZ: JFS solving SAT and evaluate SUCCESS !
KLEE: ERROR: ellint.c:502: FloatPointCheck: Common Underflow found !
KLEE: NOTE: now ignoring this error at this location
>>>JFS exec time: 5.467868e+02 ms
KLEE: WARNING: FUZZ: JFS solving SAT and evaluate SUCCESS !
KLEE: ERROR: ellint.c:502: FloatPointCheck: Common Accuracy found !
KLEE: NOTE: now ignoring this error at this location
>>>JFS exec time: 3.149570e+04 ms
......
```

We can get the coverage information by running the script:

```sh
$ cd /home/aaa/analysis/benchmark
$ ./repaly.sh
```

```sh
......# some info
\n     Running ==== > algorithm&gsl_sf_ellint_Kcomp_e&jfs&bfs_output/
====  Replay Ktest ====
===>/home/aaa/fp-solver/analysis/benchmark3/algorithm/algorithm&gsl_sf_ellint_Kcomp_e&jfs&bfs_output/test000001.ktest
CHECK:  KTests have been generated !
===>python_res: ellint.c
===>gcno: /home/aaa/fp-solver/gsl/specfunc/.libs/ellint.gcno
gcno file is exit
KTest : /home/aaa/fp-solver/analysis/benchmark3/algorithm/algorithm&gsl_sf_ellint_Kcomp_e&jfs&bfs_output/test000001.ktest
KLEE-REPLAY: klee_assume(0)!
KLEE-REPLAY: NOTE: Test file: /home/aaa/fp-solver/analysis/benchmark3/algorithm/algorithm&gsl_sf_ellint_Kcomp_e&jfs&bfs_output/test000001.ktest
KLEE-REPLAY: NOTE: Arguments: "./gsl_sf_ellint_Kcomp_e" 
KLEE-REPLAY: NOTE: Storing KLEE replay files in /tmp/klee-replay-r0B8l6
gsl: ellint.c:503: ERROR: domain error
Default GSL error handler invoked.
KLEE-REPLAY: NOTE: EXIT STATUS: ABNORMAL 6 (0 seconds)
KLEE-REPLAY: NOTE: removing /tmp/klee-replay-r0B8l6
===>ktest_time_log: /home/aaa/fp-solver/analysis/benchmark3/algorithm/algorithm&gsl_sf_ellint_Kcomp_e&jfs&bfs_output/test000001.time
../gsl/gsl_mode.h: No such file or directory
ellint.c: No such file or directory
===>cover line res:14.285714285714285 , 2
../gsl/gsl_mode.h: No such file or directory
ellint.c: No such file or directory
===>cover branch res:0
KTest : /home/aaa/fp-solver/analysis/benchmark3/algorithm/algorithm&gsl_sf_ellint_Kcomp_e&jfs&bfs_output/test000002.ktest
KLEE-REPLAY: klee_assume(0)!
KLEE-REPLAY: NOTE: Test file: /home/aaa/fp-solver/analysis/benchmark3/algorithm/algorithm&gsl_sf_ellint_Kcomp_e&jfs&bfs_output/test000001.ktest
KLEE-REPLAY: NOTE: Arguments: "./gsl_sf_ellint_Kcomp_e" 
KLEE-REPLAY: NOTE: Storing KLEE replay files in /tmp/klee-replay-YZBaVB
KLEE-REPLAY: NOTE: EXIT STATUS: NORMAL (1 seconds)
KLEE-REPLAY: NOTE: removing /tmp/klee-replay-YZBaVB
===>ktest_time_log: /home/aaa/fp-solver/analysis/benchmark3/algorithm/algorithm&gsl_sf_ellint_Kcomp_e&jfs&bfs_output/test000002.time
../gsl/gsl_mode.h: No such file or directory
ellint.c: No such file or directory
===>cover line res:50.0 , 50
../gsl/gsl_mode.h: No such file or directory
ellint.c: No such file or directory
===>cover branch res:16
...... # some info
```

Coverage information can be found in `res_all.txt`. The three columns are the name of benchmark, the coverage of analyzed function, and the execution time, respectively.

```sh
$ cat res_all.txt
```

```
algorithm&gsl_sf_ellint_Kcomp_e&jfs&bfs_output/ , 92.85714285714286 , 57, 18, 2172
```

Coverage trend information can be found in `cov_trend.txt`. `TestCase` is the benchmark and configuration information. Each row below has three columns showing the number of lines of code covered, coverage and execution time.

```sh
$ cat cov_trend.txt
```

```
=== TestCase : algorithm&gsl_sf_ellint_Kcomp_e&jfs&bfs_output/
0, 14.285714285714285 , 2, 0
1, 50.0 , 50, 16
1, 50.0 , 50, 16
33, 50.0 , 50, 16
34, 50.0 , 50, 16
68, 50.0 , 50, 16
68, 50.0 , 50, 16
70, 50.0 , 50, 16
134, 50.0 , 50, 16
136, 50.0 , 50, 16
202, 92.85714285714286 , 57, 18
364, 92.85714285714286 , 57, 18
366, 92.85714285714286 , 57, 18
591, 92.85714285714286 , 57, 18
784, 92.85714285714286 , 57, 18
913, 92.85714285714286 , 57, 18
1264, 92.85714285714286 , 57, 18
1425, 92.85714285714286 , 57, 18
1649, 92.85714285714286 , 57, 18
1714, 92.85714285714286 , 57, 18
1970, 92.85714285714286 , 57, 18
=== End
```



## Run in 50 parallel

The machine used in our experiments has 80 cores and 192GB memory. Before running the script, please select the appropriate machine and complete parameter configuration.

The execution time and solving time settings in the `run_solver.sh` script are 3600s and 30s respectively. This is a long execution time, and you can enter the script and modify it to your own needs.

```sh
$ vi run_solver.sh

......
MAX_EXE_TIME=3600
SOLVER_TIME=30
......
```

You can also modify the parallel quantity in the script:

```sh
$ vi multi_process.sh

......
pool = multiprocessing.Pool(processes=50) # parallel of 50
......
```

Then you can use `nohup python3 multi_process.py &` to execute all benchmarks in parallel.

```sh
$ nohup python3 multi_process.py &
```

If there are 50 programs in parallel, it may take 155 hours to get the results.



# Obtain experimental data

To analyze the resulting experiment results, you can run the automated scripts under the `/home/aaa/analysis/res_end`, `/home/aaa/analysis/res_trend` and `/home/aaa/analysis/found_bug` paths.

### Get the table about lines of covered code and the numbers of covered branches

```sh
cd /home/aaa/analysis/res_end
python3 get_lines_branches.py
```

### Get the figures about the trend of code coverage and branch coverage

```sh
cd /home/aaa/analysis/res_trend
python3 plot_linees.py
python3 plot_branches.py
```

### Get the table about the number of detected exceptions

```
cd /home/aaa/analysis/found_bug
python3 compute_bugs.py
```

