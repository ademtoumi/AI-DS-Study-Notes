# Line-by-Line Annotated Explanations of C Source Files

---

## 1. `atomic.c`

```c
#include <stdio.h>
#include <omp.h>
```
*Includes the standard I/O and OpenMP headers. `stdio.h` is for input/output functions, and `omp.h` enables OpenMP parallel programming support.*

**Parameters:**
- `#include <omp.h>`: No parameters, but required to use OpenMP directives and functions.

```c
int main() {
    int i = 0;
```
*Defines the main function and initializes an integer `i` to 0. This variable will be shared among threads.*

```c
#pragma omp parallel num_threads(9)
{
```
*Starts a parallel region with 9 threads. Each thread will execute the code inside the block.*

**Parameters:**
- `num_threads(9)`: Specifies the number of threads to use in the parallel region. Pass an integer value.

```c
#pragma omp atomic
i++;
```
*The `#pragma omp atomic` directive ensures that the increment operation on `i` is performed atomically, preventing race conditions. This is crucial when multiple threads update a shared variable.*

**Parameters:**
- No explicit parameters. The statement following `#pragma omp atomic` must be a simple update (e.g., `++`, `--`, `+=`, etc.) on a shared variable.

```c
printf("nombre de thread: %d thread id: %d\n ", i, omp_get_thread_num());
```
*Prints the current value of `i` and the thread's ID. `omp_get_thread_num()` returns the unique ID of the calling thread.*

**Parameters:**
- `printf`: Format string and variables to print.
- `omp_get_thread_num()`: No parameters; returns the calling thread's ID (int).

```c
}
}
```
*Closes the parallel region and the main function.*

**Where to use:**
- Use `#pragma omp atomic` when you need to update a shared variable in parallel, but the update is a simple operation (like increment/decrement). For more complex updates, use `#pragma omp critical`.
- Useful in counters, accumulators, or any scenario where a simple shared variable is updated by multiple threads.

---

## 2. `exo2exam_tp.c`

```c
#include <stdio.h>
#include <mpi.h>
```
*Includes standard I/O and MPI headers. `mpi.h` is required for MPI functions.*

**Parameters:**
- `#include <mpi.h>`: No parameters, but required for all MPI functions.

```c
int main(int argc, char** argv) {
    MPI_Init(&argc, &argv);
```
*Defines the main function and initializes the MPI environment. Always call `MPI_Init` before any other MPI function.*

**Parameters:**
- `MPI_Init(int *argc, char ***argv)`: Pass the addresses of `argc` and `argv` from `main`.

```c
    int rank, size;
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
```
*Declares variables for the process rank and size. `MPI_Comm_rank` gets the rank (ID) of the calling process.*

**Parameters:**
- `MPI_Comm_rank(MPI_Comm comm, int *rank)`: `comm` is usually `MPI_COMM_WORLD`. `rank` is a pointer to an int to store the process rank.

```c
    int value = 0;
    int received;
```
*Declares variables for message passing.*

```c
    if (rank == 1) { // J1
        int Pij = 2;
        MPI_Send(&Pij, 1, MPI_INT, 4, 0, MPI_COMM_WORLD); // to M1
    } else if (rank == 2) { // J2
        int Pij = 3;
        MPI_Send(&Pij, 1, MPI_INT, 4, 0, MPI_COMM_WORLD); // to M1
    } else if (rank == 3) { // J3
        int Pij = 5;
        MPI_Send(&Pij, 1, MPI_INT, 4, 0, MPI_COMM_WORLD); // to M1
```
*Ranks 1, 2, and 3 each send a different integer to rank 4. This simulates three jobs sending results to a manager.*

**Parameters for `MPI_Send`:**
- `void *buf`: Address of data to send.
- `int count`: Number of elements to send (here, 1).
- `MPI_Datatype datatype`: Type of data (e.g., `MPI_INT`).
- `int dest`: Destination rank (here, 4).
- `int tag`: Message tag (here, 0).
- `MPI_Comm comm`: Communicator (usually `MPI_COMM_WORLD`).

```c
    } else if (rank == 4) { // M1
        value = 0;
        for (int i = 0; i < 3; i++) {
            MPI_Recv(&received, 1, MPI_INT, MPI_ANY_SOURCE, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
            value += received;
        }
        MPI_Send(&value, 1, MPI_INT, 5, 0, MPI_COMM_WORLD); // to M2
```
*Rank 4 (manager) receives three integers from any source, sums them, and sends the result to rank 5.*

