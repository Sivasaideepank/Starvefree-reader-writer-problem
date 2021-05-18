# Reader-writer-problem
A Reader-Writer problem is a situation where multiple processes try to access some data area simultaneously. The data area that is being accessed is called "Critical Section". If a process tries to read data from critical section that process is called as reader and if a process tries to modify data it is called as writer.

If two processes try to modify same data the data may be corrupted. To avoid this we have to synchronize the processes that try to access critical section. In the synchronization multiple readers can access the data without single writer or only single writer can access the data.


# Background
For this problem there are three types of solution

## 1. First-Readers
In this solution basically the readers were given priority, i.e. first readers were allowed to access the critical section. If there were no readers then writers will be given access to modify the data in critical section in mutual exclusion. In this solution we may get a situation that the writers may starve due to continuous incoming readers.

## Implementation :

Variables used for implementation are 
Mutex = Semaphore(1); // to ensure that no other reader will execute entry section while a present reader is in it i.e to ensure mutual exclusion .
Int Readerscount = 0; // number of reader processes in critical section initialized to 0;
Resource = Semaphore(1); // used by both readers and writers to lock resource.

### Pseudo Code :
### Reader :
Do{
wait(mutex);
Readerscount = Readerscount+1;
if(Readerscount==1)  wait(resource);  //if it is first reader it reserves resource for other readers by locking resource from writers
signal(mutex);

Critical section


wait(mutex);
Readerscount = Readerscount-1;
if(readerscount==0) signal(resource);// If it is last reader unlock resource and make it available for writers.
signal(mutex);
}while(true)

### Writer : 
Do{
wait(resource);   //request for resource and if granted lock the resource as no writer or reader can enter if writer is in CS

Critical section


signal(resource);  //release the shared resource
}while(true);



## Explanation : 
+ For a reader before it enters critical section it should go through entry section. To avoid race conditions every reader which enters the entry section will lock the entry section for themselves until they are done with it, with the help of semaphore mutex.
+ Locking is done with wait(mutex) and unlocking with signal(mutex).
+ The same is valid for the exit section.
+ Once the first reader is in entry section it locks the resource preventing other writers from accessing it.
+ Other readers can use the resource but the last reader must unlock the resource making it available for writers.
+ For a writer it waits for unlocking of the resource and once all the readers are done it then locks the resource and enters the critical section avoiding readers from accessing it.
+ In this solution a stream of readers can stop all the writers and starve them because after the first reader locks the resource, no writer can use it, before lock gets released and it will only be released by the last reader. 
Hence this solution is not fair.

## 2.Second Readers-Writers Problem :


This solution is quite opposite to previous one. In this solution writers were given priority to access the critical section. If there were no writers then readers were given access to read the data in critical section. Similar to previous one we will get a situation where readers may starve because of priority given to readers.

As per the statement writer is given higher priority compared to reader.


## Implementation :
Variables used for implementation are 
Int Readerscount = 0;  // number of reader processes in critical section initialized to 0;
Int Writerscount = 0; //number of writer processes in critical section initialized to 0;
Readmutex = Semaphore(1); // to ensure that no other reader will execute readers entry section while a present reader is in it i.e to ensure mutual exclusion
Writemutex = Semaphore(1); // to ensure that no other writer will execute writers entry section while a present writer is in it i.e to ensure mutual exclusion
Resource = Semaphore(1);
R = Semaphore(1); // to lock readers for reserving space for other writers 
## PseudoCode :
### Reader :
Do {
wait(mutex);
wait(readmutex); 
Readerscount = Readerscount + 1;
if(Readerscount == 1) wait(Resource); // if it is first reader lock the resource
signal(readmutex); //release entry section for other readers
signal(mutex); 

Critical section

 
wait(readmutex);
Readerscount = Readerscount -1;
if(Readerscount==0) signal(resource);//if it is last reader release the resource
signal(readmutex);
}while(true);
(only another mutex lock is added )

### Writer:

