# Readers-Writers Problem with  Semaphores

* Writers Starve - Writers starve when readers are reading.
* Readers Starve - Readers starve when writers are writing. 
* No starve - No process starve.


Writer Process:
* Lock **wmutex** to update write_count value. If there is any writing process then **Try_to_read** is locked preventing reader processes to read. Then wmutex is released.
* Lock **rwmutex** to prevent any other writer process to write while one writer is writing. 
* Do the writing process.
* Release **rwmutex**.
* Lock **wmutex** to update write_count. If no writer process is left then release **Try_to_read** to allow reader process to read. Release **wmutex** at the end.


Reader Process:
* Check **Try_to_read** if reading is allowed.**Try_to_read** is already locked and read process is halted here only if there is any writer's process. If not released this may lead to **Reader Starvation**.
* Reader process proceed by locking Try_to_read if not already occupied by a writer process. Lock **rmutex** to update read_count. If it is first reader then lock **mutex_read_write** so that writer process can not access the resource now. Subsequently rmutex and Try_to_read are released.
* Reading is done.
* Lock **rmutex**. Update read_count. If no reading process left then release mutex_read_write. Release rmutex at the end.


## Classical Solution

We use
* Semaphore **in** initialized to 1 to check which process is going to enter critical section.
* Semaphore **mutex** initialized to 1 for mutual exculsion in reading processes while updating read_count.
* Semaphore **mutex_read_write** initialized to 1 for mutual exclusion between writing processes.
* Integer **read_count** initialized to 0 to count number of reading processes.

Structure of writer process
```
do{
  wait(in);
  wait(mutex_read_write);
  ...
//writing
  ...
  signl(mutex_read_write);
  signl(in);
}while(true);
```
Structure of reader process
```
do{
  wait(in);
  wait(mutex);
  if(++read_count==1){
    wait(mutex_read_write);
  }
  signl(mutex);
  signl(in);
  ...
//reading
  ...
  wa(--read_count==0){
    signl(mutex_read_write);
  }
  signl(mutex);
}while(true);
```
it(mutex);
  if
**Writer Process** : Lock **in** semaphore to prevent reader process from reading.
Lock **mutex_read_write** to prevent other writing processes from writing. Do the writing then release the semaphores.
Readers can block writers by locking mutex_read_write if readers obtained permission to read first.<br/>
**Reader Process** : Wait if there's any writer using **in**. Lock **mutex** to update read_count and if its the first reader lock **mutex_read_write** to 
prevent writers from writing. Release **in** and **mutex**. Do the reading.<br/>
Again lock **mutex** to update read_count and if its last reading process release **mutex_read_write** to allow writers to write.

## Faster Solution


We use
* Semaphore **in** initialized to 1 for a process entering.
* Semaphore **out** initialized to 1 for a proccess exiting.
* Semaphore **mutex_read_write** initialized to 0 for exclusion of writing and reading processes.
* Integer **in_count** initialized to 0 number of entered reading processes.

Structure of Writer process
```
do{
  wait(in);
  wait(out);
  if(in_count==out_count){
    signl(out);
  }
  else{
    wait=true;
    signl(out);
    wait(mutex_read_write);
    wait=false;
  }
  ...
  /* Writing is done*/
  ...
  signl(in);
}while(true);
```
Structure of Reader proces
```
do{
  wait(in);
  in_count++;
  signl(in);
  ...
  /*Reading is done*/
  ...
  wait(out);
  out_count++;
  if(wait && in_count==out_count){
    signl(mutex_read_write);
  }
  signl(out);
}while(true);
```

**Writer process** : Lock **in** to prevent any new reader from entering. If number of entered reading process equal to exited reading process then just continue with writing process. Else mark **wait** equal to true to let readers know that writer processes are waiting. Lock **mutex_read_write** when readers allow it to prevent other readers and writers from accessing the resource. When mutex_read_write is acquired wait is marked false and writing is done. Release **in** in the end.<br>
**Reader process** : Lock **in** to update in_count. If writer has not locked it already then it is now occupied by the reader. On updating release **in** for other processes. Reading is done. Lock **out** to update out_count. If some writer is waiting and all entered readers have exited then **mutex_read_write** is released to allow writers access to the resource.
Also called as **Second Readers-Writers problem**. It requires no writer, once added to the queue, shall be kept waiting longer than absolutely necessary. This is also called **writers-preference**.
This results in readers waiting and ultimately may lead to readers starving.

We use
* Semaphore **mutex_read_write** initialized to 1 for mutual exclusion of reading or writing processes.
* Semaphore **rmutex** initialized to 1 for mutual exclusion between reading processes. It is used for a short duration when a reading process begins and ends,
 while updating read_count, not during the reading process itself.
* Semaphore **wmutex** initialized to 1 for mutual exclusion between writing processes. It is used for a short duration when a writing process begins and ends,
 while updating write_count, not during the writing process itself.
* Semaphore **Try_to_read** initialized to 1 used by writing processes to lock out reading processes.
* Integer **read_count** initialized to 0 to count number of reading proceses.
* Integer **write_count** initialized to 0 to count number of writing processes.

Structure of Writer Process
```
do{
  wait(wmutex);
  if(++write_count == 1){
    wait(Try_to_read);
  }
  signl(wmutex);
  
  wait(mutex_read_write);
  ...
  /* writing is done*/
  ...
  signl(mutex_read_write);
  
  wait(wmutex);
  if(--write_count == 0){
    signl(Try_to_read);
  }
  signl(wmutex);
}while(true);
```
Structure of Reader Process
```
do{
  wait(Try_to_read);
  wait(rmutex);
  if (++read_count == 1) 
    wait(mutex_read_write); 
  signl(rmutex); 
  signl(Try_to_read);
  
  ...
  /*Reading is done*/
  ...
  
  wait(rmutex);
  if (--read_count == 0) 
    signl(mutex_read_write); 
  signl(rmutex);
}while(true);
```