**Parameters for `MPI_Recv`:**
- `void *buf`: Address to store received data.
- `int count`: Number of elements to receive (here, 1).
- `MPI_Datatype datatype`: Type of data (e.g., `MPI_INT`).
- `int source`: Source rank (or `MPI_ANY_SOURCE`).
- `int tag`: Message tag (here, 0).
- `MPI_Comm comm`: Communicator.
- `MPI_Status *status`: Status object (or `MPI_STATUS_IGNORE`).

**Parameters for `MPI_Send`:** (see above)

```c
    } else if (rank == 5) { // M2
        MPI_Recv(&value, 1, MPI_INT, 4, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        value += 10; // M2 processing (dummy logic)
        MPI_Send(&value, 1, MPI_INT, 6, 0, MPI_COMM_WORLD); // to M3
```
*Rank 5 receives the sum, adds 10, and sends it to rank 6. This simulates a processing stage.*

**Parameters:** (see above for `MPI_Recv` and `MPI_Send`)

```c
    } else if (rank == 6) { // M3
        MPI_Recv(&value, 1, MPI_INT, 5, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        value += 100; // M3 processing
        MPI_Send(&value, 1, MPI_INT, 0, 0, MPI_COMM_WORLD); // to root
```
*Rank 6 receives the value, adds 100, and sends it to rank 0 (root). Another processing stage.*

**Parameters:** (see above for `MPI_Recv` and `MPI_Send`)

```c
    } else if (rank == 0) { // root
        MPI_Recv(&value, 1, MPI_INT, 6, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        printf("Final result at root: %d\n", value);
    }
```
*Rank 0 receives the final result and prints it.*

```c
    MPI_Finalize();
    return 0;
}
```
*Finalizes the MPI environment and ends the program. Always call `MPI_Finalize` at the end of an MPI program.*

**Parameters:**
- `MPI_Finalize()`: No parameters.

**Where to use:**
- This pattern is useful for multi-stage pipelines, distributed processing, or workflows where results are aggregated and processed in stages.
- Similar approaches are used in distributed data processing, scientific computing, and parallel simulations.

---

## 3. `exo1exam_tp.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <omp.h>
#include <unistd.h> // pour sleep()
```
*Includes standard I/O, OpenMP, and POSIX sleep headers. `unistd.h` is used for the `sleep` function to simulate delays.*

**Parameters:**
- `#include <omp.h>`: No parameters, but required for OpenMP.
- `#include <unistd.h>`: No parameters, but required for `sleep()`.

```c
omp_lock_t write_lock;
int reader_count = 0;
int turn = 0; // 0 = read, 1 = write
```
*Declares a lock for writing, a counter for readers, and a variable to alternate between reading and writing.*

```c
void read_file(int id) {
    #pragma omp critical
    {
        printf("Thread Reader %d: trying to read...\n", id);
    }
```
*Defines the reader function. The `#pragma omp critical` ensures that the print statement is not interleaved between threads.*

**Parameters:**
- `int id`: Thread ID passed to the function.
- `#pragma omp critical`: Optionally, a name for the critical section.

```c
    while (turn != 0);
```
*Busy-wait until it's the turn for readers. This is a simple synchronization mechanism.*

```c
    #pragma omp atomic
    reader_count++;
```
*Atomically increments the reader count. This prevents race conditions when multiple readers start reading.*

**Parameters:**
- No explicit parameters. The statement must be a simple update on a shared variable.

```c
    #pragma omp critical
    {
        printf("Thread Reader %d: reading the file...\n", id);
    }
```
*Prints that the thread is reading. Again, critical to avoid interleaved output.*

**Parameters:**
- `printf`: Format string and variables to print.

```c
    sleep(1); // Simuler lecture
```
*Simulates reading by sleeping for 1 second.*

**Parameters:**
- `sleep(unsigned int seconds)`: Number of seconds to sleep.

```c
    #pragma omp atomic
    reader_count--;
```
*Atomically decrements the reader count when done.*

**Parameters:**
- No explicit parameters. The statement must be a simple update on a shared variable.

