# Technical Requirements

## Foundational
* Solidify our definitions of main terminology
* If we keep "groups" as first class citizen and not implementation detail, describe their mechanics
* Provide guidelines and best practices for creating post-platforms/w3-adapters, to encourage more interoperable ecosystem
    - the usage of enames
    - reusing and creating ontologies
    - handling authorization
    - aggregation and caching
* Make all above documentation AI friendly. This way it can answer questions and check consistency with our codebase.
* Describe methods for managing pseudonymous profiles
* Explain how anonymous transactions will work. And in general explain PET in the ecosystem.
* Explain the limits of security and privacy in currently deployed evaults
* Capacity planning for evaults and infrastructure services 

## Functional

### evault
* Evaluate the meta-envelopes model (flat) for complex topologies of data
* Strictly adhere to declared ontology within a single meta-envelope
* Provide rigorous authorization layer
    - Define levels of access to resource for users/groups/platforms. Roughly like [wac](https://solidproject.org/TR/wac)
    - Would be nice to be able to inherit ACls, but we don't have explicit hierarchies
    - Consider opt-in vs. opt-out systems
    - Can we support attribute based or fully semantic ACLs?
    - If not, we should define useful categories: e.g., reputation driven authorization
* Involve linked data specialists to aid with ontology mapping (for adapters) and alignment in general
* Add basic semantic transformation engine to evault, allowing to have Big-Bang sweeps of semantic updates, e.g., after merging two popular ontologies.
* Consider encryption of data at rest, even without solving the key management problem
* Initiate research for "rootless" provisioners -- where evault provider has no backdoor to evault envelopes  

### Infrastructural platforms
* evault provisioner
* registries -- resolvers of enames to evault protocol endpoints
    - ideally, the data is stored on platform's evault and can be 
* ontology collections -- resolvers of enames to schemas
* awareness endpoints -- fan-out and caching for awareness messages
    - awareness must respect ACLs and any other authorization mechanisms we may have

## Redundancy
* When possible, do not create special endpoints and API, just use envelopes and platform evaults.
For example, the registry can store ename->uri mappings as envelopes on its evault; awareness platform can record subscriptions information on its evault, etc.

* We want at least two instances of everything, ideally different implementations.

* We should have a discovery mechanism to obtain available infra. platforms.
The multitude of them can be used for active load balancing or for failing over when primary choice is down.

