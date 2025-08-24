- [admin](#admin)
  - [question mode](#question-mode)
  - [weekly schedule](#weekly-schedule)
  - [assessment](#assessment)
  - [topics of 4226](#topics-of-4226)
  - [plan:](#plan)
- [L1 Network Performance Models](#l1-network-performance-models)
  - [network perf metrics](#network-perf-metrics)
    - [bandwidth (R)](#bandwidth-r)
    - [throughput](#throughput)
    - [delay](#delay)
    - [response time](#response-time)
    - [others](#others)
  - [queueing delay](#queueing-delay)
    - [circuit vs packet switching](#circuit-vs-packet-switching)
  - [queueing model](#queueing-model)
    - [single server queueing model](#single-server-queueing-model)
    - [little's law](#littles-law)

# admin
Richard Ma - Prof
- tbma@comp.nus.edu.sg
Tiantong - TA
- tiantong.hu@u.nus.edu

## question mode
- raise questions in class, send emails after class
- arrange individual meetings (email)

## weekly schedule
tutorials released one week before tutorial. (one week after lecture)

## assessment
- written asst - 10%
  - given for free, need to try but no need to stress cos they dont grade (10% for free)
- prog assts - 35% << heavy on this, 50% of the time should be spent on this
- midterm - 20%
- final exam - 35%

## topics of 4226
- Network Archi
  - P2P networks
- Internet Ecosystem
  - InterDomain routing - BGP
  - Business and Economics 
- Network Management Paradigm
  - Software-Defined Networking (SDN)
- Network Performance (heavier on this part, more techniques + will test more in finals)
  - Modeling
  - Understand resource allocation, how it can affect performance

## plan: 
❑ Aug 15 - Logistics, Performance Metrics
❑ Aug 22 - Probability and Queueing Model
❑ Aug 29 - M/M/1 Model and Analysis
❑ Sep 5 - Queueing Networks
❑ Sep 12 - Network Resource Allocation
❑ Sep 19 - Introduction to SDN
❑ Sep 26 - Recess
❑ Oct 3 - Mid-term Exam (MPSH 5)
❑ Oct 10 - Software Defined Networking
❑ Oct 17 - Inter-Domain Routing Protocol
❑ Oct 24 - BGP Routing Policy
❑ Oct 31 - Internet Exchange Point
❑ Nov 7 - Peer-to-Peer Networks
❑ Nov 14 - Wrap-Up
❑ Nov 28 - Final Examo

# L1 Network Performance Models

## network perf metrics

### bandwidth (R)
- *R* = Link rate, Bandwidth, Capacity are interchangeable
  - unit: 
    - cable - 1~100Mbps, Fiber optics - 1~`0 Gbps
    - Ethernet - 3Mbps to 100Gbps
    - How many bits can be "pushed" **onto** a link per unit time - subtle difference from throughput
      - not across a link
  - related to **transmission delay**

### throughput
- Throughput of a **UDP/TCP flow** - how many bits can be **communicated** per unit time.
- a source and dest - throughput is e2e, after it goes through different routing paths.
  - how many bits can be sent through the entire flow. 
- related to propagation

### delay
- E2E delay: processing + queuing + transmission + propagation delay
- store and forward paradigm: nodes store bits and form packets before forwarding to the next node

### response time
- E2E delay x 2 (total time to respond)
- S to Dest and Dest to S may take very different routing paths (different E2E time)
  - hence need network perf analysis to estimate

### others
- dropping probability
  - the probability of packets being dropped
- utilisation
  - % of time the cpu is busy (not idle)



> [!NOTE]
> - Throughput: e2e
> - Link rate: the local point - at each link, how many bits per second can be pushed onto the link
>
> analogy:
> 
> Imagine moving house:
> - transmission delay: time to load the truck 
> - propagation delay: the time for the truck to move (transport the items)

## queueing delay

Origin of queueing delay: statistical multiplexing
- this multiplexing is where bandwidth is shared with hosts based on their demand
- input rate > output rate, queueing occurs.
  - packets wait for the output link
- TDM: time division multiplexing
  - time divided to slots for host A and B to use the same link

![](./L1-stat-multiplexing.png)


### circuit vs packet switching
- circuit switching: link is reserved for the users (landline phone)
- packet switching / statistical multiplexing: shared link

analogy of a restaurant
- circuit switching: reserved appointments (reservations only)
  - people might not show up - waste resources 
  - or cancel last minute, new requests might be rejected for nothing
  - hence: high 
- packet switching: queue 
  - if there's alot of people need to queue very long

> Eg. 
> - 1 Mb/s link
> - Each user: 
>   - 100kb/s when "active"
>   - active 10% of the time.
>
> Circuit switching: only 10 users, each take 10%
>
> Packet switching: at 35 users, P(X>10) < 0.0004 where X is the number of users that are active
>
> ?? how to come to this calc - binomial?

**packet switchinng**
- good for bursty data (fluctuating traffic)
- but: excessive congestion (input rate is too high)
  - this leads to ***queueing delay***
  
**circuit switching**
- good for traffic that is consistently high


## queueing model

### single server queueing model

![](L1-single-server-model.png)
- **modelled as** a server at the end of the queue (like a cashier at the end of a queue)
  - (but in reality it is the link)

### little's law
*L = r x t*

- **L** = Average number of customers in the system (**length of the queue**)
- **r** = **arrival rate** (input rate)
- **t** = average sojourn time (waiting/queueing time - **time they stay in the system**)


formally it is L = ƛW where ƛ is r, W is t

> to think about:
>
> A server with infinite queue and a capacity of 1.5Mb/s, can we predict / guess the queueing delay

