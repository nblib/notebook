#Description
In the field of Linux Scoket programming, non blocking I/O and multiplexing are needed to deal with a 
large number of connection request scenarios. Select,poll and epoll are the I/O multiplexing methods provided by Linux API.
Since the Linux2.6 was added to epoll, the high-performance server field has been widely used.
#select model
The file descriptors monitored by the select function fall into 3 categories, namely writefds, readfds and exceptfds.
After calling, the select function blocks until the descriptor is ready or timed out. When the select function returns, the
ready descriptor is found by traversing the fd_set.

### weakness
One drawback of select is that the number of file descriptors that can be monitored by a single process has a maximum limit,
and is generally 1024 on Linux.
# poll model
There is no maximum quantity limit,just like select function, the ready descriptor is found by traversing the pollfd
#epoll model
### Advantage
* The number of descriptors monitored is unrestricted.
* The efficiency of IO will not decrease as the number of monitors increases.
* Two modes of support level trigger and edge trigger.
* The kernel and user space MMAP the same memory.