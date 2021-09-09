



### 背景

Between concurrent tasks, writing the same variable requires a synchronization mechanism to ensure its coherence.

 Some parallel programs use the lock or the semaphore mechanisms. But these mechanisms make parallel programs difficult to design, debug, and generally perform poorly. 

We use transaction memory to improve the performance of parallel programs, and ensure the efficiency of programming.

Transactional memory could recover from errors, rollback  , implement synchronization among tasks.

The previous methods write log into memory controller before writing data into cache, the core collects meta-log，get old value，write old value into memory controller, then write new value into the cache. Writing through old value wastes time and spatial locality. 

Our method is generating log before eviction. Update cache immediately when store. Generating log when commiting, we use flush to evict cache line.

Generating log when committing, we use flush to evict cache line. Our method has two advantages, the first is that we cut off the extra operations brought to the store instruction due to the coupling with log generation, and shortens the execution time of the store instruction. The other is that our design naturally supports log coalescing. 