```c
    if (reader_count == 0)
        turn = 1; // Donner le tour à l'écriture
}
```
*If this is the last reader, gives the turn to writers.*

```c
void write_file(int id) {
    while (turn != 1);
```
*Busy-wait until it's the turn for writers.*

```c
    omp_set_lock(&write_lock);
```
*Acquires the write lock to ensure only one writer at a time.*

```c
    #pragma omp critical
    {
        printf("Thread Writer %d: writing to the file...\n", id);
    }
```
*Prints that the thread is writing. Critical section for clean output.*

```c
    sleep(1); // Simuler écriture
```
*Simulates writing by sleeping for 1 second.*

```c
    omp_unset_lock(&write_lock);
```
*Releases the write lock.*

```c
    turn = 0; // Donner le tour à la lecture
}
```
*Gives the turn back to readers.*

```c
int main() {
    omp_init_lock(&write_lock);
```
*Initializes the write lock.*

**Parameters:**
- `omp_init_lock(omp_lock_t *lock)`: Pass the address of the lock variable.

```c
    #pragma omp parallel num_threads(6)
    {
        int tid = omp_get_thread_num();
        if (tid % 2 == 0) {
            read_file(tid);
        } else {
            write_file(tid);
        }
    }
```
*Creates 6 threads. Even threads are readers, odd threads are writers. Each calls the appropriate function.*

**Parameters:**
- `num_threads(6)`: Number of threads to use in the parallel region.
- `omp_get_thread_num()`: No parameters; returns thread ID.

```c
    omp_destroy_lock(&write_lock);
    return 0;
}
```
*Destroys the lock and ends the program.*

**Parameters:**
- `omp_destroy_lock(omp_lock_t *lock)`: Pass the address of the lock variable.

**Where to use:**
- This pattern is used in readers-writers problems, file/database access, and any scenario where you need to alternate between multiple readers and exclusive writers.
- Similar approaches are used in operating systems, databases, and concurrent data structures.

---

## 4. `MPI-exo5.c`

```c
#include <stdio.h>
#include "mpi.h"
```
*Includes standard I/O and MPI headers.*

**Parameters:**
- `#include <mpi.h>`: No parameters, but required for all MPI functions.

```c
int main(int argc, char** argv){
    int rank;
    int total_processes;
```
*Defines the main function and declares variables for the process rank and total number of processes.*

```c
    MPI_Init(NULL,NULL);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &total_processes);
```
*Initializes MPI, gets the rank and the total number of processes.*

**Parameters:**
- `MPI_Init(int *argc, char ***argv)`: Pass `NULL` if you don't need to process command-line arguments.
- `MPI_Comm_rank(MPI_Comm comm, int *rank)`: `comm` is usually `MPI_COMM_WORLD`. `rank` is a pointer to an int to store the process rank.
- `MPI_Comm_size(MPI_Comm comm, int *size)`: `comm` is usually `MPI_COMM_WORLD`. `size` is a pointer to an int to store the number of processes.

```c
int number;
if (rank == 0) {
    number = -1;
    MPI_Send(&number, 1, MPI_INT, 1, 0, MPI_COMM_WORLD);
} else if (rank == 1) {
    MPI_Recv(&number, 1, MPI_INT, 0, 0, MPI_COMM_WORLD,
             MPI_STATUS_IGNORE);
    printf("Process 1 received number %d from process 0\n",
           number);
}
```
*If rank 0, sends an integer to rank 1. If rank 1, receives the integer and prints it. Demonstrates basic point-to-point communication.*

**Parameters for `MPI_Send` and `MPI_Recv`:**
- See above in exo2exam_tp.c for details.

```c
    MPI_Finalize();
    return 0;
```
*Finalizes MPI and ends the program.*

**Parameters:**
- `MPI_Finalize()`: No parameters.

```c
/*
MPI_SHORT  short int
MPI_INT    int
MPI_LONG   long int
MPI_LONG_LONG  long long int
MPI_UNSIGNED_CHAR  unsigned char
MPI_UNSIGNED_SHORT unsigned short int
MPI_UNSIGNED  unsigned int
MPI_UNSIGNED_LONG  unsigned long int
MPI_UNSIGNED_LONG_LONG unsigned long long int
MPI_FLOAT  float
MPI_DOUBLE double
MPI_LONG_DOUBLE long double
MPI_BYTE   char
*/
```
*Lists common MPI data types for reference.*

