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
    Is the extent to which you can understand the internal state or condition of a system based on the knowledge of its external outputs.
    
    Two aspects:
    Collecting data - metrics, traces and logs
    Displaying data – alerts, dashboards
    
    Data collection:
    While the system is opertional, information is gathered for three purposes:
        Alerting – Detecting that there is a problem
        Forensics – determining what caused a problem
        Improvement – finding bottlenecks in systems or determining causes of internet traffic.
    
    Implementation:
    Measurements are taken from a running system and its environment and sent to a back end.  
    The back end has a database (usually a time series database). It 
        Generates alerts 
        Generates reports
        Allows drilling down into aggregate information to get more detailed information
        Will have a dashboard to give fast indication of problems.
    
    Data can be either pushed into back end or pulled from back end
    Information gathered in the backend contains:
        Service instance ID (if log)
        VM or container instance ID i(if metric)
        User and trace id 
        May want to add deployment history for instances

    Utilization:
        Utilization is measure of some activity over some period of time. Instantaneous measurement is not meaningful. 
        
        Collected automatically by infrastructure over externally visible activities of VM or container
        CPU – percentage busy (for eg: last 5 mints)
        I/O – amount of traffic

        Collected by back end from infrastructure
        Pushed by an infrastructure plugin that is dependent on back end.
    
    Alerts:
        Back end has set of rules to establish when to send an alert. E.g. alert if CPU utilization is over 80% for 15 minutes.
        Utilization numbers are bursty. The period must be sufficiently long to indicate problem.
        False positives and false negatives are both problems.
        
    Logs:
        A log is an append only data structure written by each software application.
        Data structure include:
            Id of external request (explained later)
            clock sequence number (explained later)
            Service id
            Parameter values
            Queue lengths, disk space, memory usage
            
        Located in a fixed directory within the operating system
        Enumerates events from within application. For eg:
            entry/exit and associated parameters
            Troubleshooting
            DB modifications
            Exceptions 
        Every request gets a unique id when it enters the system. This id is passed with each message.

        Log Deamon:
        A daemon resides on each individual server. It copies logs generated by applications on that server to back end.
        Logs can grow in volume consuming server space and so daemon may also clean off log file periodically once the contents have been sent to the back end.
        
        Sequencing log entries:
        Having logs in sequential order is important for troubleshooting.
        Time stamps are inadequate for this purpose since computer clocks drift and two computers on a network will likely have different times.
        Lamport clock is early attempt to provide sequence. It provides a partial order among events.
        
        Lamport’s Logical Clock was created by Leslie Lamport. It is a procedure to determine the order of events occurring. It provides a basis for the more advanced Vector Clock Algorithm. Due to the absence of a Global Clock in a Distributed Operating System Lamport Logical Clock is needed.

        Lamport clock algorithm:
            i) A process increments its counter before each message sending event);
            ii) When a process sends a message, it includes its counter value with the message after executing step 1
            iii) On receiving a message, the counter of the recipient is updated, if necessary, to the greater of its current counter and the counter in the received message. The counter is then incremented by 1 before the message is considered received.
            
                
        Purpose of Logs:
        Logs are used to determine what happened during the incident.
        Logs can also be used to play back a sequence of events during trouble shooting.
    
    Tracing:
        Tracing provides an end to end picture of a request. The big picture.
            > Utilization helps determine whether there is a problem.
            > Logging provides picture of a single event
            > Tracing provides a sequence of events
            
        Span:
        A trace captures the end to end actions in response to a user request.
        A span is a named, timed operation that represents a piece of the trace. 
        Spans may have child spans
        Displaying spans on a time axis allows you to see parallelism and where time is being spent
        
        Open source options: https://sflanders.net/2019/03/28/an-intro-to-distributed-tracing/

        One of the possible implementations:
            a) Request enters the system from external source – user or external system
            b) Request is given a unique ID that reflects the context
            c) Context description is kept in a data base so that with context ID analyst can know details of the context.
            d) Context id becomes a portion of HTTP header.WWW standards.
            e) The context ID is inserted by the HTTP server accepting the request and propagated by every service as it fans out the request. This is transparent to the requester.
            f) Context can be used to control behavior of a service.
            
        How to Use context to affect behavior or routing.
            a) Application. You might want to know what percentage of your network traffic from search engine, social media and other marketing platforms etc.
            b) Traffic prioritization. Give priority to certain requests to maintain quality of service.
            c) Bottleneck determination. Where is the most time being spent in a collection of transactions?
    
    Design observability:    
        Process:
            1) Define contexts of interest
            2) Save status codes for definitions of context in database
            3) include  back end specific log daemons in build step of deployment
        Design:
            1) Add log entries on entry exit
                Invoking process
                Parameters
                Context
                Queue lengths
            2) Add other log entries
                Exceptions
            3) Maintain flag to turn on and off logging also to save specific log.
                Logs grow in time and will incurr cost.
            

