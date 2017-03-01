# TL;DR: The Lock Taxonomy

CS 343 - Concurrent and Parallel Programming

02-28-2017

Elvin Yung

This is roughly a summary of sections 6.1 to 6.3 inclusive.

# A note on terminology
* A **task** is a generic term for a thread/actor/process/etc.
  * The **acquiring task** or **acquirer** is the task that wants to access the resource.
  * The **releasing task** or **releaser** is the task that has just finished accessed the resource.
* An **event** is the thing that the task is waiting for. Usually this is the lock being unlocked.
* **blocking** is being used interchangeably with **waiting**. Both means that the task doesn't get rescheduled until it gets unblocked by an event.

# Broadly:
* There are two types of locks:
  * **spinning** - busy waiting
  * **blocking** - cooperative scheduling

* Locks are used for two purposes:
  * **synchronization** - notify tasks waiting for an event
  * **mutual exclusion** or **mutex** - protect a resource from being accessed by conflicting tasks

# Spinning locks
* The acquirer is responsible for checking whether the event has occurred, which it does constantly by busy-waiting
* Usually a low-level primitive

## Non-yielding
* Pure busy-wait, waste CPU cycles
* `uSpinLock` in uC++

## Yielding
* Give up time slice so that another task can be scheduled
* `uLock` in uC++

# Blocking locks
* The acquirer checks for the event once, and then goes to sleep.
* The releaser is responsible for waking up and notifying the acquirer.
* Usually stores some sort of queue of tasks waiting to be notified.

## Mutex lock
* As per name, used purely for mutual exclusion
* **single acquisition** and **multiple acquisition**/**owner** variants, different on whether the task that already holds the lock can still acquire it
  * If multiple acquisition, implementations differ on whether only 1 release is needed, or as many as acquires
* uC++ has `uOwnerLock`, i.e. multiple acquisition variant only

## Synchronization lock
* aka sync lock, condition variable, condition lock, monitor
* Is considered the "weakest" lock because there is only one operation: wait.
* The sync lock only stores a single piece of state: the list of waiting tasks. However, mutating this list is a critical section that needs to be mutex'd.
* As per name, used purely for synchronization
* `uLock` in uC++

## Barrier
* Like a sync lock, except associated with a number `n`
* When the `n`th task waits on the barrier, it wakes up everyone
* Unlike sync lock, barrier only stores the number of waiting tasks
* Only used for synchronization
* `uBarrier` in uC++

## Semaphore
* Basically, associated with some number `n`
* Two operations:
  * `P` -- to decrease, equivalent to lock acquire
  * `V` -- to increase, equivalent to lock release
* When task tries to `P` and n = 0, task blocks
* When a task `V`, it wakes up a blocking task
* `uSemaphore` in uC++

* Two types: **binary** (`n` starts as 1) and **counting** (arbitrary `n`)
* The difference between a binary semaphore and a mutex lock is that a semaphore can be initialized to 0.
