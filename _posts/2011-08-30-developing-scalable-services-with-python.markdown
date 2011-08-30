---
title: Developing scalable services with Python
layout: post
time: '22:08'
---

<!--begin excerpt-->
Developing multi-threaded applications in python is a "Pain In The Ass". And the [GIL](http://wiki.python.org/moin/GlobalInterpreterLock) (Global Interpreter Lock) takes away the advantage of utilizing multiple cores in a machine. It doesn't matter how many cores a CPU have, GIL prevents threads from running in multiple cores. So python programs would't get the maximum performance out of the CPU when they use threads in their services.
<!--end excerpt-->

In many cases you might need to write services where you need to listen on a port and wait for the client connection to do some task. If multiple clients are connecting to this service simultaneously then you might need to spawn threads to handle the requests. Considering the fact that GIL introduces a performance bottleneck, the best way to solve this situation is to use python's [multiprocessing](http://docs.python.org/library/multiprocessing.html) capabilities. This library provides almost `threading` like class implementations. 

Then a question might arise. How do you share a socket created by the server process to the newly spawned processes? It is possible since all the `fork`ed processes will have the parent's file descriptors. `multiprocessing` library already has this package named `multiprocessing.reduction` that provides a method `reduce_handle` which can serialize a socket and you can send this socket to another process using pipes. The child processes can read from the pipe and re-create the socket using `rebuild_handle`. The following example will make this idea clear to you.

{% highlight python %}
# Main Process
from multiprocessing.reduction import reduce_handle
# serialize the socket
serialized_socket = reduce_handle(client_socket.fileno())
# send it to the child/worker process
pipe_to_worker.send(serialized_socket)

# Worker Process
from multiprocessing.reduction import rebuild_handle
# get the socket from parent
serialized_socket = pipe_from_parent.recv()
# rebuild the file descriptor
fd = rebuild_handle(serialized_socket)
# create socket from fd
client_socket = socket.fromfd(fd, socket.AF_INET, socket.SOCK_STREAM)
# use the socket as usual. eg: send a message to the client
client_socket.send("Baby, I\'m so fast\r\n")
{% endhighlight %}

I created a simple wrapper for this kind of services which can be scaled. You can find it here.