# 5. Performance
    1) Basic request and response handshake with a single server
    Requests for service arrive at the system. They are placed in a queue
    They are served according to some algorithm and a response is generated. 
    The request for service may involve database access as well as computation.
    Database access involves network requests.
    Accesses across a network are slower than computation from memory.
    
    Latency of server: time between request arrival and generation of response
    Throughput: number of requests that can be served in a unit time
    
    Statistics:
    Read 1 MB (1, 000, 000 bytes) sequentially from memory ~250,000 ns 
    Send 1K (1, 000) bytes over 1 Gbps network ~10,000 ns 
    
    Reduce latency for a single server.
    With a single server, latency may be reduced by reducing the time for required computation
    Use a better algorithm.
    Reduce system overhead.
    Use a host with more resources  (faster processor, more memory, more disk)
    Changing some database accesses to memory accesses (caching)
    
    2) Handshake between 2 servers.
    Output of first server is input of second.
    Output of server 1 travels over a network to server 2.
    Latency is time at server 1 + network transport time + time at server 2.
    
    Reduce latency for a two server.
    Latency can be reduced by either reducing the latency of a server or reducing the network transport time.
    Reducing the latency of a server is as with a single server
    Reducing the network transport time can be achieved by
        Using a different protocol
        Using a faster network
        Reducing the volume of information sent.
    
    with multiple servers:
    Multiple servers can be strung together into a pipeline 
    Reducing the latency of a pipeline is achieved by reducing the latency of the servers or the network transport.
    
    How is this related to latency requirement:
    Suppose there is a latency requirement for a set of pipelined servers.
    How does this translate into requirements for the individual servers?
    Each server and network transport is given a budget where the sum of the budgets is less than overall requirement.
    
    3) Horizontal scaling
    Reducing the overall latency for a collection of independent requests can be done by having multiple servers running in parallel.
    Requests arrive at a distribution mechanism. The distribution mechanism sends each request to one of the parallel servers. The assumption is that the servers are equivalent.
    
    Horizontal scaling works best if the servers are stateless. Stateless servers must get their state from somewhere.

    How the state can be managed?
    Stateless servers can get state from Parameters or Repository external to server.
    A database may be shared across servers and may be used to store necessary state.
    Data consistency is a concern and that need to be managed.
    
    Design for performance using POD.
    A pod is a Kubernetes construct for hold containers. 
    Two containers in the same pod will be deployed together
    Communication between two containers in the same pod does not go over the network
    Place containers that communicate frequently in same pod.
    Replicate data to place data closer to retriever. Closer means not over network.
    Replace database access by in memory access.
    Concern - Caching may result in inconsistency between data in cache and data in database.
    Consistency must be managed

