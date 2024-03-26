## March 2, 2023

*[19:49]* Today I noticed I deleted the `dev.log` that I had so far. Unfortunately I cannot remember exactly what I had,
but it was about the pull request I made.

The problem still exists, I cannot use the pgagroal-cli. It still fails. I tried to use version 1.6.0 to see if this was
the problem, but it did not work either. I am afraid it won't work on ubuntu. I could fix it, but I don't even know if
I want to fix it for Ubuntu.

## March 4, 2023

I have to debug both pgagroal and pgagroal-cli.

### Remote connection vs. Local connection

*[12:00]* The code says that, if the user has specified a configuration path, that will lead to a an attempt of using a
local and remote connections at the same time. This implies that using a configuration path is exclusive to a local connection.
(this on line 281). A remote connection will be attempted whenever there is user or password or host or port
specified.

However, the default configuration file is loaded everytime? Yes, the configuration file is only loaded even if there is no
-h or -p or -U or -P. So in the current state of things, the configuration file cannot be specified while setting the -h
or -p, but the default file is loaded for other configuration. Is that the expected behaviour? I DO NOT UNDERSTAND THIS
BEHAVIOUR.

Also: I bet from line 300 on there can be improvement to avoid repetition.

## Is Alive (and other commands)

*[12:50]* Something I need to investigate is why I get the debug message

```
2024-03-04 12:50:08 WARN  management.c:1074 pgagroal_management_read_isalive: read: 3 Invalid argument
```

everytime i try to run a command. In this case it is for isalive. I have tried to run a bunch of different configurations
and so on. I will try to debug it.

Aparentemente eu preciso enable uma base de dados primeiro. Estranho, isso não está na documentação. VERIFICAR.

Now, I have done two types of connections, a local connection and a remote connection. When I try the local connection
the function pgagroal_connect_unix_socket is called and stablishes a connection with a unix socket directly.

If the remote connection is enabled, then the function pgagroal_connect is called to stablish a remote connection.
The connection is stablished correctly, otherwise the server would not be receiving the communication. However, I believe the
configuration is wrong, because the database it tries to connect to is called "admin" instead of "test".

## Approaching the error messaging system problem

*[21:00]* Sent my thoughts about how to approach the error messaging system problem. [thoughts](https://github.com/agroal/pgagroal/issues/403)

## March 11, 2023

*[12:00]* I have been thinking about a new approach to the error messaging system. And this is what i intend to describe with the following message:

"""
Thank you for your feedback, @fluca1978 .

My intention with this PR was to get your feedback on what you had imagined and what I could improved.

Considering I am on the right track, based on your message, I had ideas that could simplify this new command struct, but I did not want to implement because my intention was to modify the least amount of code that was already implemented.

I was thinking of, instead of assigning action, mode, log traces, having a callback function that would be called whenever a command was executed. This callback function would be responsible for the action, mode and log traces. This would simplify the command struct and the command execution.

The callback function would be called with the parsed command struct, that could contain more than arg_count and args.

I also wanted to remove the error_message from the command struct, because I believe it is not necessary. The error message could be printed directly to stderr.

This callback function would be defined as 

```c int (*action)(struct pgagroal_parsed_command* cmd)```

"""


## March 18, 2023

The PR has been merged. I will start working on the GSoC proposal.

## March 20, 2023

*[12:40]* I will start working on the GSoC proposal now, along with my favorite ghost writer.

The structure for the project is:

"""
1. Description of the Problem and Proposed Solution
2. Implementation Details
3. Timeline
4. Conclusion
"""

Let's start by adding information about the description of the problem.

Here is the description of the project by pgagroal.

```
    pgagroal: I/O layer
    Project Description
    Replace the existing I/O layer (libev based) with an in-place high-performance one.

    This will require writing a layer that abstracts epoll, io_uring, ... but keeping it at a minimum.

    350 hours

    Mentors
    Luca Ferrari (fluca1978 at gmail dot com)
    Jesper Pedersen (jesper dot pedersen at comcast dot net)
    Reach out for feedback

    References
    [1] https://github.com/agroal/pgagroal/
    [2] https://man7.org/linux/man-pages/man7/epoll.7.html
    [3] https://man.archlinux.org/man/io_uring.7.en
```

The pgagroal project understands that libev does not suffice its performance needs.

It would be nice to understand how the libev plays a role in pgagroal. How it takes most of its time.

Maybe I could launch perf for evaluating the performance of pgagroal. I will try this on clion.

Maybe try to use a flame graph.

The point is I need to benchmark the pooler. I will eventually have to benchmark it when I have to modify it,
so i might as well as benchmark it now to understand how it performs.

While i am crafting an email to Jesper and Luca, I am also reading some info online so I can understand how pgbench works.

Notice that pgbench was used by Jesper to evaluate the performance of pgagroal against other poolers.

It appears that pgbench is enough to do a good benchmark.

Some good references here:

1. https://medium.com/@dmitry.romanoff/how-to-measure-performance-of-postgresql-database-server-s-b27e2e5130aa
2. https://www.cloudbees.com/blog/tuning-postgresql-with-pgbench
3. https://medium.com/@c.ucanefe/pgbench-load-test-166bdfb5c75a
4. https://www.postgresql.org/docs/11/pgbench.html 

*[16:00]* Debugging to see the call stack and described the 
proposed modifications:

libev starts to be mentioned at line 939.

The code below implements the definition of the default loop.
This might be removed in the new implementation, as it will
comprise only one implementation.


```c
/* libev */
main_loop = ev_default_loop(pgagroal_libev(config->libev));
if (!main_loop)
{
pgagroal_log_fatal("pgagroal: No loop implementation (%x) (%x)",
pgagroal_libev(config->libev), ev_supported_backends());
#ifdef HAVE_LINUX
sd_notifyf(0, "STATUS=No loop implementation (%x) (%x)", pgagroal_libev(config->libev), ev_supported_backends());
#endif
goto error;
}
```

The following code implements signal_watcher that contains ev_signal
struct, which should be modified in the new implementation. The
signal_handlers will be implemented in the code using linux handlers.
We can keep the same function signature and keep the callbacks.
[TODO: search how these handlers are created]

```c
   ev_signal_init((struct ev_signal*)&signal_watcher[0], shutdown_cb, SIGTERM);
   ev_signal_init((struct ev_signal*)&signal_watcher[1], reload_cb, SIGHUP);
   ev_signal_init((struct ev_signal*)&signal_watcher[2], shutdown_cb, SIGINT);
   ev_signal_init((struct ev_signal*)&signal_watcher[3], graceful_cb, SIGTRAP);
   ev_signal_init((struct ev_signal*)&signal_watcher[4], coredump_cb, SIGABRT);
   ev_signal_init((struct ev_signal*)&signal_watcher[5], shutdown_cb, SIGALRM);

for (int i = 0; i < 6; i++)
{
signal_watcher[i].slot = -1;
ev_signal_start(main_loop, (struct ev_signal*)&signal_watcher[i]);
}

```

Next we have the following code, that is responsible for starting io_management and the loop.
This code will have to be entirely refactored.

```c

static void
start_mgt(void)
{
   memset(&io_mgt, 0, sizeof(struct accept_io));
   ev_io_init((struct ev_io*)&io_mgt, accept_mgt_cb, unix_management_socket, EV_READ);
   io_mgt.socket = unix_management_socket;
   io_mgt.argv = argv_ptr;
   ev_io_start(main_loop, (struct ev_io*)&io_mgt);
}

```


