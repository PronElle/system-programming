# System-Programming

<p align="center">
<a href="https://imgbb.com/"><img src="https://www.politocomunica.polito.it/var/politocomunica/storage/images/media/images/marchio_e_logotipo_politecnico_di_torino/55127-3-ita-IT/marchio_e_logotipo_politecnico_di_torino_full.png"></a>
</p> 

<p align="center">
 <img alt="C++" src="https://img.shields.io/badge/cmake-v3.0.0-green"/>
 <img alt="developement" src="https://img.shields.io/badge/C++-11 | 14 | 17 | 20-blue.svg?style=flat&logo=c%2B%2B"/> 

</p>

This repository contains the OS API laboratories of the "Programmazione di Sistema" (System Programming) class at PoliTO and personal solutions to the programming exercises from recent exams.

## Index
* Lab1: [MessageStore](Lab1)

* Lab2:
  - [Custom STL Vector](Lab2/ex2)
    - templates
    - raw pointers
    - no custom iterators
  - [Simple File System](Lab2/ex3) 
  
* Lab3
  - [Custom STL Vector](Lab3/ex1)
    - templates
    - raw pointers
    - iterators
  - [STL-like matrix](Lab3/ex2)
    - templates
    - smart pointers
  
* Lab4 : map-reduce pattern

* Lab5
  - [custom lock_guard](Lab5/mylockguard.h)
  - [CountingSemaphore](Lab5/CountingSemaphore.h)
  
* Lab6
  - [custom packaged_task](Lab6/myptask.h)
  - [thread pool](Lab6/threadpool.h)
  - [dynamic thread pool](Lab6/dynamic_pool.h)
  
* Joiner

  - [specifications](#Joiner)
  -  [solution](exams/Joiner.h)

* Exchanger

  - [specifications](#Exchanger)
  - [solution](exams/Exchanger.h)

* RAII guard

  - [specifications](#RAIIGuard)
  - [solution](exams/raii_guard.h)

* SingleThreadExecutor

  - [specifications](#SingleThreadExecutor)
  - [solution](exams/SingleThreadExecutor.h)

* CBuffer 

  - [specifications](#CBuffer)
  - [solution](exams/CBuffer.h)
  
  
  
## Joiner

The software handling a machine tool has N  threads, each of them concurrently reading a value from a sensor. The N readings are gathered in some data structure to be eventually processed by the system. For this purpose, your are required to implement the following thread-safe class

```C++
class Joiner {
public: 
    Joiner(int N);
    std::map <int, double> supply(int key, double value);
}
```

  the blocking method `supply(...)` receives a couple key-value generated by a single thread (key: thread id, value: reading) and blocks the thread (without any CPU consumption) until the other N-1 threads are done providing their reading. 

When `supply(...)` receives the N readings, every thread is woken-up  and the method gets ready for the next N readings and retrieves the map. Make sure not to allow any thread reading again when the Joiner is not ready (a woken-up might be fast enough to provide a new reading before the Joiner cleaned its state for the next N readings).

## Exchanger

The generic class `Exchanger<T>` allows two threads to exchange a value of type T.  It only provides the following method:

```C++ 
T exchange( T t)
```

which blocks the calling thread (without any CPU consumption) until another thread calls the same method on the same instance of the object. When this happend, the method retrieves the object passed as parameter from the other thread.  

## RAII Guard

A company developed a multiplatform software which contains some functions allocating a lot of memory which is freed after their execution. Since the named software is intended to be multithreading, if more threads execute those functions, the required quantity of memory might overcome the limits of the system, resulting in the death of the process. 

Implement a piece of code allowing to specify and limit and maximum number of threads allowed to use those function, blocking (without any CPU consumption) every other thread attempting to call them until the number of executions goes behind threshold (i.e. when one thread is done, another might "enter").  

## SingleThreadExecutor

The class `SingleThreadExecutor` implements the concept of a thread pool based on a single thread. 

When the method `submit(...)` is invoked, a new task is enqueued in the pool and executed when the single thread is ready.

When the method `close()` is invoked, new jobs can't be enqueued anymore and further attempts to call `submit(...)` are gonna raise an exception. 

Calling the method `join()` , all the ramaining tasks are executed and the single thread embedded in the pool terminates.
Implement the following class. Include all the missing parts required for your implementation.

```C++
class SingleThreadExecutor {
public:
    void submit(std::packaged_task<void()> t); 
    void close(); 
    void join(); 
};
```



## Cbuffer 

In a concurrent system two threads run in parallel. The first one produces a sequence of readings from a sensor, while the other one elaborates them. 

Since the two tasks have different timing, they employ the following generic shared structure 

```C++
template <typename T>
class CBuffer {
public:
  void next(T t);
  void fail(std::exception_ptr eptr);
  void terminate();
  std::optional<T> consume();
};
```

  The producer thread enqueues data calling `next(T t)`, while the consumer gets the oldest value (in a FIFO mechanism) calling `consume()` or waits if the buffer is empty. The producer can send a stop signal to declare the enqueuing ended. 
If an exception is raised during the reading, the producer calls `fail(...)`. After a single invocation of  `terminate()` or `fail(...)`, the buffer won't accept any new value and the following invokations of `next(...)`, `fail(...)` or `terminate()` must raise an exception. 

If `terminate()` has been called and the buffer is empty, the method `consume()` must retrieve an empty object. 

If `fail(...)` has been called and the buffer is empty, the method `consume()` rethrows the very same exception using `std::rethrow_exception(...)` .
Implement the following class respect the C++17 standard (add everything needed in the class definition.)


