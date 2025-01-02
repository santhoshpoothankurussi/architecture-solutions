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
    Integrability (or integration capability) refers to the ability of a software system or component to effectively connect, communicate, and work together with other systems or components. One portion of integrability is interoperability, which is the ability to usefully exchange information between two software components.

    a. A component must know how to route a message to other components. 
    b. Both components must agree on the format of the data to be exchanged.
    c. The components must correctly interpret the data being exchanged.
    
    Routing options:-
        Manual- user informs system of existence and location of a particular component. System informs other components.
        Discovery – component is entered into a table with component ID and location. Table is used to look up component by ID
        Plug and play – when component is plugged into a network, it locates a recipient using a discovery service. The recipient may be 
        a middleware component or an intended component with which to interoperate
        If the new component does not know location of discovery service, it can broadcast “I am here” message and discovery service should have a listener for such messages.

    Two components can communicate through a third middleware component by publishing or subscribing to a topic. Middleware upon initialization creates a list of topics. Each registers with the middleware for specific topics. The middleware forwards messages from one component to the other.
    
    How to manage interface symantics:-
    Both parties wishing to interoperate must agree on symantics. For e.g. What is a residential address format? it differs in various portions of the world.
    A third party may define agreed upon interpretation. Domain specific organization 
    May require translation from one representation to another.

    Managing multi lingual interface:
    If the two components that are interoperating are writing in different languages, managing the interface may become difficult.
    This is true even when the two components are distributed.
    
    Using Protocol Buffer:-
    Protocol buffer allows multi language communication using a schema that defines data types
    A protocol buffer specification is used to specify an interface
    Language specific compilers used for each side of an interface
    Allows different languages to communicate across a message based interface
    
    For eg: Service A written in Python calls Service B written in Go
    Interface specification written as .proto file
    Python  protocol buffer compiler produces python procedure interface for Service A
    Go protocol buffer compiler produces procedure interface for Service B
    Service A code calls Python procedure interface which sends data received by Service B procedure (written in Go)
    
    Using Messagebus:-
    A message bus is a common pattern used to facilitate interoperability.
    Components can publish or subscribe to listed topics and send message via these topics.
    The message bus can translate message from one format to another also make semantic translations
    Kafka is a commonly used message bus.

# 3. Modifiability
    Modifiability is the ability to make a change to the behavior of a system.
    Behviour changes can be achieved with one one of the below methods:
        a. Compile time through code changes
        b. Build time through incorporation of different dependencies
        c. Initialization time through configuration parameters or discovery
        d. Run time through either configuration parameters or particular types of input

    a) Achieving Compile time changes
        Coupling: An overlap in responsibilities among modules. Strong dependency of one module over another. Modifications localized to one module are easier. Makes it more likely that changes are localized to one module.
        
        Cohesion: The relation between the responsibilities in one module.
        Strive for high cohesion and low coupling.

        How to determine high coupling:-
            i) Using code. Design Structure Matrix (DSM)
            ii) Once system is Live in production. Use architecture debt hotspots
        
        ### Design Structure Matrix (DSM)

        A Design Structure Matrix (DSM) is a visual representation of dependencies among modules in an architecture
        Represented by a square matrix with modules along both axes.
        Place mark in the matrix if there is a dependency between module in row and module in column.
        https://dsmsuite.github.io/dsm_overview

        DSM can be used to determine
            i) A cyclic relation between two modules
            ii) A relation against the expected layering.
            iii) A relationship that skips a layer.
            iv) An unused module without incoming relationships.
            v) A module with many in and outgoing relationships 
        
        ### Architecture debt hotspots

        Begin with a DSM
        Add marks to cells from modules that change simultaneously. This information is available from a version control system.
        Focus on bug fixes, not new functionality (available from issue control system)
        Cells in upper diagonal with marks represent modules with architecture debt
        
        Reducing Coupling by Refactoring. Move overlapped code to a third module.

    b) Achieving Compile time changes
        In a distributed system, the location of a component can change and must be discovered.
        Mechanisms for discovery include
            Domain Name System
            Service Mesh
            Load balancer
        
        During build replace one component over another which offer similar functionality. 

     c) Achieving Installation time changes
        
        Option1:- Configuration parameters
        When system is invoked it acquires configuration parameters using following methods.
            From a special file in a known location
            From OS environment variables
            From configuration database
            From a special tool
            
        Values of configuration parameters will affect the behavior of a system.
        For eg: Configuration parameters allow a system to operate in different environments. 
        Make database URL a configuration parameter. This allows a different database to be used during development than during production.
    
    d) Achieving Runtime changes
        Possible by changing 
        i) Location of instance and number of instances
        ii) Sequence of events can be changed by orchestrator or as a result of failure
       
    ### How to apply these techniques while designing microservices.

    To define:-
    “”Microservices architecture consists of collections of light-weight, loosely-coupled services. Each service implements a single business capability. 

    While designing microservice
        Start with each microservice implementing a single business capability
        Identify microservices with high coupling using DSM.
        Reduce coupling either through refactoring or introduction of intermediaries
        Each microservice becomes a container
        Place containers with high intercommunication needs into a single pod.


# 4. Observability

# 5. Performance

# 6. Security
     
        
