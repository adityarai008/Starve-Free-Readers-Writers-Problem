# Readers-Writers Problem with  Semaphores

* Writers Starve - Writers starve when readers are reading.
* Readers Starve - Readers starve when writers are writing. 
* No starve - No process starve.



# Writer Starve

We use 
<ul>
  <li>Semaphore <b>mutex_read_write</b> initialized to 1 for mutual exclusion of reading or writing processes.</li>
  <li>Semaphore <b>mutex</b> initialized to 1 for mutual exclusion between reading processes. It is used for a short duration when a reading process begins and ends, while updating read_count, <b>not</b> during the reading process itself.</li>
  <li>Integer <b>read_count</b> initialized to 0 to keep track of number reading processes.</li>
 </ul>

Structure of Writer Process
```
do {
  wait(mutex_read_write); 
               ...     
              //Writing
               ... 
  signal(mutex_read_write); 
} while (true);

```

Structure of Reader Process
```
do {
  wait(mutex);
  if (++read_count == 1) 
    wait(mutex_read_write); 
  signal(mutex); 
  ...
  /* reading is performed */ 
  ... 
  wait(mutex);
  if (--read_count == 0) 
    signal(mutex_read_write); 
  signal(mutex); 
} while (true);
```

Reader Process : 
* First lock mutex to update number of reading processes(read_count). If there's any reader process then mutex_read_write is locked. This puts writers on <b>wait</b>. On updating number of reading processes mutex is released.
*  Reading is performed.
*  Lock mutex to update read_count. If no reading process left release mutex_read_write else dont. If mutex_read_write is not released due to continuous request from reader process(i.e. read_count is not 0) then it may lead **Writers Starvation** . On updating read_count release mutex.

Writer Process:
* Lock mutex_read_write.
* Do the writing.
* Release mutex_read_write.
# Reader Starve
Also called as **Second Readers-Writers problem**. It requires no writer, once added to the queue, shall be kept waiting longer than absolutely necessary. This is also called **writers-preference**.
This results in readers waiting and ultimately may lead to readers starving.

We use
* Semaphore **mutex_read_write** initialized to 1 for mutual exclusion of reading or writing processes.
* Semaphore **mutex_read** initialized to 1 for mutual exclusion between reading processes. It is used for a short duration when a reading process begins and ends,
 while updating read_count, not during the reading process itself.
* Semaphore **wmutex** initialized to 1 for mutual exclusion between writing processes. It is used for a short duration when a writing process begins and ends,
 while updating write_count, not during the writing process itself.
* Semaphore **readTry** initialized to 1 used by writing processes to lock out reading processes.
* Integer **read_count** initialized to 0 to count number of reading proceses.
* Integer **write_count** initialized to 0 to count number of writing processes.

Structure of Writer Process
```
do{
  wait(wmutex);
  if(++write_count == 1){
    wait(readTry);
  }
  signal(wmutex);
  
  wait(mutex_read_write);
  ...
  /* writing is done*/
  ...
  signal(mutex_read_write);
  
  wait(wmutex);
  if(--write_count == 0){
    signal(readTry);
  }
  signal(wmutex);
}while(true);
```
Structure of Reader Process
```
do{
  wait(readTry);
  wait(mutex_read);
  if (++read_count == 1) 
    wait(mutex_read_write); 
  signal(mutex_read); 
  signal(readTry);
  
  ...
  /*Reading is done*/
  ...
  
  wait(mutex_read);
  if (--read_count == 0) 
    signal(mutex_read_write); 
  signal(mutex_read);
}while(true);
```
Reader Process:
* Check with **readTry** if reading is allowed. If there is any writer process readTry is already locked and read process is halted here only. If not released this may lead to **Reader Starvation**.
* Reader process proceed by locking readTry if not already occupied by a writer process. Lock **mutex_read** to update read_count. If it is first reader then lock **mutex_read_write** so that writer process can not access the resource now. Subsequently mutex_read and readTry are released.
* Reading is done.
* Lock **mutex_read**. Update read_count. If no reading process left then release mutex_read_write. Release mutex_read at the end.

Writer Process:
* Lock **wmutex** to update write_count value. If there is any writing process then **readTry** is locked preventing reader processes to read. Then wmutex is released.
* Lock **rwmutex** to prevent any other writer process to write while one writer is writing. 
* Do the writing process.
* Release **rwmutex**.
* Lock **wmutex** to update write_count. If no writer process is left then release **readTry** to allow reader process to read. Release **wmutex** at the end.

# No starve
It requires that no thread shall be allowed to starve; that is, 
the operation of obtaining a lock on the shared data will always terminate in a bounded amount of time.
Here I present two solutions for the problem. One is the classical solution and the other is faster solution. The faster solution is 
fast as in it requires less number of semaphore lockings.

We use
* Semaphore **in** initialized to 1 for a process entering.
* Semaphore **out** initialized to 1 for a proccess exiting.
* Semaphore **mutex_read_write** initialized to 0 for exclusion of writing and reading processes.
* Integer **in_count** initialized to 0 number of entered reading processes.
* Integer **out_count** initialized to 0 number of exited reading processes.
* Boolean **wait** initialized to false to check if a writer is waiting.

Structure of Writer process
```
do{
  wait(in);
  wait(out);
  if(in_count==out_count){
    signal(out);
  }
  else{
    wait=true;
    signal(out);
    wait(mutex_read_write);
    wait=false;
  }
  ...
  /* Writing is done*/
  ...
  signal(in);
}while(true);
```
Structure of Reader proces
```
do{
  wait(in);
  in_count++;
  signal(in);
  ...
  /*Reading is done*/
  ...
  wait(out);
  out_count++;
  if(wait && in_count==out_count){
    signal(mutex_read_write);
  }
  signal(out);
}while(true);
```

**Writer process** : Lock **in** to prevent any new reader from entering. If number of entered reading process equal to exited reading process then just continue with writing process. Else mark **wait** equal to true to let readers know that writer processes are waiting. Lock **mutex_read_write** when readers allow it to prevent other readers and writers from accessing the resource. When mutex_read_write is acquired wait is marked false and writing is done. Release **in** in the end.<br>
**Reader process** : Lock **in** to update in_count. If writer has not locked it already then it is now occupied by the reader. On updating release **in** for other processes. Reading is done. Lock **out** to update out_count. If some writer is waiting and all entered readers have exited then **mutex_read_write** is released to allow writers access to the resource.
