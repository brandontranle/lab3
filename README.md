# Hash Hash Hash
This lab tests three different implementations of hash tables that abide to concurrency safety. The three implementations are labeled seperately as <code>base</code>, <code>v1</code>, and <code>v2</code>. <code> base</code> is a default configuration given in the skeleton, that does not use parallelism and merely runs the hash table in sequential order. The <code>v1</code> implementation creates a severe bottleneck as a tradeoff for safety, which stems from a "course-grained" locking mechanism. The <code>v2</code> implementation leads away from this and utilizes a "fine-grained" locking mechanism that assigns a mutex to every entry, thus allowing threads to access multiple segements of the hash table simultaneously. 

## Building
```shell
make
```

## Running
```shell
./hash-table-tester -t 8 -s 50000
```

#### Explanation
The <code>-t</code> flag refers to testing with a number of threads. 
A number following this line will indicate how many threads are expected to run.

The <code>-s</code> flag refers to enabling the number of hash table entires must be added for the test.
A number following this line will indicate how many entries are added. In all of these tests, 50000 will be used. 

## First Implementation
In the <code> hash_table_v1_add_entry </code> function, I utilized a global mutex that was used to lock the critical section in the function. To begin this implementation, I initialized a global mutex variable (a lock), which was used in the <code> hash_table_v1_add_entry </code> function and the <code> hash_table_v1_destroy </code> function. Within the add entry, I locked the critical section which is responsible for adding entries via defining the list entry, head entry, and hash table entry. Within this function, the lock is unlocked after the entry was constructed and added. 

In the <code> hash_table_v1_destroy </code> function, the lock was destroyed upon freeing the hash table via the "pthread_mutex_destroy" built-in function. This is done to ensure there are no memory leaks.

### Performance
```shell
/*the code was tested three times using this line*/
./hash-table-tester -t 8 -s 50000
```
#### Results
```shell
Generation: 107,800 usec
Hash table base: 1,624,771 usec
  - 0 missing
Hash table v1: 2,802,588 usec
  - 0 missing
```

Version 1 is a little slower/faster than the base version, as there is severe thread overhead with the lock. Version 1 of the hash table implementation introduces a critical "bottleneck" due to its reliance on a single global lock for managing access to the entire hash table. The more threads that attempt to access the hash table, the more pronounced the contention becomes, leading to a situation where additional threads do not contribute to increased performance, and decrease overall throughput due to overheads related to lock contention and thread management. 

## Second Implementation
In the <code> hash_table_v2_add_entry </code> function, I began locking the critical section that involves defining a <code>list_head</code> and <code> list_entry</code> after the hash_entry was defined. The lock is unlocked after adding the defined entry to the list. If it already exists, however, the value of the entry is set and the lock is unlocked. 

Additional modifications were made for optimization and correctness. To begin, a mutex was assigned and intialized to each <code>hash_table_entry</code>. This reduces the contention that spurred from implementation 1, and thus allows parallelism on different segments of the hash table. In other words, threads do not have to wait for one another to use a global lock, rather they can now all work simultaneously without conflicitng between each other. With my implementation, I modifed the mutex cleanup from implementation 1 to correctly clean out each mutex that was intialized in the beginning, thus preventing memory leaks. 

### Performance

The following test were performed using 8 cores:
```
./hash-table-tester -t 8 -s 50000
```
#### Test 1
```shell
Generation: 87,880 usec
Hash table base: 1,608,304 usec
  - 0 missing
Hash table v1: 2,224,283 usec
  - 0 missing
Hash table v2: 214,676 usec
  - 0 missing
```

#### Test 2
```shell
Generation: 83,825 usec
Hash table base: 1,357,712 usec
  - 0 missing
Hash table v1: 2,215,107 usec
  - 0 missing
Hash table v2: 256,973 usec
  - 0 missing
```

Test 3
```shell
Generation: 86,889 usec
Hash table base: 1,484,637 usec
  - 0 missing
Hash table v1: 2,101,980 usec
  - 0 missing
Hash table v2: 223,768 usec
  - 0 missing
```

There is a significant speedup improvement with the second implementations parallelism approach as the average speedup is 6.47x compared to the base when testing 8 cores and threads. The reason why the base and first implemenetation's iteration of this assignment is slow is simply because they craete bottlenecks by using a sequential locking system despite there being safety. This means that the previous iterations have threads competing for hash_table entries and lock access just for the sake of not colliding with each other. 

### Additional Analysis
#### Additional Test 1
```shell
./hash-table-tester -t 6 -s 50000
Generation: 65,587 usec
Hash table base: 621,442 usec
  - 0 missing
Hash table v1: 1,052,327 usec
  - 0 missing
Hash table v2: 130,863 usec
  - 0 missing
```
#### Additional Test 2
```shell
./hash-table-tester -t 4 -s 50000
Generation: 42,159 usec
Hash table base: 257,224 usec
  - 0 missing
Hash table v1: 416,507 usec
  - 0 missing
Hash table v2: 77,724 usec
  - 0 missing
```
Examining these additional tests show that despite there being less cores, the second implementation is still significantly faster thanks to the optimization and correctness of assiging mutexes to each entry, thus allowing parallelism for dramatic performance improvement. 

## Cleaning up
```shell
make clean
```