Do{
wait(writemutex); 
Writerscount = Writerscount+1;
if(Writerscount==1)  wait(mutex);  //if it is first writer then it locks critical section from other readers 
signal(writemutex);
wait(Resource) //lock the shared resource as no writer or reader can enter if writer is in CS

Critical section

signal(resource); //release the shared resource
wait(writemutex);
Writerscount = Writerscount-1;
if(Writerscount==0) signal(mutex);// If it is last writer allow readers to enter critical section by unlocking resource
signal(writemutex);
}while(true)
## Explanation :
+ Here preference is given to writers as readers have to lock and unlock mutex individually while writers don't need to.
+ The first writer will lock the mutex and then all subsequent writers can simply use the resource .The last writer must release the mutex semaphore,opening the gate for readers to try reading. 
+ The reader must wait for last writer to unlock the mutex and resource.
+ If a reader locks the mutex semaphore it indicates to writer that there is a reader in critical section.So the writer will wait for the reader to release the mutex and then the writer will immediately lock it .
+ Though, the writer will not be able to access the resource until the current reader has released the resource, after the reader is finished with the resource in the critical section.
+ A reader will know that there are no writers waiting for resource with mutex semaphore It enters critical section by releasing mutex..
+ As soon as a writer shows up, it will try to set the mutex and hang up there waiting for the current reader to release the mutex. It will then take control over the resource as soon as the current reader is done reading and lock all future readers out. All subsequent readers will hang up at the mutex semaphore waiting for the writers to be finished with the resource and to open the gate by releasing mutex.


# 3. Starve-Free
In this solution we will avoid starvation by giving no priority to readers or writer. Now we are going to look a Starve-Free solution of Readers-Writers problem

The classical solution of the problem results in starvation of either reader or writer. In this solution,I have tried to propose a starve free solution. While proposing the solution only one assumption is made i.e. Semaphore preserves the first in first out(FIFO) order when locking and releasing theprocesses( Semaphore uses a FIFO queue to maintain the list of blocked processes)

### Semaphore
Designing a Semaphore with FIRST-IN-FIRST-OUT(FIFO) queue to maintain the list of blocked processes
```cpp
// The code for a FIFO semaphore.
struct Semaphore{
  int value = 1;
  FIFO_Queue* Q = new FIFO_Queue();
}
    
void Acquire(Semaphore *S,int* process_id){
  S->value--;
  if(S->value < 0){
  S->Q->push(process_id);
  block(); //this function will block the process by using system call and will transfer it to the waiting queue
           //the process will remain in the waiting queue till it is waken up by the wakeup() system calls
           //this is a type of non busy waiting
  }
}
    
void Release(Semaphore *S){
  S->value++;
  if(S->value <= 0){
  int* PID = S->Q->pop();
  wakeup(PID); //this function will wakeup the process with the given pid using system calls
  }
}


//The code for the queue which will allow us to make a FIFO semaphore.
struct FIFO_Queue{
    ProcessBlock* front, rear;
    int* pop(){
        if(front == NULL){
            return -1;            // Error : underflow.
        }
        else{
            int* val = front->value;
            front = front->next;
            if(front == NULL)
            {
                rear = NULL;
            }
            return val;
        }
    }
    void* push(int* val){
        ProcessBlock* blk = new ProcessBlock();
        blk->value = val;
        if(rear == NULL){
            front = rear = n;
            
        }
        else{
            rear->next = blk;
            rear = blk;
        }
    }
    
}

// A Process Block.
struct ProcessBlock{
    ProcessBlock* next;
    int* process_block;
}
```

Let's see a writer process. First the process will check the number of current and waitingreaders. If both, the number of waiting readers and number of current readers are zero or their sum is zero, then  it will read directly. Otherwise, it will wait until a reader process release and allows that writer to enter. Once the writer process is done, if it is the last writer, then it will release and allow all the writers "waiting" readers process to enter. Similarily, the last reader process will release and allow the "waiting" writers processes to enter.

Now let's visualize the problem by imagining a room and two benches outside the room for waiting/queued reader and writer process. Suppose you are a writer, if there is no reader inside and no reader on the bench, you can enter. If there are writers inside, the last writer to leave the room will allow all the readers on the bench to enter. Similar in case of readers inside, then the last reader before leaving will allow all the writers on the bench to enter in the room.

