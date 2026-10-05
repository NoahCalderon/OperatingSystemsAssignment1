CLC - Assignment 1: Producer and Consumer
Course: CST-315
Project date: September 29, 2026

GROUP MEMBERS AND RESPONSIBILITIES
Noah Calderon-Zuniga - Programming
Implemented the C++ producer and consumer threads, circular buffer,
and thread coordination using a mutex and condition variables.

Ethan Calhoun - Documentation
Prepared the implementation overview, reasoning, and explanation of
program behavior and execution results.

PROJECT OVERVIEW
This program demonstrates the producer-consumer problem using POSIX
threads in C++. Three producers and three consumers share a bounded
circular buffer containing five integer slots. Producers generate
uniquely numbered items, and consumers remove items in insertion order.
The program runs continuously until stopped with Ctrl+C.

FILES
producer_consumer.cpp - C++ program source code.
README.txt - Project overview and compile/run instructions.
Assignment 1 CST315 Overview Document (1).docx - Implementation
explanation, design reasoning, and program execution documentation.

REQUIREMENTS
- Ubuntu or another Linux environment with POSIX threads support.
- g++ with C++17 support.
- The producer_consumer.cpp source file in the working directory.

COMPILING AND RUNNING
Open a terminal in the folder containing producer_consumer.cpp.

Compile:
g++ -std=c++17 -Wall -Wextra -pthread producer_consumer.cpp -o producer_consumer

Run:
./producer_consumer

Stop:
Press Ctrl+C in the terminal.

HOW THE PROGRAM WORKS
1. main() initializes the synchronization objects and creates three
   producer threads and three consumer threads with pthread_create().
   Each thread receives its own ID through an element of an ID array.

2. Producers lock the mutex before accessing shared state. If the
   buffer is full, they wait on condProd. Otherwise, they generate an
   item number, insert the item, update the buffer state, and signal
   condCons so a waiting consumer can proceed.

3. Consumers lock the same mutex. If the buffer is empty, they wait
   on condCons. Otherwise, they remove the oldest item, update the
   buffer state, and signal condProd so a waiting producer can proceed.

4. pthread_cond_wait() releases the mutex while waiting and reacquires
   it before returning. Each wait is inside a while loop so the thread
   checks the buffer condition again after waking.

5. Each producer sleeps for one second between attempts, and each
   consumer sleeps for two seconds. Sleeps occur outside the critical
   section. The faster production rate tends to keep items available,
   but it does not guarantee that consumers never wait.

6. main() calls pthread_join() on the threads. Because the worker loops
   run indefinitely, the joins do not return during normal operation.
   Ctrl+C terminates the program; cleanup after the joins is not a
   graceful shutdown path.

SHARED VARIABLES
buffer[N] - Circular integer buffer with N = 5 slots.
in        - Index where the next produced item is inserted.
out       - Index of the next item to be consumed.
count     - Number of items currently buffered, from 0 through 5.
nextItem  - Shared counter used to generate unique item numbers.

The in and out indices wrap using modulo N. The mutex protects the
buffer, indices, count, nextItem, and status output from simultaneous
access. Condition variables let blocked threads sleep instead of
repeatedly checking the buffer and wasting CPU time.

EXPECTED OUTPUT AND VERIFICATION
The terminal displays production, consumption, buffer counts, and
waiting messages. The order of thread IDs can vary between runs
because the operating system schedules the threads.

Check the following during execution:
- Produced item numbers are unique and increase sequentially.
- Consumers remove items in the same order they were inserted.
- No item is consumed more than once.
- The buffer count stays between 0 and 5.
- Producers wait when the buffer is full and resume after space opens.
- Consumers wait when the buffer is empty and resume after an item arrives.

Items still in the buffer when Ctrl+C is pressed may not be consumed.
To exercise the empty-buffer case, temporarily change PRODUCER_DELAY
to 2 and CONSUMER_DELAY to 1, then rebuild and run. Restore the original
values and rebuild after the check. These are verification steps, not
claims that additional tests have been performed.

IMPLEMENTATION SCOPE
This version uses six threads within one process and a five-integer
buffer. It uses a mutex and condition variables, with insertion and
removal handled inside the thread functions rather than separate put()
and get() functions. Consumers wait when no items are available.

The original README identifies differences from the written assignment:
its request for separate processes, a single-word buffer, separate put()
and get() functions, and a restriction on synchronization mechanisms.
The implementation follows the threaded approach described in the
accompanying overview; those differences still require instructor
clarification before claiming full compliance with the written directions.
