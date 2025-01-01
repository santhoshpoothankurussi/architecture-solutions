# architecture-solutions
I would like to share solutions to some of the common problems faced while scaling architecture for large enterprise systems.

This content is available on my website and in leanpub.com

Table of contents:-
    1. **Availability**
    2. **Integrability**
    3. **Modifiability**
    4. **Observability**
    5. **Performance**
    6. **Security**


# 1. Availability
    Availability is defined as the degree to which a system is operational and accessible when required for use.

    Typically measured in terms of percentage. eg: 99.999% uptime.
    Availability = successful request / total request.

    # How to maintain system availability

    a. Identify single point of failure
    b. Build mechanisms to detect failures
        Timeouts
        Health check
    c. Introducte appropriate redundancy. Mitigate failure with available redundancy. 

    Types of Redundancy:
        Duplicate code
        Duplicate executable
        Duplicate Data
        Duplicate hardware
        Duplicate request

    Cause of failued requests:
    Service A is requesting service from Service B. A request fails because either:
        Service B is overloaded or
        Service B has failed
    
    **Scenario 1: Service B is overloaded**

    Distinguishing between these two cases is difficult. Because Service A has set a timeout. After timeout period, default action is for Service A to retry same request. Typically, retry must be limited to minimum number of times (2 or 3) before deciding service B has failed.
    Using above behaviour we need to determine the cause of failed request.

    Problems with Retries:
        a. Service B now has two/multiple identical requests in queue
        b. Retry spike. Service A generate duplicate request (if it did not receive a response within stipulated time). Service B has many client each of which doing the same thing.
    
    Solving above problems caused with Rerties
        a. Identical request in Queue
            Option 1: Service B is idempotent. Identical request always return same result.
            Option 2: Service A include requestID and Service B keep track of which request it has served.
        b. Retry spikes
            Option 1: Limit request at upstream locations like API gateways or Loadbalancers.
            Option 2: Limit retry from requestor.
        c. For over request, use masking technique to avoid congestion or failure

    

    **Scenario 2: Service B has failed**

    In cloud failure is inevitable. The best solution is to detect failures using heartbeat check or by sensing lack of response for a service.
    Service A should have fallback in case Service B fails.
    Service A must have alternate method to determine values. Showing default recommendation is better than personalized ones.

    Recovery strategies:-
    a. Recovery from failure over of a stateful service
        i) Hot replicas. In this method you keep a secondary mirror instand up to date and make active if primary fails
        ii) Checkpoint. Here you record state on persistent repository and restore when service re-start after failure. Trade off will be between the cost of creating checkpoint and the data lost between last checkpoint and failure.  
    b. Recovery from failure over of a stateless service
        i) Create new instance of service. Better to have pre allocated instance. Here the tradeoff will be the cost of second instance vs the time required to create new instance.
        ii) If there are multiple instance, over allocate.
        iii) Distributed Coordination Service (e.g. Zookeeper, etcd, consul) can be used to maintain state across multiple stateless instances. Distributed Coordination Service is fault tolerant.
    c. Failure of AWS availability zone. Roughly might happen once a year.
        i) First option is to live with it.
        ii) Keep a replica in different zone. For higher transaction systems, data in transit during failure might get lost.

    **Domino effect of nearest neighbour rollover.** 
    Suppose your system is hosted in availability zone A. And you have set up redundancy on the closest availability zone to the zone A.
    Availability zone A fails, and your system fails over to the closest zone.So does every other system hosted in zone A. Closest zone gets overloaded.

    **Key Measures for systems.**
    These measures will vary for each system in an organisation.

    RTO – recovery time objective. How long before system is back in service again. System will be available with data saved previously.
    RPO – recovery point objective. How much data can be lost in the event of a disaster. Data is saved till this point.

    How to divide your systems into tiers based on RTO and RPO?
    Tier 1 (mission critical) – 15 minutes
    Tier 2 (important support ) - 2 hours
    Tier 3 (less important support) - 4 hours
    Tier 4 (everything else) - 24 hours
    
    Other issues:-
    **Scheduling and Priority Inverions.**

    In real time systems, threads are prioritized. 
    Priority inversion causes lower priorities to be run ahead of higher priorities. Or the lower priority items will never get executed.
    Priority inversion is detected by looking at priorities of various threads vs execution of those threads.Priority inversion is prevented by using appropriate scheduling mechanism.
    
# 2. Integrability

# 3. Modifiability

# 4. Observability

# 5. Performance

# 6. Security
     
        