**Where to use:**
- Use this pattern for simple message passing between two processes, such as in distributed control, signaling, or data transfer.
- Similar approaches are used in distributed systems, parallel simulations, and client-server models.

---

## 5. `MPI-exo1.c`

```c
#include <stdio.h>
#include "mpi.h"
int main (int argc, char * argv[]) {
double debut_temps, fin_temps;
int nom_long, nbr_proc,num_proc, rang;
char nom_proc [MPI_MAX_PROCESSOR_NAME];
MPI_Init(NULL, NULL);
debut_temps = MPI_Wtime();
MPI_Comm_size(MPI_COMM_WORLD, &nbr_proc);
MPI_Comm_rank (MPI_COMM_WORLD, &rang);

MPI_Get_processor_name (nom_proc, &nom_long);
printf("processeur numéro %d sur la machine %s parmi %d processeurs \n", rang,nom_proc,nbr_proc);
fin_temps = MPI_Wtime();
printf ("temps écoulé sur %d =  %f  \n", num_proc, fin_temps - debut_temps);
MPI_Finalize ();
return 0;
}
```
*This program demonstrates how to get the processor name, rank, and number of processes in MPI, and how to measure elapsed time. `MPI_Wtime()` is used for timing. The output shows which process is running on which machine and the elapsed time.*

**Parameters:**
- `MPI_Init(int *argc, char ***argv)`: Pass `NULL` if you don't need to process command-line arguments.
- `MPI_Comm_size(MPI_Comm comm, int *size)`: `comm` is usually `MPI_COMM_WORLD`. `size` is a pointer to an int to store the number of processes.
- `MPI_Comm_rank(MPI_Comm comm, int *rank)`: `comm` is usually `MPI_COMM_WORLD`. `rank` is a pointer to an int to store the process rank.
- `MPI_Get_processor_name(char *name, int *resultlen)`: `name` is a buffer to store the processor name. `resultlen` is a pointer to an int to store the length of the name.
- `MPI_Wtime()`: No parameters; returns current wall-clock time as `double`.
- `MPI_Finalize()`: No parameters.

**Where to use:**
- Use this pattern for benchmarking, debugging, or logging in distributed MPI programs.
- Similar approaches are used in cluster management, distributed monitoring, and performance analysis.

---

## 6. `exo1.c`

```c
#include<stdio.h>
#include<stdlib.h>
#include<omp.h>
int main (int argc, char const *argv[]){
  int n,a=0;
  #pragma omp parallel for shared(a)
   for(n=0;n<8;n++){
    printf("Element %d traité par le thread %d a=%d\n\n",n,omp_get_thread_num(),a);
    a++; 
  }
}
```
*This program demonstrates a parallel for loop in OpenMP with a shared variable. The variable `a` is incremented by multiple threads without synchronization, which can lead to a race condition. Each thread prints its thread ID and the value of `a`.*

**Parameters:**
- `#pragma omp parallel for shared(a)`: `shared(a)` makes `a` shared among all threads. You can also use `private(var)` for thread-local variables.
- `omp_get_thread_num()`: No parameters; returns thread ID.
- `printf`: Format string and variables to print.

**Where to use:**
- Use this pattern for parallelizing independent loop iterations. Be careful with shared variables; use `atomic` or `critical` if updates must be synchronized.
- Similar approaches are used in parallel data processing, simulations, and scientific computing.

---

## 7. `sections.c`

```c
#include <omp.h>
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char* argv[])
{
int i=0;
omp_set_num_threads (2);
#pragma  omp  parallel sections
{
#pragma  omp section
{
printf("Nous sommes %d threads, je suis thread %d.i: %d \n", omp_get_num_threads(), omp_get_thread_num(),i);
}
#pragma  omp section
{
printf("Nous sommes %d threads, je suis thread %d.i: %d \n", omp_get_num_threads(), omp_get_thread_num(),i);
}
}}
```
*This program demonstrates the use of OpenMP `sections` to execute different code blocks in parallel. Each section is executed by a different thread. Useful for parallelizing tasks that are independent but not identical.*

