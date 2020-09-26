---
published: true
title: Networking Refresher 
use_math: true
category: dev
layout: default
---
# Networking Refresher

# Table of Contents

* TOC
{:toc}

# TCP/IP Protocols

## HTTP(S)

- Format of web content
- Sends content to client without maintaining state information. Considered a "PULL" protocol.
- HTTP 1.0 is 1 TCP connection, setup/teardown, per object in web page.
- HTTP 1.1 is 1 TCP connection for all objects in web page sequentially.
- HTTP 2.0 is HTTP 1.1 with pipelining(multithreaded transactions under a single connection).
- HTTPS is just HTTP running in SSL. Requires SSL certificate, and end-to-end encryption using public key.

## FTP

- 2 TCP connections - called **out of band**. 1 for state keeping, 1 for data stream.
- Main file transfer protocol

## SMTP

- Main mail **sending** protocol
    - Mail stays in sender's server if receiver's mail server is down.
- Considered a "PUSH" protocol.

# UDP Protocols

## DNS

- Translates hostname to IP.
    - Single domain name can have many IP's - distributed service, or aliasing.
- UDP call to authoritative servers, who then UDP call to more specific servers covering specific url namespaces. This is recursive.
- Usually DNS caching server to handle load on authoritative servers.

## P2P Basics

- CDN's, Content delivery networks, minimize distance between origin servers and clients by replicating origin server resources.

### Comparison of P2P & Client/Server

Given $N$ clients, a file of size $F$, and servers' upload rate of $u_s$, and i-th client's upload rate of $u_i$, 
and minimum download speed of any client to be $d_{min}$, we have this for the client/server architecture:

$$max(\frac{NF}{u_s}, \frac{F}{d_{min}})$$

For P2P, we have:

$$max(\frac{NF}{u_s + \sum_i u_i}, \frac{F}{d_{min}}, \frac{F}{u_s})$$

since we have the time it takes to distribute N copies of files, using upload speeds of all computers, as well as the minimum 
download time bottleneck, and the shards of files initially required to send to each client, we get the respective parts of the equation above.

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
