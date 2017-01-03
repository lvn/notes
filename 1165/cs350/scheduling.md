# Scheduling

CS 350 - Operating Systems
Spring 2016
Elvin Yung

**Previous:** [Virtual Memory](vm.md)
**Next:** [I/O](io.md)

## Motivation
* A **scheduling policy** determines how to assigning work (called *tasks* or *jobs*) to workers.
* In an OS, the jobs are the threads/processes (basically same thing in OS161), and the workers are the processors.

* For now, the main metric that we care about is the average **turnaround time**: the time from the job's arrival to the job's completion.
* It would also be great if the scheduling was *fair*, i.e. that every job gets a proportionate amount of resources, although we don't talk about that here.

## Assumptions
A bunch of mostly-unreasonable assumptions that we'll make, and then remove one by one:
* 1) Each job takes the same amount of time to run.
* 2) Every job arrives at the same time.
* 3) When a job runs, it runs to completion.
* 4) Jobs only use the CPU, and perform no I/O
* 5) We know the workload of each job.

## First In, First Out (FIFO)
* (Assume that jobs don't all arrive at the *exact* same time, i.e. that assumption 2 doesn't strictly hold.)
* The most naive way to do it: just keep taking the first incomplete job and running it.

### Example
* Suppose we have 3 jobs, A, B, and C, arriving in that order, and they each take 1 unit of time.
* Then the turnaround time for A is 1, for B it's 2, and for C it's 3.
* The average turnaround time is 2.

### Relax Assumption 1
* Now relax assumption 1, i.e. now assume that jobs can have different workloads.
* Suppose we have 3 jobs, A, B, and C, arriving in that order. A takes 10 units of time, and B and C each take 1 unit of time.
* Then the turnaround time for A is 10, for B it's 11, and for C it's 12.
* Average turnaround time = 11
* That's terrible!

* Obvious flaw of FIFO: if a long job gets run, many jobs can pile up behind it (**convoy effect**).

## Shortest Job First (SJF)
* Also pretty simple: just keep picking the job with the smallest workload.
* Works great *as long as assumption 2 holds*.

* With our previous example, one correct order is this: B, C, A
* Then the turnaround time for B is 1, for C it's 2, and for A it's 12.
* Average turnaround time = 5
* Huge improvement.

### Relax Assumption 2
* Now suppose jobs can arrive at different times.
* In particular, suppose that job A arrives at t = 0, and then jobs B and C arrive at t = 1.
* Then at the beginning the scheduler would run A since that's the only (and thus shortest) job, and then B and C would still be forced to wait behind A.
* In other words, the convoy effect still occurs.

## Shortest Time-to-Completion First (STCF)
* Now we relax assumption 3.
* We introduce **preemption**: the scheduler can choose to interrupt a job while it's running, run another job, and go back to it later.
* STCF works like this: when a new job is enqueued, if the new job has less time left than the current job, it preempts the current job and runs the new job.

* With our previous example, when job B arrives at t = 1, job A still has 9 time units left, whereas job A only 1 left, so A gets preempted and B gets run.
* Then at t = 2, job B is done, but job C only has 1 time unit left while B still has 9, so C gets run.

* The turnaround times are as follows:
  * A: 12 - 0 = 12
  * B: 2 - 1 = 1
  * C: 3 - 1 = 2
  * Average turnaround time: 5

### Relax Assumption 4
* STCF is great, but from the 1960s onward computers started being interactive.
* Interactivity means that jobs might spend time waiting for I/O (e.g. writing to disk, waiting for stdin, etc.)
* This means that we care about the **response time**: the time between when the job first arrives, and when it gets run for the first time.
* For our previous example, A and B each have response times of 0, but C has a response time of 1.

## Round Robin (RR)
* So it's clear that preempting only when a new job arrives doesn't always work.
* We already know about timer interrupts - why not use those?
* In particular, we will run jobs a **slice** (or **quantum**) at a time, i.e. every time we run a job, we only run it until the next timer interrupt.
* RR works like this:
  * Run a slice of the first job on the queue.
  * If the job isn't complete at the end of the quantum, move it to the back.
  * Repeat.

## Multi-Level Feedback Queue (MLFQ)
* MLFQ is the most common scheduling algorithm in modern OSes. It's used in OS X, Linux (since 2.4), and Windows (since NT).
* It essentially gives preference to short I/O bound jobs.

* The basic setup is that there are multiple queues with different priority levels (i.e. the *multi-level* part).
* Then, the scheduler runs jobs based on these rules:
  * Each job starts off at the end of the highest priority queue.
  * The scheduler processes the jobs in the highest non-empty priority queue, in a round-robin fashion.
  * When a job is run, it's allocated a quantum of time.
    * If it finishes before the end of the quantum, it leaves the system.
    * If it uses up the entire quantum but doesn't finish, it's preempted, and put at the end of the queue below (obviously, unless it's already at the bottom).
    * If it gives up CPU control before the end of the quantum without finishing, it stays at the same level.
  * In other words, the scheduler uses *feedback* to determine which priority a job should be at.

* You'll notice that it's possible that jobs in lower level queues might take a long time to complete if a lot of jobs enter (**starvation**).
* Some MLFQ implementations work around this by periodically moving all jobs back to the top priority queue.

## Interesting Extra Stuff
* [What really happened on Mars?](http://research.microsoft.com/en-us/um/people/mbj/Mars_Pathfinder/Mars_Pathfinder.html) - tl;dr how the *Pathfinder* Mars probe got brought down by a crappy scheduling policy.
* [A Quantitative Measure of Fairness and Discrimination for Resource Allocation in Shared Computer Systems](http://www.cs.wustl.edu/~jain/papers/ftp/fairness.pdf) by R. Jain et al. - A common metric for scheduling fairness.
* [MIT OCW Queueing Theory course](http://ocw.mit.edu/courses/sloan-school-of-management/15-072j-queues-theory-and-applications-spring-2006/)

**Next:** [I/O](io.md)
