---
layout: post
title: Interview Question - What Happens When You Visit a Web Page
category: Misc
---
# Interview Question

To me, interview questions can be considered a form of testing. While reading [A Guide to Effective Studying and Learning](https://www.amazon.com/Guide-Effective-Studying-Learning-Strategies/dp/0190214473) by Matthew Rhodes, Anne Cleary and Edward DeLosh it is understood that testing is a great way of improving ones knowledge. With this in mind, I thought it would be good to test my knowledge on a particular interview question. However, on my journey to answering this I want to develop my knowledge further and more in-depth.

# The Question at Hand

What happens when you visit a web page?

# Purpose of the Question

The purpose of this blog post is to develop an understanding on what happens in the underlying architectures when visiting a web page. As this can be answered in many ways, my approach is to go form a top down approach.
The following flow demonstrates the topics which will be talked about throughout this post. As there will be quiet a few sections of technologies being utilised, I will be using material that is further in-depth to help support my logic, but summarising them in accordance and connecting them together in a logical manner. For further information on those areas please see the topics.

# Topology

The following topology demonstrates a end to end request for a user traversing to a web page.

![Topology](/images/interview_web_page/topology.png "topology")

# Browser

Of the two core browsers that are seen on most individuals machines, Firefox and Chromium based make up the masses.
However, to say that Firefox is close to the overall leader is far from true. W3Schools which gets an average of [50 Million views per-month](https://www.w3schools.com/browsers/default.asp), Chrome is situated at 79% of the overall distribution, which is further supported by NetMarketShare with a [65% distribution as of March 2019](https://netmarketshare.com/browser-market-share.aspx?options=%7B%22filter%22%3A%7B%22%24and%22%3A%5B%7B%22deviceType%22%3A%7B%22%24in%22%3A%5B%22Desktop%2Flaptop%22%5D%7D%7D%5D%7D%2C%22dateLabel%22%3A%22Trend%22%2C%22attributes%22%3A%22share%22%2C%22group%22%3A%22browser%22%2C%22sort%22%3A%7B%22share%22%3A-1%7D%2C%22id%22%3A%22browsersDesktop%22%2C%22dateInterval%22%3A%22Monthly%22%2C%22dateStart%22%3A%222018-04%22%2C%22dateEnd%22%3A%222019-03%22%2C%22segments%22%3A%22-1000%22%7D) and Statistica which shows Chrome being the most popular desktop browser installation at a [70% installation as of November 2018](https://www.statista.com/statistics/544400/market-share-of-internet-browsers-desktop/).

## How Does Chrome Work

This section will introduce but summaries portions of the overall architecture as outlined within [Mariko Kosaka's Inside look at modern web browser](https://developers.google.com/web/updates/2018/09/inside-browser-part1) series, this series focuses purely on the Chrome browser.

[Part 1](https://developers.google.com/web/updates/2018/09/inside-browser-part1) - Focuses on the underlying fundamentals of the operating system, with a very good introduction to Multi-Process Architecture and the security enhancements it brings.

[Part 2](https://developers.google.com/web/updates/2018/09/inside-browser-part2) - Focuses on the steps in which navigation takes place within the Chrome web browser.

[Part 3](https://developers.google.com/web/updates/2018/09/inside-browser-part3) - Focuses on the rendering pipeline. This presents the inner workings of what is being displayed on the open tab.

[Part 4](https://developers.google.com/web/updates/2018/09/inside-browser-part4) - Focuses on the Compositor to improve user experience.

### Fundamentals

Before continuing it is worth describing what a process and a thread is. The following is a VERY brief description of these two topics.

* Process - A process is an instance of a running program for example, Chrome is a process instance, Word is a process instance and VMWare is another instance. If there are multiple users on a host running multiple instances of Chrome, then there will be multiple processes of Chrome (One for each person) [[1]](http://heather.cs.ucdavis.edu/~matloff/UnixAndC/Unix/Processes.pdf)

* Threads - A thread is an execution block within a process. A thread has its own instructions and area of memory within the processes memory space. This can allow for a programs functionality to be executed simultaneously. It can be analogised as a micro-process within the process itself [[2]](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSPR/Reference/Threads)

The traditional method of running applications is to utilise one process with higher counts of threads, however, the use of the Inter Process Communication (IPC) mechanism is an alternative.

The IPC works on the basis of running multiple processes with few threads. This is beneficial with modern browsers as the IPC allows large applications to execute processes for different functionality. For example, modern web browsers will rely on the distribution of processes for functionality such as, networking, graphics processing and file system utility (just to name as few).

The use of the IPC can be seen within the browser’s tabs. Each tab is a new process, with the overarching master process being the Browser Process and its child the GPU Process.

![Tabs](/images/interview_web_page/web_page_tabs.png "Opened Tabs")

These tabs can then be seen within the Task Manager of Chrome. The Task Manager can be found through the Main Menu -> More Tools -> Task Manager.

![TaskManager](/images/interview_web_page/task_manager_tabs.png "Task Manager Tabs")

The use of the IPC can mitigate the issue of application hangs. For example, if you had one process with multiple threads for Networking and File Systems Utility and the process hangs, then the application crashes. Utilising the IPC can reduce the issue of overall application crashes as if one process fails then the application can continue.

As performance is the major contender for the switch to IPC, it brings security enhancements as well. The browser breaks away from sharing **all** data between each tab(process and thread) which helps add to the defence-in-depth mentality to prevent [CSRF attacks](https://www.exit.wtf/vrd/2019/04/04/Insecure_Deserialization.html). This is then further extended with the [Same Origin Policy](https://www.w3.org/Security/wiki/Same_Origin_Policy) and can be coupled with secure coding practices.

### Navigation

Part 2 is the key to answering the proposed question. Navigation works in my mind similar to how the Model-View-Controller pattern works within web-based frameworks. We have a master process (Browser process), with child threads (Network and Renderer Process) for functionality which outputs to our rendered view on the browser.

#### Browser Process

The master process is referred to as the Browser Process. The address bar within the UI Thread will be interpreted as either a URI or search query for Google's search engine. If the URI is correct (follows a naming conventions) then the Browser Process will trigger a Network Thread.

#### Network Thread

The Network Thread is the key component in answering the proposed question. The Network Thread performs the heavy lifting within the transfer and receiving of data to be rendered and utilised. Once the browsers URI is committed to the Network Thread, a DNS look up will be generated via the DNS Process [[3]](https://www.cloudflare.com/learning/dns/what-is-dns/) and then a TLS data stream is established.

The data streams first initial packet **Content-Type** headers MIME will be checked to determine if the data is to be rendered or downloaded. This is determined by the following Content-Types:

Content-Type: "text/plain"
or
Content-Type: "application/octet-stream"

If the Content-Type is equal to "application/octet-stream", then we can pass the content towards the "Download Manager Thread". If however, the Content-Type is equal to "text/plain" then we can pass the data stream towards the Renderer Thread.

After the MIME has been determined and the data stream is beginning to be passed to the Render Process, a SafeBrowser check is made. SafeBrowsing is the process in which the content is flagged as malicious. As well as a [Cross-Origin Read Blocking](https://www.chromium.org/Home/chromium-security/corb-for-developers) is carried out. It is worth reading into Cross-Origin Read Blocking as it helps mitigate side-channel attacks such as Spectre, Meltdown and potentially RowHammer.

Finally the network Thread is passed back to the UI Thread to be passed towards an open Rendering Process.

### DNS Caching
DNS Caching is a common approach to increase speed when browsing. Chormium-based web browsers utilise the DNS Pre-fetching mechanic to pre-emptively gather the required destination before a user may need to. Pre-fetching utilises the same mechanics as a standard DNS request, but does __NOT__ use the browsers networking stack. This is to allow compatibility across all Operating System network stacks.

To perform this Pre-fetch any web page that is loaded that has Hyperlinks, the Chromium browser will resolve all IP addresses linked to that web page. This process can take significant time, in doing so eight asynchronous threads are created purely for DNS Pre-fetching to manage the wait times of all the DNS requests. This approach results in threads not being the cause of a bottleneck throughout the Operating Systems network stack.

The results are then stored within the Browsers DNS Cache which can be found under the following url:
[chrome://net-internals/#hsts](chrome://net-internals/#hsts)

![DNSCache](/images/interview_web_page/dnscache.png "DNSCache")

While this approach works well with HTTP traffic, it does not work with HTTPS traffic. This is a security measure to prevent man-in-the-middle attacks. Finally, the last characteristic of the DNS Pre-fetching is that on reloading of the browser, the last 10 domains are remembered and stored. This is to allow for loading of previous closed tabs.

For further information on DNS Pre-fetching see: [DNS Pre-fetching](https://www.chromium.org/developers/design-documents/dns-prefetching)

#### Render Process

Once the UI Thread has received the appropriate call, the Browser Process will communicate through the IPC to a Render Process. This process will commit the open data stream from the Browser process across to the open Render Process. Once passed, the document is rendered into the browser pane and a commit is sent back to the browser for confirmation and the page has been loaded.

The following section of the post focuses on the redirection and leaving a web page if the content is dynamic within the page. This process although interesting doesn't directly link to the question at hand.

### Rendering

Rendering is the core stepping stone in which code segments become what you see within this very tab. The Render Process has a layer of parsing in which it generated a Document Object Model also referred to as the DOM from the transmitted HTML structure.

In addition, to generate the colours and dynamic content via JavaScript, a concurrent pre-load scan is run via the network thread. Concurrency is key within this section as long drawn out processes and threads can delay the time it takes to generate the overall content.

The DOM Tree is our overall functionality and content of the web page, however, before the Render Process can display this information, the DOM tree is combined with a Layout Tree. In order to do this at speed, a main worker thread is sent down the DOM structure to determine the overall geometry of the web page. Additional structures such as colour and functionality are calculated and added to the overall Layout Structure. Finally, to generate a layered structure, each leaf of the Layout Tree is further split into a Layer Tree. This will finally be converted and presented to the screen, the term of this is referred to as rasterization.

For more information on the rendering processes and compositor process I would suggest reading part four of Mariko Kosaka's blog post, as they are extremely interesting. However, for the portion of this part of the question I feel as If part 1,2 and 3 satisfy the overall understanding of the browser back-end.


## DNS
Before further looking into DNS, it is worth noting early on key terminology to better understand what is being referred to. The following list is the key terminologies regarding DNS:

* Stub Resolver - A Local copy of DNS records within the Browser or Operating System

* DNS Recursor - A system situated within an organisation e.g., ISP, OpenDNS, Cloudflare. These organisations requests information against the root, Top Level and Second Level Domains and provide it back to the user that made the DNS query at the start

* Root Level Domain - Thirteen servers, that are the highest point of the DNS hierarchy which provides locations for authoritative nameservers below.

* Authoritative Nameserver - Are pointed to via the Top Level Domain Server. These servers hold information regarding to exit.wtf

* Nameserver - A computer system that contains information and resources regarding CNAME's, MX, NS, A or AAAA records

* Top Level Domains - The first tier below the root level domain, such domains are .com, .net, .ru, .cn, .wtf. These servers point towards an Authorative Nameserver

* DNS Query - The request which is sent to determine the IP address of a CNAME

* Recursive Query - A query sent typically via the Recursive Resolver which contains the appropriate Resource or an error message

* Non-Recursive Query - A standard query from an authoritative position for example, an authoritative server requesting a resource or its local copy

* Iterative Query - A query, that will determine a given 'best answer' for the request. If however, this information is not correct, the DNS server will respond with an appropriate DNS server located one level below. This process is iterated until the correct result is found

* Caching - The technique in keeping a local copy of an associating resource

* CNAME- Also known as a Conical Name, is the typical search address e.g., www.exit.wtf

* A Record - An IPv4 address associated to a CNAME

* AAAA Record - Pronounced Quad-A, is the term referred to for the IPv6 address of the CNAME

* MX Record - A Mail Exchanger Record is a record which takes email based on behalf of the Domain

* NS Record - This is the associating Nameserver record to our A or AAAA record

* TTL - Also known as Time-To-Live, this determines the lifespan of data being sent

Now that we understand what is being discussed we can continue. Following our Network Thread from the Browser section. We first begin to translate our location such as www.exit.wtf to the associated IP address. Because conical names are easier to remember, a DNS provides us a mapping for the conical name to our associating IP addresses. In DNS terminology, the conical names are known as CNAMES and our IPv4 addresses are A records and the IPv6 addresses are referred to as AAAA(quad-a) records. Using the [nslookup](https://docs.exit.wtf/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc725991(v%3Dws.11)) tool, we can see the associating CNAME and A record.

![nslookup](/images/interview_web_page/nslookup.png "nslookup")

The following scapy program can be used to visualise what the nslookup command carries out, you will require to install graphviz to visualise your graph:
```
from scapy.all import *
hosts = "exit.wtf"
res, unans = traceroute(hosts)
res.graph(target="> traceroute_graph.svg")
```
The following outputs a svg graph as shown below

![DNStraceroute](/images/interview_web_page/dnstraceroute.svg "DNS Traceroute Graph")

Although simple, nslookup does quiet a large amount of heavy lifting for us to be able to visit a web page. The nslookup tool which carries out our DNS query can be broken down into nine core steps [[4]](https://royal.pingdom.com/a-visual-explanation-of-how-dns-lookups-work/), [[5]](https://www.cloudflare.com/learning/dns/what-is-dns/):

1. Using our browser we type in our URI www.exit.wtf the query is then passed down to its network thread, and then sent to our DNS process

2. The query through the browser into the inner workings of our Operating System to the stub resolver and its cache for the requested CNAME's, A record

3. If the stub-resolver does not have an A record, it is then passed to the Recursive Resolver which can be situated within a local ISP. The Recursive resolver then queries one of thirteen DNS root nameserver's also represented as (.)

4. One of the thirteen root servers then responds to the Recursive Resolver with the address of a Top Level Domain DNS server e.g., .com, .net or .ru. These Top Level Domain nameservers stores the information regarding any website that is registered under the .com flag. For example, google.com, cisco.com would be situated under the Top Level Domain nameserver .com, where as exit.wtf will be situated under the Top Level Domain nameserver .wtf

5.  The resolver then makes a request to the appropriate Top Level Domain, in this case .wtf

6. The TLD server (.wtf) then responds with the IP address of the exit's domain nameserver, exit.wtf

7. Knowing the final lookup address, the recursive resolver sends a query to the exit.wtf's nameserver

8. The IP address for www.exit.wtf is then returned to the resolver from the nameserver

9. The DNS resolver then responds to the web browser with the IP address of the domain requested initially

The following image presents a graphical overview of the steps above.

![DNSDiagram](/images/interview_web_page/dnsdiagram.png "DNS Transaction")

Once the query as been gathered and passed back to the web browser, a HTTP request for the appropriate web page can be issued to the web server of www.exit.wtf.

## HTTP
As the user now has the address of the appropriate web server, a HTTP request can be sent to the server. Once the server has accepted the request, it analyses the web directory for a file requested within the URL.

Using the following example: http://www.exit.wtf/Windows_Diffing/ the Windows_Diffing is searched within the file systems \_posts directory, if it cannot be found it is then recursively searched in the other directories until it is found.

![WindowsDiffing](/images/interview_web_page/diffing.png "Windows Diffing")

As the post is a markdown page, the content is Static not Dynamic.

* Static Web Sites - The web content is sent 'as-is', due to the content not requiring a data stream for consistent change. It is also static as there is no back-end database.

* Dynamic Web Sites - The web content is sent across through a data stream through the Application Server situated on the Web Server. Examples of Application Servers are Django, Reactive.js, Jekyll, Laravel and CodeIgniter, there are also other Content Management Systems to support content delivery.

As the content is static, the server will send the full page to the browser. If, however, it does not, the application server will build and send the content back to the browser. If, however, the content cannot be found, the server will send a 404 error.

![404](/images/interview_web_page/404.png "404")

Once the file has been received by the Browser, the Network Process passes it to the Main Browser thread which is passed to the Render Process as described above.

Finally, we have a rendered web-page as shown below.

![WebPage](/images/interview_web_page/webpage.png "WebPage")


# HTTPS Handshake
One final part to the picture, is encryption. Encryption is continuously being adopted across the field of Computing and technology consumption. This is partly due to the wide scope of surveillance and end-user privacy. As large portions of the Internet utilise plain text communication, the adoption of HTTPS has become wide-spread. HTTPS Also referred to as HyperText Transfer Protocol with Secure Socket Layer (SSL) is the approach in which organisations and privacy advocates layer security on top of web traffic.

SSL was first implemented in 1994, but much has been done to develop and secure it to meet the needs of today's needs. When SSL is talked about today, we talk about the Transport Layer Security (TLS) mechanism. SSL/TLS is the approach of encrypting plain text communications with strong public key encryption. Before continuing further, a piece of key cryptography information needs to be discussed:

* Symmetric Encryption - The use of encrypting data and decrypting data with the same encryption key and mathematical function

* Asymmetric Encryption - The use of Private key to encrypt data and a Public key to decrypt data with the same mathematical function

Due to not being the owner of the server side of a request, symmetric key encryption is not a strong candidate for HTTPS. Therefore, the use of Asymmetric encryption (Public Key Infrastructure (PKI)). However, it is common to ask, how can the public key decrypt the private key? Commonly, a public and private key are generated together (this is not always the case though) and both define a relative relationship between each other.

## How Does PKI Work?
As this topic can be broken out into a wider blog post, it is worth discussing how it works in a very high-level view.

* Like all communication protocols, a request is made from the Client to the respective Server
* The Server responds with an a _public key_, while keeping its _private key_ secret
* Using the provided _public key_, a _session key_ is encrypted and transferred to the Server from the Client, this process creates our __Symmetric Key Encryption__ mechanism to be used later on
* The Server decrypts the session key using the _private key_
* Finally, __Asymmetric Encryption__ is dropped and the _session key_ is used to establish __Symmetric Encryption__

This process is first established before any HTTP traffic is delivered to the client from the server.

The following graphic provides a graphical representation of the steps listed above.

![PKI](/images/interview_web_page/pki.png "PKI Transaction")

For further information on this process, [Analysis of the HTTPS Certificate Ecosystem](chrome-extension://oemmndcbldboiebfnladdacbdfmadadm/http://delivery.acm.org/10.1145/2510000/2504755/p291-durumeric.pdf?ip=87.239.255.103&id=2504755&acc=OA&key=4D4702B0C3E38B35%2E4D4702B0C3E38B35%2E4D4702B0C3E38B35%2E595DDC89FD3F921D&__acm__=1556482942_1035b21b9679a1f9b373ddf32b054762), [CloudFlare SSL](https://www.cloudflare.com/learning/ssl/how-does-ssl-work/), [Mozilla: HTTP and Your Online Security](https://blog.mozilla.org/internetcitizen/2017/04/21/https-protect/), [Google](https://support.google.com/webmasters/answer/6073543?hl=en).

# Conclusion
While this has covered only a partial amount of information there is many strands in which you can answer the question "What happens when you visit a web page?". Covering the key areas of Browser functionality, DNS and HTTPS/PKI, a breadth of knowledge is discussed such as, Networking/Network Infrastructure (DNS), System Internals (Browsers: Processes, Threads, Caching and Rendering) and Encryption (HTTPS/PKI). While a large portion of this touches on these topics, writing this blog post has provided a gateway into these areas for myself, I therefore, encourage anyone to challenge themselves to write a detailed answer towards an Interview-based question.

Thank you for reading.

_~eXit_
