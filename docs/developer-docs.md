# dynomite

# Connections

Dynomite has 6 connection types. Three are inherited from Twemproxy and three are new to Dynomite:

- **Proxy**: listens for client connections (default: 8102)
- **Client**: incoming connection from the client
- **Server**: outgoing connection to the underlying data store.
- **Dnode Peer Proxy**: listens to connections from other dynomite node (default 8101)
- **Dnode Peer Client**: incoming connection from other dnode
- **Dnode Peer Server**: outgoing connection to other dnode.

# 1. dynomite.c

```c
main()
	dn_coredump_init(); // set unlimited dump core resource limits.
	dn_set_default_options(&nci); // set default values for dynomite instance
	dn_get_options(argc, argv, &nci); // get the CLI options
	dn_pre_run(&nci); // setup before running dynomite: logging, pid, daemonize, signals
	dn_run(&nci); // start dynomite (move to 2)
	dn_post_run(&nci); // cleanup when shutting down dynomite
```

# 2. dynomite.c
```c
dn_run(struct instance *nci)
	core_start(nci); // initialize buffers, messages, connection
	core_loop(ctx); // primary loop to process messages (move to 3)
	core_stop(ctx); // deinitialize connections, message queue, memory buffers, destroy context (run @shutdown only)
```

# 3. dyn_core.c

```c
core_loop(struct context *ctx)
	core_process_messages();
```


Questions
1. dyn_queue.h line 313
- What is `#define STAILQ_INIT(head) do {`?

2. dyn_queue.h line 271
- What is `#define STAILQ_HEAD(name, type)`?
- What is the point of **

3. dyn_message.c line 651
- What is the use of a red-black tree here?

4. dyn_core.c line 539
- Why is the call back set to msg itself? `msg->cb(msg);`


# Parse dynomite.yaml: `src/dyn_conf.c`

```c
sp->dnode_proxy_endpoint.pname = cp->dyn_listen.pname;
...
```

`dnode_proxy_endpoint` is used in:

- dyn_conf.c
- dyn_core.h
- dyn_dnode_msg.c
- dyn_dnode_peer.c
- dyn_dnode_proxy.c

## gossip

```c
sp->g_interval = cp->gos_interval;
```

## seeds

```c
sp->seed_provider = cp->dyn_seed_provider;
```

`sp->seed_provider` is only used in:

- `dyn_gossip.c`



In `dyn_gossip.c`, `gossip_set_seeds_provider()` sets `gn_pool.seeds_provider = NULL;`.

## listen

```c
sp->proxy_endpoint.pname = cp->listen.pname;
sp->proxy_endpoint.port = (uint16_t)cp->listen.port;
...
sp->dnode_proxy_endpoint.pname = cp->dyn_listen.pname;
sp->dnode_proxy_endpoint.port = (uint16_t)cp->dyn_listen.port;
```

### proxy_endpoint

`proxy_endpoint` is used in:

- ./src/dyn_proxy.c

## timeout

```c
sp->timeout = cp->timeout;
```

# dyn_server.h

server_pool is a collection of servers and their continuum. 

Each
server_pool is the owner of a single proxy connection and one or more client connections. server_pool itself is owned by the current
context.

Each server is the owner of one or more server connections. server itself is owned by the server_pool.
 
# dyn_connection.h

In twemproxy there are 3 types of connections:

- `PROXY`: listens for client connections (default: 8102)
- `CLIENT`: incoming connection from the client
- `SERVER`: outgoing connection to the underlying data store.

Dynomite extended this same concept and added 3 other types of connections:

- `DNODE_PEER_PROXY`: listens to connections from other dynomite node (default 8101)
- `DNODE_PEER_CLIENT`: incoming connection from other dnode
- `DNODE_PEER_SERVER`: outgoing connection to other dnode.

# dyn_dict.h

Hash Tables Implementation.

This file implements in-memory hash tables with insert/del/replace/find/
get-random-element operations. Hash tables will auto-resize if needed
tables of power of two in size are used, collisions are handled by
chaining. See the source code for more information... :)
 
# dyn_queue.h

This file defines five types of data structures: singly-linked lists,
singly-linked tail queues, lists, tail queues, and circular queues.

A singly-linked list is headed by a single forward pointer. The elements
are singly linked for minimum space and pointer manipulation overhead at
the expense of O(n) removal for arbitrary elements. New elements can be
added to the list after an existing element or at the head of the list.
Elements being removed from the head of the list should use the explicit
macro for this purpose for optimum efficiency. A singly-linked list may
only be traversed in the forward direction.  Singly-linked lists are ideal
for applications with large datasets and few or no removals or for
implementing a LIFO queue.

A singly-linked tail queue is headed by a pair of pointers, one to the
head of the list and the other to the tail of the list. The elements are
singly linked for minimum space and pointer manipulation overhead at the
expense of O(n) removal for arbitrary elements. New elements can be added
to the list after an existing element, at the head of the list, or at the
end of the list. Elements being removed from the head of the tail queue
should use the explicit macro for this purpose for optimum efficiency.
A singly-linked tail queue may only be traversed in the forward direction.
Singly-linked tail queues are ideal for applications with large datasets
and few or no removals or for implementing a FIFO queue.

A list is headed by a single forward pointer (or an array of forward
pointers for a hash table header). The elements are doubly linked
so that an arbitrary element can be removed without a need to
traverse the list. New elements can be added to the list before
or after an existing element or at the head of the list. A list
may only be traversed in the forward direction.

A tail queue is headed by a pair of pointers, one to the head of the
list and the other to the tail of the list. The elements are doubly
linked so that an arbitrary element can be removed without a need to
traverse the list. New elements can be added to the list before or
after an existing element, at the head of the list, or at the end of
the list. A tail queue may be traversed in either direction.

A circle queue is headed by a pair of pointers, one to the head of the
list and the other to the tail of the list. The elements are doubly
linked so that an arbitrary element can be removed without a need to
traverse the list. New elements can be added to the list before or after
an existing element, at the head of the list, or at the end of the list.
A circle queue may be traversed in either direction, but has a more
complex end of list detection.

For details on the use of these macros, see the queue(3) manual page.

```bash
                     SLIST   LIST    STAILQ  TAILQ   CIRCLEQ
_HEAD                +       +       +       +       +
_HEAD_INITIALIZER    +       +       +       +       +
_ENTRY               +       +       +       +       +
_INIT                +       +       +       +       +
_EMPTY               +       +       +       +       +
_FIRST               +       +       +       +       +
_NEXT                +       +       +       +       +
_PREV                -       -       -       +       +
_LAST                -       -       +       +       +
_FOREACH             +       +       +       +       +
_FOREACH_REVERSE     -       -       -       +       +
_INSERT_HEAD         +       +       +       +       +
_INSERT_BEFORE       -       +       -       +       +
_INSERT_AFTER        +       +       +       +       +
_INSERT_TAIL         -       -       +       +       +
_REMOVE_HEAD         +       -       +       -       -
_REMOVE              +       +       +       +       +
```
 