# 6. Security
    security by defenition is the protection of computer systems and information from harm, theft, and unauthorized use.

    CIA Defenition:
    Confidentiality – only authorized users can access data and resources.
    Integrity – data is not corrupted or modified.
    Availability – information and resources are available to authorized users.
    
    Basic structure of Security:
    1) Principle of least privilege
        A security architecture should be designed so that each entity is granted the minimum system resources and authorizations that the entity needs to perform its function. An entity is a person or a program. Entities must be authenticated and authorized to perform its function.
        
        Authentication: 
            is the process of verifying that you are who you say you are
            
            factor:
            What you know (password, answers to questions)
            What you have (smartphone, key card)
            What you are (biometrics)
        
        Authorization:
            Authorization can be either as an individual or as a member of a group.
            Individuals are assigned privileges. More commonly, individuals are assigned roles and roles are assigned privileges. Role Based Access Control (RBAC) is the common norm in most organizations.
            
            factor:
            An authorization server provides a client a token that encodes privileges
            Tokens usually expire after a specified interval.
            Examples: OAuth, Kerberos
            Resource manager – possibly API – verifies tokens prior to allowing access.
        
    2) Data Protection
        Data can be
            At rest – stored on a permanent storage medium.
            In transit – being transmitted over the internet.
            In use – in memory of a process.
        
        Data should be protected from unauthorized reading or modification.
        
        Hashing:
        “Hello” >> SHA-3 >> 256 bit number
        A hash is a one way transformation based on a public algorithm with no  key. Not possible (very difficult) to decrypt.

        Hashing is Used to verify integrity of data
            Passwords: save hash of password but not password. When user  enters password, compare to hash to verify.
            Downloads: publish hash of software available for download.  Compare hash of downloaded software. Verifies that software has  not been modified.
        
        Encryption:
        Method of encoding data so that it is not readable without key.
        
        Two forms of encryption
        i) Symmetric – the same key is used for encryption and decryption. Suitable for data at rest.
            NIST (National Institute of Science and Technology) approved algorithm is AES with key lengths of >128 bits
        
        ii) Asymmetric- one key is used for encryption and a separate key is used for decryption.
            Also known as public/private key encryption
            Messages encrypted with public key can be decrypted by private key (and vice versa)
            NIST approved algorithms: DSA, RSA, ECDSA >1024 bits
            
        Symmetric encryption is ~4000x faster than asymmetric encryption.
        TLS uses asymmetric encryption to verify identity and symmetric encryption to transfer data
        
        Digital Signature:
            A digital signature is a means for sending an open signed letter.
            Anyone can read it but it is guaranteed to come from a particular party.
            You wish to send “text”.
            Hash “text” to get a hash value
            Encrypt the hash value with your private key
            The message consists of	“text”+encrypted hash value.
            The message cannot be altered. 
            The hash value guarantees the integrity of the message, and the senders private key encrypts the hash value.
            Changing the message would require knowing the sender's private key.
            
            Why encrypt only Hash value?
            For Efficiency. The hash value is much shorter than the full message. The time to decrypt depends on the length of the message. Shorter is faster.
            For Compatibility. Different signing schemes exist and just encrypting the hash provides compatibility with multiple schemes.
            
        TLS:
        TLS (Transport Layer Security) is designed to provide communications security over a computer network and to avoid eavesdropping and tampering. 
        TLS depends on
            Symmetric encryption
            Asymmetric encryption
            Certificates
        
    3) Protecting Data in use
        Typically transactional data will be stored 
            Within an application
            Within a cookie
            Within cache
            Within log
        Must be decrypted
        
        Protection:
        Require authorization to access data
        Restrict sensitive data in use. Do not store sensitive data in cookies or logs
        Purge caches Periodically or on specific events. eg. end of a session

    4) Protecting data at rest
        i) Use symmetric encryption
        ii) Backups
        iii) Maintaining current backups in distinct datacenters with distinct access controls will guard against a ransomware attack.

    5) Protecting Resources
        1) Perimeter security
            Restricting access
            Authentication
            Authorization
            Assumption is that if user can get through perimeter, they are not malicious
        2) Zero trust security
            Perimeter +  authorization tokens. Do not trust even users who get through perimeter authorization checks.
        3) Mitigate a DDoS attack
            Limit attack surface
            Ensure adequate network capacity
            Ensure adequate server capacity
            Maintain current back ups with different access privileges.
        4) Limiting Access
            Restrict number of access points. eg. require access through a gateway
            Restrict traffic. eg. Firewalls can restrict access based on port numbers.
        5) Limit attack surface
            The attack surface is the total number of all possible entry points for unauthorized access into any system. 
            Techniques:
            Disabling access to most ports. Firewalls only forward messages to specific ports.
            Segmenting networks. Use principle of least privilege to control access
        6)SBOM
            Software Bill of Materials can be created when the executable is being built.
            An SBOM enumerates all the dependencies included in your service. 
            This adds capabilty to scan for vulnerabilities once your service is in production.
            A vulnerability in a dependency may have been discovered after you deployed the dependency.
        7) Secure design pattern
            Privilege Separation:
            Each component possesses minimum privileges required for it to function
            
            Distrustful Composition:
            Components don’t fully trust each other
            Canonicalize, sanitize, normalize, and validate inputs to limit potential attacks
            Sanitize outputs to prevent information and capability leaks
            Require access tokens

            Use memory safe languages. since memory safety bugs are often security issues, memory safe languages are more secure than languages that are not memory safe. Memory safe languages include Rust, Go, C#, Java, Swift, Python, and JavaScript.
            
            https://www.cisa.gov/sites/default/files/2023-04/principles_approaches_for_security-by-design-default_508_0.pdf
                    
        