**Parameters:**
- `omp_set_num_threads(int n)`: Sets the number of threads for subsequent parallel regions.
- `#pragma omp parallel sections`: No explicit parameters, but you can use `num_threads(n)` to specify thread count.
- `#pragma omp section`: No parameters; marks a block to be executed by a thread.
- `omp_get_num_threads()`, `omp_get_thread_num()`: No parameters; return total threads and thread ID, respectively.
- `printf`: Format string and variables to print.

**Where to use:**
- Use `sections` when you have different tasks to run in parallel, such as in pipelined computations or multi-stage processing.
- Similar approaches are used in multimedia processing, scientific workflows, and concurrent servers.

---

## 8. `for2.c`

```c
#include <omp.h>
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char* argv[])
{
omp_set_num_threads(3);
# pragma  omp  parallel
for( int i = 0; i < 3; i++ ) {
printf( "i = %d: NThread = %d\n", i, omp_get_thread_num () );
}
}
```
*This program shows a simple parallel region in OpenMP. Each thread executes the entire loop independently, printing the loop index and its thread ID. This is different from a parallel for, where iterations are divided among threads.*

**Parameters:**
- `omp_set_num_threads(int n)`: Sets the number of threads for subsequent parallel regions.
- `#pragma omp parallel`: No explicit parameters, but you can use `num_threads(n)` to specify thread count.
- `omp_get_thread_num()`: No parameters; returns thread ID.
- `printf`: Format string and variables to print.

**Where to use:**
- Use this pattern for independent work in each thread, such as initializing thread-local data or running independent tasks.
- Similar approaches are used in task parallelism and thread-based servers.

---

## 9. `dynamique.c`

```c
#include <omp.h>
#include <stdio.h>
#include <stdlib.h>
void dyn(int j,int nbr){
printf( "j : %d, nbr : %d\n", j, nbr  );
omp_set_num_threads(nbr);
# pragma  omp  parallel
for( int i = 0; i < j; i++ ) {
printf( "i = %d: NThread = %d\n", i, omp_get_thread_num () );
}
}
int main(int argc, char* argv[])
{
dyn(5,3);
}
```
*This program demonstrates dynamic setting of the number of threads and parallel loop execution. The function `dyn` sets the number of threads and runs a parallel loop. Useful for scenarios where the degree of parallelism is determined at runtime.*

**Parameters:**
- `dyn(int j, int nbr)`: `j` is the number of loop iterations, `nbr` is the number of threads.
- `omp_set_num_threads(int n)`: Sets the number of threads for subsequent parallel regions.
- `#pragma omp parallel`: No explicit parameters, but you can use `num_threads(n)`.
- `omp_get_thread_num()`: No parameters; returns thread ID.
- `printf`: Format string and variables to print.

**Where to use:**
- Use this pattern when the number of threads should be set dynamically, such as in adaptive algorithms or user-configurable parallelism.
- Similar approaches are used in scientific computing and high-performance applications.

---

## 10. `for.c`

```c
#include <omp.h>
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char* argv[])
{
int i;
omp_set_num_threads (10);
    # pragma omp parallel
for(i=0;i<10;i++) {
printf("Nous sommes %d threads, je suis thread %d.i: %d \n", omp_get_num_threads(), omp_get_thread_num(),i);
}
}
```
*This program shows a parallel region in OpenMP with a loop. Each thread executes the entire loop independently. Useful for thread-local initialization or repeated tasks.*

**Parameters:**
- `omp_set_num_threads(int n)`: Sets the number of threads for subsequent parallel regions.
- `#pragma omp parallel`: No explicit parameters, but you can use `num_threads(n)`.
- `omp_get_num_threads()`, `omp_get_thread_num()`: No parameters; return total threads and thread ID, respectively.
- `printf`: Format string and variables to print.

**Where to use:**
- Use this pattern for repeated work in each thread, or when each thread must process all data.
- Similar approaches are used in simulations and multi-threaded servers.

---

## 11. `crit.c`

```c
#include <omp.h>
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
int main()
{
    int i;
    int max;
    int tab[9];
 srand(time(0));
    for (i = 0; i < 9; i++)
    {
        tab[i] = rand();
        printf("%d\n", tab[i]);
    }
    #pragma omp parallel for num_threads(9)
        for (i = 0; i < 9; i++)
        {
	#pragma omp critical
                   { if (tab[i] > max)                
                        max = tab[i];}
        }
    printf("max = %d\n", max);
}
```
*This program demonstrates the use of the OpenMP `critical` directive to find the maximum value in an array in parallel. Each thread checks if its value is greater than the current max inside a critical section. Useful for reductions where atomic is not sufficient.*

