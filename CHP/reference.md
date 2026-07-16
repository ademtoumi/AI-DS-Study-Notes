# Exam Reference: Key OpenMP & MPI Patterns

---

## OpenMP Patterns

### 1. Atomic Operations
```c
#pragma omp atomic
shared_var++;
```
- **Use when:** Multiple threads update a simple shared variable (e.g., counters).
- **Why:** Prevents race conditions for simple operations.
- **Parameters:**
  - No explicit parameters. The statement following `#pragma omp atomic` must be a simple update (e.g., `++`, `--`, `+=`, etc.) on a shared variable.

### 2. Critical Sections
```c
#pragma omp critical
{
    // Only one thread at a time
}
```
- **Use when:** Updating shared variables with complex logic (e.g., finding max/min).
- **Why:** Ensures exclusive access to the block.
- **Parameters:**
  - Optionally, a name: `#pragma omp critical(name)`. This allows multiple independent critical sections.

### 3. Parallel For
```c
#pragma omp parallel for
for (int i = 0; i < N; i++) {
    // Work
}
```
- **Use when:** Loop iterations are independent.
- **Why:** Distributes work across threads for speedup.
- **Parameters:**
  - `N`: The number of loop iterations. Should be an integer.
  - Optional OpenMP clauses: `shared(var)`, `private(var)`, `schedule(type, chunk)`, etc.

### 4. Sections
```c
#pragma omp parallel sections
{
    #pragma omp section
    { /* Task 1 */ }
    #pragma omp section
    { /* Task 2 */ }
}
```
- **Use when:** Different tasks can run in parallel.
- **Why:** Each section runs on a separate thread.
- **Parameters:**
  - No explicit parameters. Each `section` block contains code to be run by a thread.

### 5. Locks
```c
omp_lock_t lock;
omp_init_lock(&lock);
omp_set_lock(&lock);
// Critical work
omp_unset_lock(&lock);
omp_destroy_lock(&lock);
```
- **Use when:** You need explicit control over access to a resource.
- **Why:** Useful for custom synchronization (e.g., readers-writers).
- **Parameters:**
  - `omp_lock_t *lock`: A pointer to a lock variable. Must be initialized with `omp_init_lock` before use.

### 6. Scheduling
```c
#pragma omp for schedule(static, chunk)
#pragma omp for schedule(dynamic, chunk)
```
- **Use when:** Workload per iteration varies (dynamic) or is uniform (static).
- **Why:** Controls how loop iterations are assigned to threads.
- **Parameters:**
  - `static` or `dynamic`: Scheduling type.
  - `chunk`: (Optional) Number of iterations per chunk (integer).

### 7. Thread Info
```c
omp_get_thread_num(); // Thread ID
omp_get_num_threads(); // Total threads
```
- **Use when:** Debugging, logging, or assigning thread-specific work.
- **Parameters:**
  - None. These functions take no arguments and return an integer.

---

## MPI Patterns

### 1. Initialization & Finalization
```c
MPI_Init(&argc, &argv);
// ... MPI code ...
MPI_Finalize();
```
- **Use when:** Every MPI program. Always initialize and finalize.
- **Parameters:**
  - `int *argc, char ***argv`: Pointers to the command-line argument count and values. Pass the addresses from `main`.

### 2. Rank & Size
```c
int rank, size;
MPI_Comm_rank(MPI_COMM_WORLD, &rank);
MPI_Comm_size(MPI_COMM_WORLD, &size);
```
- **Use when:** Need to know which process you are and how many there are.
- **Parameters:**
  - `MPI_Comm comm`: Communicator (usually `MPI_COMM_WORLD`).
  - `int *rank`/`int *size`: Address of an integer to store the result.

### 3. Point-to-Point Communication
```c
// Send
MPI_Send(&data, count, MPI_INT, dest, tag, MPI_COMM_WORLD);
// Receive
MPI_Recv(&data, count, MPI_INT, source, tag, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
```
- **Use when:** Sending data from one process to another.
- **Why:** Fundamental for distributed algorithms.
- **Parameters:**
  - `void *buf`: Address of data to send/receive.
  - `int count`: Number of elements.
  - `MPI_Datatype datatype`: Type of data (e.g., `MPI_INT`).
  - `int dest`/`int source`: Destination/source rank.
  - `int tag`: Message tag (integer, must match between send/recv).
  - `MPI_Comm comm`: Communicator (usually `MPI_COMM_WORLD`).
  - `MPI_Status *status`: Status object for receive (can use `MPI_STATUS_IGNORE`).

### 4. Processor Name & Timing
```c
char name[MPI_MAX_PROCESSOR_NAME];
int len;
MPI_Get_processor_name(name, &len);
double t1 = MPI_Wtime();
// ...
double t2 = MPI_Wtime();
printf("Elapsed: %f", t2-t1);
```
- **Use when:** Debugging, benchmarking, or logging.
- **Parameters:**
  - `char *name`: Buffer to store the processor name.
  - `int *len`: Address of an integer to store the length of the name.
  - `MPI_Wtime()`: No parameters; returns current wall-clock time as `double`.

### 5. Multi-Stage Pipeline (Example)
- **Pattern:** Several processes send data to a manager, which aggregates and forwards results through stages.
- **Use when:** Data processing pipelines, distributed workflows.
- **Parameters:**
  - Use the same parameters as in point-to-point communication for each stage.

---

## Practical Advice
- **Race Conditions:** Use `atomic`, `critical`, or locks to protect shared data.
- **Parallel Loops:** Use `parallel for` for independent iterations. Avoid shared state unless protected.
- **Scheduling:** Use `dynamic` for irregular workloads, `static` for regular ones.
- **MPI Send/Recv:** Always match sends and receives (same tag, type, and communicator).
- **Debugging:** Print thread/process IDs to understand parallel execution.
- **Readers-Writers:** Use locks and a turn variable to alternate access.
- **Sections:** Use for heterogeneous parallel tasks.

---

## Quick Code Snippets
- **OpenMP Parallel Region:**
  ```c
  #pragma omp parallel num_threads(N)
  {
      // Parallel work
  }
  ```
  - `N`: Number of threads (integer).
- **MPI Basic Program:**
  ```c
  #include <mpi.h>
  int main(int argc, char** argv) {
      MPI_Init(&argc, &argv);
      // ...
      MPI_Finalize();
  }
  ```
  - `argc, argv`: Command-line arguments from `main`.

---

**Review these patterns, parameters, and snippets before your exam for quick recall of how and when to use each parallel programming construct!** 