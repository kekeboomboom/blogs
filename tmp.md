# tmp



Packet-based load balancers are generally deployed in cut-through mode, so they are installed on the normal path of the traffic and divert it according to the configuration. The return traffic doesn't necessarily pass through the load balancer. Some modifications may be applied to the network destination address in order to direct the traffic to the proper destination. In this case, it is mandatory that the return traffic passes through the load balancer. If the routes doesn't make this possible, the load balancer may also replace the packets' source address with its own in order to force the return traffic to pass through it.



Proxy-based load balancers are deployed as a server with their own IP addresses
and ports, without architecture changes. Sometimes this requires to perform some
adaptations to the applications so that clients are properly directed to the
load balancer's IP address and not directly to the server's. Some load balancers
may have to adjust some servers' responses to make this possible (e.g. the HTTP
Location header field used in HTTP redirects). Some proxy-based load balancers
may intercept traffic for an address they don't own, and spoof the client's
address when connecting to the server. This allows them to be deployed as if
they were a regular router or firewall, in a cut-through mode very similar to
the packet based load balancers. This is particularly appreciated for products
which combine both packet mode and proxy mode. In this case DSR is obviously
still not possible and the return traffic still has to be routed back to the
load balancer.

A very scalable layered approach would consist in having a front router which
receives traffic from multiple load balanced links, and uses ECMP to distribute
this traffic to a first layer of multiple stateful packet-based load balancers
(L4). These L4 load balancers in turn pass the traffic to an even larger number
of proxy-based load balancers (L7), which have to parse the contents to decide
what server will ultimately receive the traffic.

The number of components and possible paths for the traffic increases the risk
of failure; in very large environments, it is even normal to permanently have
a few faulty components being fixed or replaced. Load balancing done without
awareness of the whole stack's health significantly degrades availability. For
this reason, any sane load balancer will verify that the components it intends
to deliver the traffic to are still alive and reachable, and it will stop
delivering traffic to faulty ones. This can be achieved using various methods.

The most common one consists in periodically sending probes to ensure the
component is still operational. These probes are called "health checks". They
must be representative of the type of failure to address. For example a ping-
based check will not detect that a web server has crashed and doesn't listen to
a port anymore, while a connection to the port will verify this, and a more
advanced request may even validate that the server still works and that the
database it relies on is still accessible. Health checks often involve a few
retries to cover for occasional measuring errors. The period between checks
must be small enough to ensure the faulty component is not used for too long
after an error occurs.

Other methods consist in sampling the production traffic sent to a destination
to observe if it is processed correctly or not, and to evict the components
which return inappropriate responses. However this requires to sacrifice a part
of the production traffic and this is not always acceptable. A combination of
these two mechanisms provides the best of both worlds, with both of them being
used to detect a fault, and only health checks to detect the end of the fault.
A last method involves centralized reporting : a central monitoring agent
periodically updates all load balancers about all components' state. This gives
a global view of the infrastructure to all components, though sometimes with
less accuracy or responsiveness. It's best suited for environments with many
load balancers and many servers.

Layer 7 load balancers also face another challenge known as stickiness or
persistence. The principle is that they generally have to direct multiple
subsequent requests or connections from a same origin (such as an end user) to
the same target. The best known example is the shopping cart on an online
store. If each click leads to a new connection, the user must always be sent
to the server which holds his shopping cart. Content-awareness makes it easier
to spot some elements in the request to identify the server to deliver it to,
but that's not always enough. For example if the source address is used as a
key to pick a server, it can be decided that a hash-based algorithm will be
used and that a given IP address will always be sent to the same server based
on a divide of the address by the number of available servers. But if one
server fails, the result changes and all users are suddenly sent to a different
server and lose their shopping cart. The solution against this issue consists
in memorizing the chosen target so that each time the same visitor is seen,
he's directed to the same server regardless of the number of available servers.
The information may be stored in the load balancer's memory, in which case it
may have to be replicated to other load balancers if it's not alone, or it may
be stored in the client's memory using various methods provided that the client
is able to present this information back with every request (cookie insertion,
redirection to a sub-domain, etc). This mechanism provides the extra benefit of
not having to rely on unstable or unevenly distributed information (such as the
source IP address). This is in fact the strongest reason to adopt a layer 7
load balancer instead of a layer 4 one.

In order to extract information such as a cookie, a host header field, a URL
or whatever, a load balancer may need to decrypt SSL/TLS traffic and even
possibly to re-encrypt it when passing it to the server. This expensive task
explains why in some high-traffic infrastructures, sometimes there may be a
lot of load balancers.

Since a layer 7 load balancer may perform a number of complex operations on the
traffic (decrypt, parse, modify, match cookies, decide what server to send to,
etc), it can definitely cause some trouble and will very commonly be accused of
being responsible for a lot of trouble that it only revealed. Often it will be
discovered that servers are unstable and periodically go up and down, or for
web servers, that they deliver pages with some hard-coded links forcing the
clients to connect directly to one specific server without passing via the load
balancer, or that they take ages to respond under high load causing timeouts.
That's why logging is an extremely important aspect of layer 7 load balancing.
Once a trouble is reported, it is important to figure if the load balancer took
a wrong decision and if so why so that it doesn't happen anymore.

