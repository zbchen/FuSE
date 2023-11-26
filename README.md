# Pre install docker environment

We have deployed the experimental environment on docker. Please pre install docker on your host

# Download

Download docker image:
```sh
$ sudo docker pull dockeryangxu/fpse:1.0
```

If the image is pulled successfully, please check there is an image named apsecpaper/apsecpaper exists.
```sh
$ sudo docker images
REPOSITORY          TAG       IMAGE ID       CREATED       SIZE
dockeryangxu/fpse   1.0       5b15cd41514d   2 hours ago   28.1GB
```


Start to run the container in interactive mode.
```sh
$ sudo docker run -it dockeryangxu/fpse:1.0 bash
```

# Run a example

Enter the directory `/home/aaa/analysis3/test` and list all contents.

```sh
$ cd /home/aaa/analysis3/test
```

The following code is the example mentioned in our paper. The source code for the example is in `example/simple.c`.

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
$ ./run_solver.sh example simple smt-dreal bfs
```

Less than 5 seconds, we can see the output at the terminal. The pathes are explored completely.
```sh

...... 

KLEE: output directory is "/home/aaa/analysis3/test/example/example&simple&smt-dreal&bfs_output"
KLEE: Using Z3 solver backend
KLEE: ERROR: (location information missing) FloatPointCheck: FP Invalid found !
KLEE: NOTE: now ignoring this error at this location
>>>Synergy-Z3 exec time: 8.099706e+01 ms
KLEE: WARNING: SMT-DREAL: Z3 solving SAT and evaluate SUCCESS !
>>>Synergy-Z3 exec time: 4.461000e-03 ms
>>>Synergy-dreal exec time: 1.747823e+00 ms
KLEE: WARNING: SMT-DREAL: DReal solving UNKNOWN with all support and remove this state !
>>>Synergy-Z3 exec time: 5.987000e-03 ms
>>>Synergy-dreal exec time: 6.644266e+00 ms
KLEE: WARNING: SMT-DReal: DReal solving SAT and evaluate FAILURE and using JFS with seeds to solve !
>>>Fuzz with seed exec time: 1.730070e+03 ms
KLEE: WARNING: FUZZ with seed: solving SAT and evaluate SUCCESS !
>>>Synergy-Z3 exec time: 5.746000e-03 ms
>>>Synergy-dreal exec time: 1.023797e+00 ms
KLEE: WARNING: SMT-DReal: DReal solving SAT and evaluate FAILURE and using JFS with seeds to solve !
>>>Fuzz with seed exec time: 1.762201e+03 ms
KLEE: WARNING: FUZZ with seed: solving SAT and evaluate SUCCESS !
>>>Synergy-Z3 exec time: 1.161700e-02 ms
>>>Synergy-dreal exec time: 1.716124e+00 ms
KLEE: WARNING: SMT-DREAL: DReal solving UNKNOWN with all support and remove this state !
>>>Synergy-Z3 exec time: 5.629000e-03 ms
>>>Synergy-dreal exec time: 2.724896e+00 ms
KLEE: WARNING: SMT-DReal: DReal solving SAT and evaluate FAILURE and using JFS with seeds to solve !
>>>Fuzz with seed exec time: 1.237251e+04 ms
KLEE: WARNING: FUZZ with seed: solving UNKNOWN evalute FAILURE and remove the state !
>>>Synergy-Z3 exec time: 5.046000e-03 ms
>>>Synergy-dreal exec time: 1.002400e-02 ms
KLEE: WARNING: SMT-DREAL: DReal solving UNKNOWN with all support and remove this state !
>>>Synergy-Z3 exec time: 1.575271e+02 ms
KLEE: WARNING: SMT-DREAL: Z3 solving UNSAT and remove the state !

KLEE: done: total instructions = 42
KLEE: done: completed paths = 3
KLEE: done: partially completed paths = 1
KLEE: done: generated tests = 4
```

If you want to analyze this program using the BVFP solver, please modify the `solver_type` parameter to `smt`/`bitwuzla`/`mathsat5`.  KLEE will take a long time (more than 30min) to finish the program (you can CTRL+C to intrupt this program), because the source code of `sin/cos/log` are not avalible.  it can not explore all pathes fastly:

```sh
$ ./run_solver.sh example simple smt bfs
...... # take a long time