Now the logic behind the algorithm provided for starvation free solution to Readers-Writers problem is given below. Here, we have assumed two scenarios. In first case, initially a Reader process is inside critical section. In second case, initially a Writer process is inside critical section. 

  1. **Let us assume that initially Readers are executing the critical section.**
     + Now, if another Reader arrives, then the algorithm gives it the permission to access the critical section as multiple Readers are allowed inside critical section. 
     + However, if a Writer arrives then, the algorithm will ask him to wait until all currently active Readers (which are present inside critical section) complete their execution.
     + Now, there is Writer that is wating for entering the critical section. Thus, the algorithm will not give permission to anymore Reader (that comes after that Writer) to access the critical section.
     + Once, all the active Readers complete execution, then all waiting Writers (until that time) are given opportunity to access critical section one at a time.
     + Thus, above logic eliminates the possibility of the starvation of any Writer. 
  
  2. **Now, let us assume that initially a Writer is executing the critical section.**
     + Now, if another Writer approaches, then it must wait as algorithm gives permission to only one Writer to access the critical section at a time.
     + In this way, if continuously n Writers request for access to the critical section, then their request will be granted one at a time, but not necessarily in the same order.
     + However, if a Reader arrives then, the algorithm will ask him to wait until all Writers (which are waiting before that Reader for access to the critical section) complete their execution.
     + Now, there is a Reader that is waiting for entering the critical section. Thus, the algorithm will not give permission to anymore Writer (that comes after that Reader) to access the critical section.
     + Once, all the Writers that were waiting before that Reader complete execution, then all waiting Readers (until that time) are given opportunity to access critical section simultaneously.
     + Thus, above logic eliminates the possibility of the starvation of any Reader. 
   
  3. Thus, Readers and Writers are allowed to access to the critical section in alternate turns with the constraint that multiple Readers can access the critical section simultaneously, but only one Writer can access the critical section at a time. In addition, though the above-mentioned algorithm does not rigidly follow First Come First Serve (FCFS) approach, but it is based on that strategy only. 


## Semaphores
Three semaphores are used as explained below:

1. readaccess_semaphore : it is a semaphore that reading processes sleep upon. A Reading process starts reading in the critical section only when read_sem releases, until then it waits. It is initialised to 0.
2. writeaccess_semaphore : it is a semaphore that writing processes sleep upon. A writing process starts reading in the critical section only when write-sem releases, until then it waits. It is initialised to 0.
3. mutex : it a mutex semaphore for controlled access to the following variables --> current_writers, waiting_writers, current_readers, waiting_readers.
It provides mutual exclusion among Readers and Writers. It is initialised to 1.

## Variables Used
1. current_readers : It is the number of readers who are currently in the CRITICAL section and reading the data.
2. waiting_readers : It is the number of readers that are waiting for access to the critical section. It gains access after group of writers that came before are completed.
3. current_writers : It is the number of writers who are currently in the CRITICAL section and writing the data.
4. waiting_writers : It is the count of writers which are waiting for access to the critical section. It gains access only when group of readers that came before it completes their execution.

## Psuedo code
Below is the code for the Reader's writers problem that is starvation free and deadlock-free.
Here is the psuedo code :-
```
// for reading 
if(active + waiting writers are zero)
	release and enter the process
else 
	wait for release
	
//Read DATA

if( this is the last reader)
	release the queued writers
  
// for writing 
if(active and waiting readers are zero)
	release and enter the process
else 
	wait for release
	
//Write DATA

if( this is the last writer)
	release the queued readers
```
## Correctness of Solution
### Mutual Exclusion
The mutex semaphore ensures that only a single writer can access the critical section at any moment of time thus ensuring mutual exclusion between the writers and also when the first reader try to access the critical section it has to acquire the  mutex lock to access the critical section thus ensuring mutual exclusion between the readers and writers.
### Bounded Waiting
Before accessing the critical section any reader or writer have to first acquire the turn semaphore which uses a FIFO queue for the blocked processes. Thus as the queue uses a FIFO policy, every process has to wait for a finite amount of time before it can access the critical section thus meeting the requirement of bounded waiting.
### Progress Requirement
The code is structured so that there are no chances for deadlock and also the readers and writers takes a finite amount of time to pass through the critical section and also at the end of each reader writer code they release the semaphore for other processes to enter into critical section.


