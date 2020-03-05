# Elastic Load Balancer
Distribute incoming traffic across multiple targets (e.g. EC2 instances, containers, IP addresses, lambda functions).

## Types of Load Balancers
* ### Application Load Balancer (ALB)
Operates at the Application Layer (OSI L7), handling HTTP/HTTPS traffic. Allows routing requests to specific web servers.

* ### Network Load Balancer (NLB)
Operates at the Network Layer (OSI L4), handling TCP traffic. Recommended for performance.

* ### Classic Load Balancer (CLB)
Operates at OSI L7 and OSI L4, however has limited functions. Not recommended for use except for apps built in the EC2-Classic network.

Because load balancers intercept traffic between clients and servers, your server access logs contain only the IP address of the load balancer. To get the IP address of the end user, check the **X-Forwarded-For** request header that gets passed to the server.

## Troubleshooting
**504 Error** means the gateway has timed out i.e. the application it's pointing to hasn't responded within the idle timeout period. The ELB isn't broken; check the web server or database.
