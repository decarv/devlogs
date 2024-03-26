# GSoC Proposal - PostgreSQL: pgagroal

## 0. Contributor Information

**Name:** Henrique A. de Carvalho
**Email:** [decarv.henrique@gmail.com](mailto:decarv.henrique@gmail.com)
**GitHub:** [decarv](https://github.com/decarv)
**Languages:** Portuguese, English
**Country:** Brazil (GMT -03:00)
**Who Am I:** I earned a Bachelor's degree in Computer Science from the University of São Paulo (2020 - 2023) and currently work as a Backend Software Engineer at a medium-scale Investment Fund. Throughout my bachelor's program, I did bunch of different stuff, from algorithms and data structures, to systems engineering, to machine learning theory and applications, computer vision, computer graphics and application development and benchmarking. Nevertheless, the things that live in my heart are systems engineering, performance-oriented development and computer graphics, which are things I rarely have the opportunity to do. Therefore, I see contributing to open-source as a great way to stay close to what makes me happy in software engineering and, of course, to be able to provide good software for free to other humans. Specifically, my interest in pgagroal stems from its system program nature and its promise of high-performance — subjects I'm passionate about, especially when it involves optimizing for speed (not that I know that much, it's just that I want to learn about it that much). My main objective with this project is to learn from the community and assist pgagroal in achieving unparalleled performance among connection pools.
**Contributions:** [#408](https://github.com/agroal/pgagroal/pull/408), [#411](https://github.com/agroal/pgagroal/pull/411), [#427](https://github.com/agroal/pgagroal/pull/427)

## 1. Synopsis

This project consists of replacing the I/O Layer of pgagroal, today highly dependant on [libev library](TODO), for a pgagroal's own implementation of this I/O Layer. The so called I/O Layer consists of an event loop (ev) abstracted by libev. 

The motivation behind this project is because libev is not being maintained any longer. Therefore we need an efficient (i.e. maintainable, reliable, fast, lightweight, secure and scalable) implementation of an ev that can be maintained by the pgagroal community.

Currently, pgagroal depends on libev to (a) watch for incomming read/write requests from its connections in a non-blocking fashion (I/O multiplexing); and (b) to watch signals.

I/O multiplexing in Linux is done with select, poll and, most recently (kernel version 2.6) with epoll, which is used by libev on the background. epoll works by decoupling the monitor registration from the actual monitoring, and does this with an intuitive API, using three simple system calls, one to create an epoll instance, one to add or remove file descriptors to monitor (along with the events to monitor in each file descriptor), and one to actually monitor the file descriptors.

A successful implementation of an efficient ev for pgagroal necessarilly comprises the utilization of epoll as well as other linux I/O features for efficient I/O.

The objective of this proposal is to provide a plan to achieve such implementation, which shall be, at the end of this program, at least as efficient as libev, but fully maintained and controlled by the pgagroal community.

## 2. Proposal

As mentioned before, my objectives with this proposal is to implement an efficient ev for pgagroal. 

In order to accomplish this, I could benefit of dividing the implementation into two phases: (a) Experimentation (Phase 1); (b) Continuous Implementation and Profiling Loop (Phase 2).

For the **Phase 1**, I propose the implementation of ev.h and ev.c containing a simple abstraction for an ev (with `epoll`) that closely follows (in a first moment) the interface used by pgagroal with libev. 

This would allow the main code to be changed only slightly, where it makes sense, avoiding unnecessaryly redesigning the main code and understanding where I would have to modify it in order to connect it to an ev interface. 

The result of this first phase would be a small footprint ev that suffices for pgagroal specific uses, resulting in minimal changes in function signatures and behavior of the main code. 

For the **Phase 2**, I first propose the definition of tests and profiling (for speed and memory) for pgagroal's new ev, done in different settings, enabling comparison between previous and future versions. 

The idea here is to acurately measure resource utilisation for pgagroal in areas we believe are important for a connection pool, so that we can make sure that pgagroal is going on the direction we believe is correct. 

This will enable the identification of bottlenecks and places where pgagroal can benefit from optimizations while being able to measure potential optimizations implementations. 

Second, I propose diving deeper into improvements that could be made to the simple ev implementation of the previous phase, here I intend to investigate the potential necessary changes of structure of the main code, considering (a) the usage of io\_uring, (b) the different configurations for epoll (e.g. edge vs. level trigger), (c) cache performance, (d) optimizations with memory layout, (e) reducing system calls, (f) vectorization of read and writes.

The tests and the profiling should lead the development after each commit.


## . EV Implementation Details

pgagroal uses a default main loop configuration for libev, which uses epoll and io\_uring where needed. 

This main loop is fed by pgagroal main code with main file descriptors, file descriptor for management and file descriptor for postgres server. 


The implementation depends on me knowing where I need to modify the project in order to completely replace libev.
Here follows a through description of where in the project libev is used and how I intend to replace its usage.

```c 
./main.c:#include <ev.h>
./main.c:   /* libev */
./main.c:   main_loop = ev_default_loop(pgagroal_libev(config->libev));
./main.c:                         pgagroal_libev(config->libev), ev_supported_backends());
./main.c:      sd_notifyf(0, "STATUS=No loop implementation (%x) (%x)", pgagroal_libev(config->libev), ev_supported_backends());
./main.c:   pgagroal_libev_engines();
./main.c:   pgagroal_log_debug("libev engine: %s", pgagroal_libev_engine(ev_backend(main_loop)));
```

Here libev is used for [TODO].

```c 
./libpgagroal/pipeline_perf.c:#include <ev.h>
```

Here libev is used for [TODO].

```c
./libpgagroal/worker.c:#include <ev.h>
./libpgagroal/worker.c:      loop = ev_loop_new(pgagroal_libev(config->libev));
```

Here libev is used for [TODO].

```c
./libpgagroal/remote.c:#include <ev.h>
```

Here libev is used for [TODO].

```c
./libpgagroal/pipeline_session.c:#include <ev.h>
```

Here libev is used for [TODO].

```c
   /* libev */
   restart_string("libev", config->libev, reload->libev, true);
   config->buffer_size = reload->buffer_size;
   config->keep_alive = reload->keep_alive;
   config->nodelay = reload->nodelay;
   config->non_blocking = reload->non_blocking;
   config->backlog = reload->backlog;

./libpgagroal/configuration.c:   /* libev */
./libpgagroal/configuration.c:   restart_string("libev", config->libev, reload->libev, true);
./libpgagroal/configuration.c:   else if (key_in_section("libev", section, key, true, &unknown))
./libpgagroal/configuration.c:      memcpy(config->libev, value, max);
```

Here libev is used for [TODO].

```c
./libpgagroal/prometheus.c:#include <ev.h>
```

Here libev is used for [TODO].

```c
./libpgagroal/pipeline_transaction.c:#include <ev.h>
```

Here libev is used for [TODO].

```c
./libpgagroal/utils.c:#include <ev.h>
./libpgagroal/utils.c:pgagroal_libev_engines(void)
./libpgagroal/utils.c:      pgagroal_log_debug("libev available: select");
./libpgagroal/utils.c:      pgagroal_log_debug("libev available: poll");
./libpgagroal/utils.c:      pgagroal_log_debug("libev available: epoll");
./libpgagroal/utils.c:      pgagroal_log_debug("libev available: linuxaio");
./libpgagroal/utils.c:      pgagroal_log_debug("libev available: iouring");
./libpgagroal/utils.c:      pgagroal_log_debug("libev available: kqueue");
./libpgagroal/utils.c:      pgagroal_log_debug("libev available: devpoll");
./libpgagroal/utils.c:      pgagroal_log_debug("libev available: port");
./libpgagroal/utils.c:pgagroal_libev(char* engine)
./libpgagroal/utils.c:            pgagroal_log_warn("libev not available: select");
./libpgagroal/utils.c:            pgagroal_log_warn("libev not available: poll");
./libpgagroal/utils.c:            pgagroal_log_warn("libev not available: epoll");
./libpgagroal/utils.c:            pgagroal_log_warn("libev not available: iouring");
./libpgagroal/utils.c:            pgagroal_log_warn("libev not available: devpoll");
./libpgagroal/utils.c:            pgagroal_log_warn("libev not available: port");
./libpgagroal/utils.c:         pgagroal_log_warn("libev unknown option: %s", engine);
./libpgagroal/utils.c:pgagroal_libev_engine(unsigned int val)
```

Here libev is used for [TODO].

```c
./CMakeLists.txt:    ${LIBEV_INCLUDE_DIRS}
./CMakeLists.txt:    ${LIBEV_LIBRARIES}
./CMakeLists.txt:    ${LIBEV_INCLUDE_DIRS}
./CMakeLists.txt:    ${LIBEV_LIBRARIES}
./CMakeLists.txt:    ${LIBEV_INCLUDE_DIRS}
./CMakeLists.txt:    ${LIBEV_LIBRARIES}
```

Here libev is used for [TODO].


```c
./include/prometheus.h:#include <ev.h>
./include/remote.h:#include <ev.h>
./include/pgagroal.h:#include <ev.h>
./include/pgagroal.h:   char libev[MISC_LENGTH]; /**< Name of libev mode */
./include/utils.h:   struct ev_signal signal; /**< The libev base type */
./include/utils.h: * Print the available libev engines
./include/utils.h:pgagroal_libev_engines(void);
./include/utils.h: * Get the constant for a libev engine
./include/utils.h:pgagroal_libev(char* engine);
./include/utils.h: * Get the name for a libev engine
./include/utils.h:pgagroal_libev_engine(unsigned int val);
./include/worker.h:#include <ev.h>
./include/worker.h:   struct ev_io io;      /**< The libev base type */
./include/pipeline.h:#include <ev.h>
```

Here libev is used for [TODO].


```c prometheus.h

uses ev.h

```


```c remote.h

uses ev.h

```



```c remote.h

uses ev.h

```

./include/pgagroal.h:#include <ev.h>
./include/pgagroal.h:   char libev[MISC_LENGTH]; /**< Name of libev mode */
./include/utils.h:   struct ev_signal signal; /**< The libev base type */
./include/utils.h: * Print the available libev engines
./include/utils.h:pgagroal_libev_engines(void);
./include/utils.h: * Get the constant for a libev engine
./include/utils.h:pgagroal_libev(char* engine);
./include/utils.h: * Get the name for a libev engine
./include/utils.h:pgagroal_libev_engine(unsigned int val);
./include/worker.h:#include <ev.h>
./include/worker.h:   struct ev_io io;      /**< The libev base type */
./include/pipeline.h:#include <ev.h>




## . Testing and Profiling Implementation Details

With testing and profiling I intend to achieve a way to measure how the implementation of the ev is evolving in comparison to previous pgagroal versions and to previous versions.

Tests could be achieved through testing frameworks in C or just by testing the behaviour with simulated postgres client connections as shell scripts.

Profiling could be achieved through the usage of linux tools such as gprof and perf. These tools could be wrapped around Python scripts to enable easy data parsing, storage and analysis.

## . Deliverables

**Phase 1**
- Fully functional event loop 

**Phase 2**
- Testing and Profiling Strategy
- Tests setup
- Profiling tools setup
- Investigation of optimisations


## . Timeline

Below I set a timeline for 22 weeks. I try to keep two weeks as a division for each row.

Week, Date, Description
1 & 2, May    01 - May       12, This is a community bonding period, we could set up a call to know each other, to talk about pgagroal (the history behind it and the future of the project) and to talk about this project in specific. This is likely the start of Phase 1.
3 & 4, May    13 - May       27, Work on the first version of the ev.
4 & 5, May    27 - June      09, Deliver the first version of an ev, fully working.
6 & 7, June   10 - June      30, Work on testing and profiling strategy and setup.
8, June   24 - June 30, Finish testing and profiling setup
9 & 10, July   01 - July      14, The midterm evaluations, from 8 to 12. I intend to visit my family during the mid-year holidays. I will be reachable and responding to emails, but I intend to mostly rest around this time.
11 & 12, July   15 - July 28, Investigation of improvements to the ev.
13 & 14, July 29 - August 12, Second Evaluation.
September 3 - November 4, Continue investigation of improvements to the ev and write reports.
November 11 - 18:00 UTC
Final date for mentors to submit evaluations for GSoC contributor projects with extended deadlines