**Parameters:**
- `#pragma omp parallel for num_threads(9)`: `num_threads(9)` specifies the number of threads.
- `#pragma omp critical`: Optionally, a name for the critical section.
- `printf`: Format string and variables to print.

**Where to use:**
- Use `critical` for complex updates to shared variables, such as finding a maximum or minimum.
- Similar approaches are used in parallel reductions, statistics, and concurrent data structures.

---

## 12. `numthread.c`

```c
#include <omp.h>
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char* argv[])
{
//omp_set_num_threads(10);
    # pragma omp parallel num_threads(8)
    {
        printf("Nous sommes %d threads, je suis thread %d.\n", omp_get_num_threads(), omp_get_thread_num());
    }
}
```
*This program shows how to specify the number of threads in an OpenMP parallel region. Each thread prints the total number of threads and its thread ID. Useful for debugging and learning OpenMP.*

**Parameters:**
- `#pragma omp parallel num_threads(8)`: `num_threads(8)` specifies the number of threads.
- `omp_get_num_threads()`, `omp_get_thread_num()`: No parameters; return total threads and thread ID, respectively.
- `printf`: Format string and variables to print.

**Where to use:**
- Use this pattern to control parallelism explicitly, such as in resource-constrained environments.
- Similar approaches are used in parallel libraries and frameworks.

---

## 13. `dist.c`

```c
#include <stdio.h>
#include <omp.h>
int main() {
   int i = 0;
   #pragma omp parallel num_threads(9)
{
#pragma omp for schedule(static,2)
  for(i=0; i<9; i++)
printf("nombre de thread: %d thread id: %d\n ", i,omp_get_thread_num ());
#pragma omp for schedule(dynamic,2)
  for(i=0; i<9; i++)
printf("nombre : %d thread id: %d\n ", i,omp_get_thread_num ());
}
}
```
*This program demonstrates static and dynamic scheduling in OpenMP parallel for loops. Static scheduling assigns iterations in blocks, dynamic scheduling assigns them as threads become available. Useful for load balancing.*

**Parameters:**
- `#pragma omp parallel num_threads(9)`: `num_threads(9)` specifies the number of threads.
- `#pragma omp for schedule(static,2)`: `static` is the scheduling type, `2` is the chunk size.
- `#pragma omp for schedule(dynamic,2)`: `dynamic` is the scheduling type, `2` is the chunk size.
- `omp_get_thread_num()`: No parameters; returns thread ID.
- `printf`: Format string and variables to print.

**Where to use:**
- Use static scheduling for uniform workloads, dynamic for irregular workloads.
- Similar approaches are used in scientific computing, simulations, and parallel data processing.

---

## 14. `synchr.c`

```c
#include <stdio.h>
#include <omp.h>
int main() {
   int i = 0;
int n=10;
   #pragma omp parallel num_threads(9)
{
#pragma omp for schedule(static,2)
  for(i=0; i<n; i++)
printf("nombre de thread: %d thread id: %d\n ", i,omp_get_thread_num ());
#pragma omp for schedule(dynamic,3)
  for(i=0; i<n; i++)
printf("nbr de thread: %d thread id: %d\n ", i,omp_get_thread_num ());
}
}
```
*This program shows the difference between static and dynamic scheduling in OpenMP with different chunk sizes. Useful for understanding and tuning parallel loop performance.*

**Parameters:**
- `#pragma omp parallel num_threads(9)`: `num_threads(9)` specifies the number of threads.
- `#pragma omp for schedule(static,2)`: `static` is the scheduling type, `2` is the chunk size.
- `#pragma omp for schedule(dynamic,3)`: `dynamic` is the scheduling type, `3` is the chunk size.
- `omp_get_thread_num()`: No parameters; returns thread ID.
- `printf`: Format string and variables to print.

**Where to use:**
- Use this pattern to experiment with scheduling strategies for performance tuning.
- Similar approaches are used in high-performance computing and parallel analytics. 