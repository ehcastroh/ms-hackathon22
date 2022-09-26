# MS-HACKATHON-2022

Improving Customer Onboarding Experience Using Discrete Event Simulation

The following describes the problem, constraints, and assumption made when simulating ATS' onboarding process as people operating in a simulated system consisting of two sequential services within a queueing system. The simulation is meant to represent current conditions and not a future state or process. The document is broken into the following sections:

1. [Problem Formulation and Study Design](#problem-formulation-and-study-design)
2. [Collecting Data and Defining the Model](#collecting-data-and-defining-the-model)
3. [Documentation of Assumptions](#assumptions-document)
4. [Discrete Event Simulation using SimPy](#discrete-event-simulation-using-simpy)
5. [Production Runs](#production-runs)
6. [Analysis](#analysis)
7. [Recommended Actions and Next Steps](#recommended-actions-and-next-steps)


## Problem Formulation and Study Design

A customer wanting to use ATS (ATS.Service) must complete a multi-step sequence of steps that may take several weeks to finalize. This study aims to identify, understand ways and, if possible, to improve the customer's experience during the ATS onboarding process -- either by identifying areas where the system could be improved or by improving the experience in ares where the system, as it stands, cannot be improved.

Whenever a 1P customer wants to onboard to ATS' solutions they complete a process that can be broken down to two main sub-processes: 

1. **Stating Interest** -- generally involves a back-and-forth exchange of requesting information about solution, then proving ATS will all the necessary information to onboard to solution. 
2. **Tenant Onboarding** - the process of using the customer's provided information to deploy solutions.


**Note:** The following terms are used throughout the document:

- **Customer** - a customer is a any service wanting to onboard to ATS Solution
- **Server** - an on-call engineer that is responsible for completing the customer onboarding
- **Service** - the act of onboarding a Customer, performed by a Server
- **Event** - an instantaneous occurrence that may change the system state
- **Arrival Rate ($\lambda$)** is the number of customers arriving per unit time -- a Poisson process at a rate of $\lambda = 1/E(A)$, where $A$ is the arrival time.
  - e.g., 3 customers per week.
- **Service Rate ($mu$)** is the number of customers that a single server can serve per unit time -- each customer requires an exponentially distributed time period with an average service rate of $\mu = 1/E(S)$, where $S$ is the time it takes to complete a service.
  - e.g. 1 week per customer => 4 customers per month.
- **Number of servers ($c$)** is the number of servers that can serve a customer -- that is, perform the necessary tasks to move the customer past servicing and having them exit the system. 

### Stating Interest

The first sub-process of the ATS customer flow can be broken down to the following steps:

1. **START:** A customer arrives at ATS' site. They may have found the site through word of mouth, by searching, or by being redirected there by an ATS team member.
2. The customer spends some time familiarizing themselves on the information and actions that will be required of them. This annealing process is generally involves multiple email exchanges between the customer and ATS engineer. 
3. **END:** the Stating Interest process ends when a customer submits the required information for an on-call engineering to onboard customer to ATS.

For details and assumption on the modeling approach for the Stating Interest sub-process, see [Modeling Stating Interest System Using GI/G/s Queue](#modeling-stating-interest-system-using-gigs-queue)

### Tenant Onboarding

The second sub-process, Tenant Onboarding, can be broken down to the following steps:

1. **START:** A customer arrives at the onboarding queue once they've completed the prerequisite ADO ticket. The on-call engineer then works with customer to address issues with ticket. 
2. One or more ATS engineers communicate with customer to understand scenarios and to generally educates the customer on ATS solutions. This type of engagement occurs at least once while the customer is undergoing the onboarding process.
3. **END:** a customer has been fully served when their tenant has been onboarded, and they hae received an email notification of onboarding completion.

For details and assumptions on the modeling approach for the Tenant Onboarding sub-process, see [Modeling Tenant Onboarding Using M/M/c queue](#modeling-tenant-onboarding-using-mmc-queue)

## Collecting Data and Defining the Model

### Collecting Data

To understand the ATS onboarding flow, several interviews with ATS engineers we carried out. As part of the interviews, estimate times for processing each step were also recorded. Given that this is a learning project, carried out during Hackathon week, it should be noted that the several limitations in the data acquisition approach and process definitions used in this simulation would be fully addressed in a full experiment.  

### Defining the Model

<img style="text-aling:center; padding: 10px 20px;" src="images/onboarding.png" alt="Onboarding flow">

As previously mentioned, the ATS onboarding flow can be broken down to two core components: (1) Stating Interest, and (2) Tenant Onboarding. Stating Interest is the process by which a customer arrives at a server that ensures the customer arrives at the necessary knowledge state to undergo tenant onboarding, while Tenant Onboarding is the process by which ATS clusters are deployed on behalf of the customer. The description of the modeling approaches used for each component follow.

#### General Simulation Description

The simulation beings in an "empty-and-idle state;" i.e., no customers are present and the server is idle. At time 0, we begin waiting for the arrival of the first customer, which occurs after the first interarrival time $A_1$, rather than at time 0. We wish to simulate this system until a fixed number ($n$) of customers have completed their delays in queue, and they have been served -- i.e., the simulation end when the $n$th customer enter Service.

To measure the performance of the system we will look at estimates of the following quantities:

1. The **expected average delay in queue** of the $n$ customers completing their delays during the simulation -- denoted by $d(n)$. $d(n)$ is the average of a large, finite, number of $n$-customer delays. From a single run of the simulation resulting in customer delays $D_1, D_2, \dots, D_n$ we have an estimator of $$\hat{d}(n)=\frac{\sum_{i=1}^n D_i}{n}$$
2. The **expected average number of customers in the queue** that are not being served -- denoted by $q(n)$. Then, given the number of customers in queue at time $t$ is $t\ge0$, $$\hat{q}(n)=\sum_{i=0}^{\infty} iP_i$$ is the weighted average of the possible values of $i$ for the queue length $Q(t)$. If we let $T_i$ be the total time during the simulation that the queue is of length $i$, then $T(n) = T_0+T_1+\cdots$ and $P_i=T_i/T(n)$ we have  $$\hat{q}(n)=\frac{\sum_{i=0}^{\infty} iT_i}{T(n)}$$,
3. The **expected utilization of the server** is the expected proportion of time during the simulation (from time 0 to time $T(n)$) that the server is busy serving a customer -- denoted by $u(n)$. From a single simulation we have for a 'busy function' $B(t)$ where $B(t)=1$ if the server is busy at time $t$ and $B(t)=0$ if server is idle at time $t$ we $$\hat{u}(n) =\frac{\int_0^{T(n)}B(t) ~dt}{T(n)}$$

#### Modeling Stating Interest System Using GI/G/s Queue

<img style="text-aling:center; padding: 10px 20px;" src="images/GIGs.png" alt="Gi/G/s Model">

##### Problem Statement

A customer can inquire about ATS' solutions at any point in time -- shown in the diagram as arriving at the 'Get-Help Desk.' We assume that the interarrival times of customers are IID exponential random variables with mean of $<TBD>$. Each server, i.e., an on-call engineer (gen. 2 max), will have a separate queue. An arriving customer joins the queue for the first server to engage. A server takes a mean service time of $<TBD>$ to serve a customer. 

There are two types of random variables in this model: interarrival times, and service times.

| Stream | Purpose |
| ----- | ----- |
| 1 | Interarrival times |
| 2 | Service times |

A common way of simulating distributed queues in which a customer can switch between servers, known as jockeying, is done through a [multiteller bank with jockeying](http://tdesell.cs.und.edu/lectures/cs445-03-modeling_complex_systems.pdf) simulation. In our case, instead of a customer switching tellers to one with a shorter queue a customer switches servers when the on-call rotation of the lead terminates.

##### Event Descriptions

| Event description | Event type | Notes |
|----- | ----- | ----- |
| Arrival of a customer to the 'Get Help Desk' | 1 | -- |
| Exchanges of information | 2 | There may be multiple exchanges that affect the system's state. For simplicity, we only consider a single transition from 'not having enough knowledge' to 'having sufficient knowledge' to onboard to ATS. |
| End of sup-process simulation | 3 | 


#### Modeling Tenant Onboarding Using M/M/c Queue

<img style="text-aling:center; padding: 10px 20px; width:100%" src="images/MMc.png" alt="M/M/c Model">

##### Problem Statement

The Tenant Onboarding sub-process is essentially a linear progression that begins when a customer arrives at the queue, i.e., the customer submitted an ADO work item, and terminates when the customer's tenant is onboarded. A customer who arrives and finds the server idle enters service immediately with a service time that is independent of the interarrival times. A customer who arrives and finds the server busy joins the end of a single queue. Upon completing the service for a customer, the server chooses a customer from the queue (if any) in a first-in, first-out (FIFO) manner. 

[Single Server Queueing System](https://www.cs.wm.edu/~esmirni/Teaching/cs526/section1.2.pdf) simulation is a common way of modeling systems such as the Tenant Onboarding sup-process.

##### Event Descriptions

| Event description | Event type |
|----- | ----- |
| Arrival of a customer to the system | 1 |
| Departure of a customer from the system after completing the service | 2 | 

## General Simulation Assumptions 

- Interarrival times $A_1, A_2, \dots$ are independent and identically distributed (IID) random variables -- that is, they all have the same probability distribution)
- All servers are identical
- All customers are identical
- A customer who arrives and finds the server idle enters service immediately, and the service times $S_1, S_2, \dots$ of successive customers are IID.
- A  customer who arrives and finds the server busy joins the end of a single queue.
- One server can only serve one customer at a time
- Upon completing service for a customer, the server chooses a customer from the queue in a first-in, first-out manner.
- Customers experience more pain *while* waiting to be served than while *being* served. Pain(expected average waiting time in system) > Pain (expected average delay in queue)
- In order to use discrete events, as opposed to continuous events, we must consider time as discrete units. A unit of time can be anywhere between 0 and infinity, so long as the counting units are integers.
- Any engineer that can field questions during Stating Interest stage and/or Tenant Onboarding stage, is assumed to have the knowledge of the group. For example, a meeting can include multiple ATS engineers, but the outcome of 'attaining the required knowledge to submit an ADO ticket' is the same independent of the combinatorics. 

## Discrete Event Simulation using SimPy

### Stating Interest

Following is a breakdown of the steps a customer might take as part of completing the Stating Interest sub-process:

1. Arrive at ATS and wait to be engaged.
2. Ask questions and request resources.
3. Wait to have question answered and requests met.
4. Collect required information, or complete learning requirements
5. Choose to schedule a meeting
    - if they schedule a meeting, then they meet
    - if they don't schedule a meeting, they skip to the last step.

6. Complete and submit ADO ticket.

#### Assumptions

In this sequence, ADO engineers are a **resource** that ATS makes available to its customers, and they help customers through the **process** of getting to the place where they can submit an ADO ticket. 

1. Based on observations, either an on-call dev of a more senior engineer fields questions from a customer that is interested in on-boarding to ATS. Hence, `help_desk_eng` can be up to 5.
2. A customer can arrive at the 'Get-Help Desk' at any time of the day, but ATS only fields questions from 8am to 5pm. As such, an arrival that happen at 5pm on a Friday will not be engaged until 8am on the following Monday (63 hrs total). Thus $engage~customer \in (10~mins, (64*60)~mins)$.
3. `on_call_eng` is the number of ADO engineers that can confirm the necessary information has been included in the ADO ticket prior to exiting service. This is generally done by the on-call engineers (2 max)
4. If a meeting is schedule, either one or both of the on-call engineers attend, or a senior+ engineer can run the meeting. Thus, `meeting_eng` can be 5 engineers max.  
5. For simplicity purposes, we assume that the customer takes at least a full workday to collect the information and attain the necessary knowledge to file onboarding ticket. The customer is assumed to be eager, thus it it will not take more than a full week for the customer to have completed the 8 hours worth of work required.
6. If a meeting is schedule, the meeting is assumed to be one-time, hour-long, event that is scheduled at a minimum of 24hrs in advance. For example, if a meeting is schedule on Monday at 9am for the earliest available slot, the meeting would conclude at 10am on Tuesday (total of 25hrs).

| Process in `help_desk` | Requests in `go_to_stating_interest`
| ---------- | ---------- |
| `get_help` | Request an `get_help_eng`|
| `collect_complete` | Request an `on_call_eng` |
| `schedule_meeting` | Request a `meeting_eng` | 

The `get_help_eng` is a shared resource shared by many customers arriving at ATS.

## Production Runs

## Analysis

## Recommended Actions and Next Steps



# CATCHALL

Di = delay in queue of ith customer
 Wi = Di + Si = waiting time in system of ith customer
 Q(t) = number of customers in queue at time t
 L(t) = number of customers in system at time t [Q(t) plus number of customers
 being served at time t]