KLEE: done: total instructions = 17992
KLEE: done: completed paths = 65
KLEE: done: partially completed paths = 377
KLEE: done: generated tests = 19
```

# Obtain experimental results

Our experiments were performed on a server with Intel(R) Xeon(R) Platinum 8269CY CPU (2.50GHz) and the operating system is Ubuntu 18.04 LTS. 

To obtain the results, a machine with similar CPUs is required. Moreover, our experiments were run in 54 parallel.

## Analyze a program in Benchmark

Navigate to `/home/aaa/analysis3/benchmark3` and list the contents.

```sh
$ cd /home/aaa/analysis3/benchmark3
```

If you want to obtain experimental results for a single test program, e.g., `sf/gsl_sf_airy_Ai_deriv_e.c`. For example, obtain the experimental results of `jfs+dfs`.

```sh
$ ./run_solver.sh sf gsl_sf_airy_Ai_deriv_e jfs dfs
```

After running, log and test cases are generated in the corresponding directory, you can read the log as follow:

```sh
$ vi "sf/gsl_sf_airy_Ai_deriv_e&jfs&dfs.runlog"
......
>>>JFS exec time: 1.297852e+03 ms
KLEE: WARNING: FUZZ: JFS solving SAT and evaluate SUCCESS !
KLEE: ERROR: airy_der.c:712: FloatPointCheck: Common Underflow found !
KLEE: NOTE: now ignoring this error at this location
>>>JFS exec time: 1.273386e+03 ms
KLEE: WARNING: FUZZ: JFS solving SAT and evaluate SUCCESS !
KLEE: ERROR: airy_der.c:712: FloatPointCheck: Common Accuracy found !
KLEE: NOTE: now ignoring this error at this location
>>>JFS exec time: 1.231738e+03 ms
KLEE: WARNING: FUZZ: JFS solving SAT and evaluate SUCCESS !
>>>JFS exec time: 1.288826e+03 ms
KLEE: WARNING: FUZZ: JFS solving SAT and evaluate SUCCESS !
KLEE: ERROR: airy_der.c:722: FloatPointCheck: Common Overflow found !
KLEE: NOTE: now ignoring this error at this location
......
```

We can get the coverage information by running the script:

```sh
$ cd /home/aaa/analysis3/benchmark3
$ ./repaly.sh

......# some info
     Running ==== > sf&gsl_sf_airy_Ai_deriv_e&jfs&dfs_output/
====  Replay Ktest ====
===>/home/aaa/analysis3/benchmark3/sf/sf&gsl_sf_airy_Ai_deriv_e&jfs&dfs_output/test000001.ktest
CHECK:  KTests have been generated !
===>python_res: airy_der.c
===>gcno: /home/aaa/gsl/specfunc/.libs/airy_der.gcno
gcno file is exit
KTest : /home/aaa/analysis3/benchmark3/sf/sf&gsl_sf_airy_Ai_deriv_e&jfs&dfs_output/test000001.ktest
KLEE-REPLAY: klee_assume(0)!
KLEE-REPLAY: NOTE: Test file: /home/aaa/analysis3/benchmark3/sf/sf&gsl_sf_airy_Ai_deriv_e&jfs&dfs_output/test000001.ktest
KLEE-REPLAY: NOTE: Arguments: "./gsl_sf_airy_Ai_deriv_e" 
KLEE-REPLAY: NOTE: Storing KLEE replay files in /tmp/klee-replay-SrsJ2t
KLEE-REPLAY: NOTE: EXIT STATUS: NORMAL (0 seconds)
KLEE-REPLAY: NOTE: removing /tmp/klee-replay-SrsJ2t
===>ktest_time_log: /home/aaa/analysis3/benchmark3/sf/sf&gsl_sf_airy_Ai_deriv_e&jfs&dfs_output/test000001.time
../gsl/gsl_mode.h: No such file or directory
./cheb_eval_mode.c: No such file or directory
airy_der.c: No such file or directory
===>gcov_res: File '../gsl/gsl_mode.h' Lines executed:100.00% of 1 ../gsl/gsl_mode.h:creating 'gsl_mode.h.gcov' File './cheb_eval_mode.c' Lines executed:93.33% of 15 ./cheb_eval_mode.c:creating 'cheb_eval_mode.c.gcov' File 'airy_der.c' Lines executed:5.43% of 184 airy_der.c:creating 'airy_der.c.gcov'
===>cover_res:39.130434782608695 , 25
...... # some info

```

Coverage information can be found in `res_all.txt`. The three columns are the name of benchmark, the coverage of analyzed function, and the execution time, respectively.

```sh
$ cat res_all.txt

sf&gsl_sf_airy_Ai_deriv_e&jfs&dfs_output/ , 73.91304347826086 , 53,
```

Coverage trend information can be found in `cov_trend.txt`. `TestCase` is the benchmark and configuration information. Each row below has three columns showing the number of lines of code covered, coverage and execution time.

```sh
$ cat cov_trend.txt

=== TestCase : sf&gsl_sf_airy_Ai_deriv_e&jfs&dfs_output/
34, 39.130434782608695 , 25
35, 39.130434782608695 , 25
39, 47.82608695652174 , 27
72, 47.82608695652174 , 27
75, 47.82608695652174 , 27
109, 47.82608695652174 , 27
111, 47.82608695652174 , 27
309, 73.91304347826086 , 45
377, 73.91304347826086 , 53
478, 73.91304347826086 , 53
579, 73.91304347826086 , 53
647, 73.91304347826086 , 53
=== End
```



## Run in 54 parallel

The machine used in our experiments has 104 cores and 192GB memory. Before running the script, please select the appropriate machine and complete parameter configuration.

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
pool = multiprocessing.Pool(processes=54) # parallel of 54
......
```

Then you can use `nohup python3 multi_process.py &` to execute all benchmarks in parallel.

```sh
$ nohup python3 multi_process.py &
```

If there are 54 programs in parallel, it may take 144 hours to get the results.

