# Technical Requirements

> ⚠️ This document is not a requirements specification.  
> It contains a mix of ideas, research topics, and design considerations.  
> It needs to be decomposed into requirements, architecture, and research artifacts.

## Foundational
* Solidify our definitions of main terminology
* If we keep "groups" as first class citizen and not implementation detail, describe their mechanics
* Provide guidelines and best practices for creating post-platforms/w3-adapters, to encourage more interoperable ecosystem
    - the usage of eNames
    - reusing and creating ontologies
    - handling authorization
    - aggregation and caching
* Make all above documentation AI friendly. This way it can answer questions and check consistency with our codebase
* Describe methods for managing pseudonymous profiles
* Explain how anonymous transactions will work. And in general explain PET in the ecosystem.
* Explain the limits of security and privacy in currently deployed eVaults
* Capacity planning for eVaults and infrastructure services 

## Functional

### eVault
* Carefully define and generalize the model of controlling eName (forms of binding)
* Support deduplication of eNames and eVaults, perhaps with eMover
* Evaluate the meta-envelopes model (flat) for complex topologies of data
* Strictly adhere to declared ontology within a single meta-envelope
* Explore dynamic GraphQL queries/mutation from schemas
* Research ways to map (without loss of performance) knowledge graph inference and formalisms such as SPARQL to meta-envelopes 
* Provide rigorous authorization layer
    - Define levels of access to resource for users/groups/platforms. Roughly like [wac](https://solidproject.org/TR/wac)
    - Would be nice to be able to inherit ACLs, but we don't have explicit hierarchies
    - Consider opt-in vs. opt-out systems
    - Can we support attribute based or fully semantic ACLs?
    - If not, we should define useful categories: e.g., reputation driven authorization, authenticated users, certified platforms
* Improve platform's authentication with eVaults [details](platauth.md)
* Involve linked data specialists to aid with ontology mapping (for adapters) and alignment in general
* Explain how two mostly similar ontologies can be reconciled at the schema and eVault level
* Add basic semantic transformation engine to eVault, allowing to have Big-Bang sweeps of semantic updates, e.g., after merging two popular ontologies.
* Consider encryption of data at rest: with external keys, keys on eVault, keys in hardware
* Initiate research for "rootless" provisioners -- where eVault provider has no backdoor to eVault envelopes
* Rethink "audit" logs
    - recorded data (platform, user, timestamp, hash of payload)
    - how is non repudiation guaranteed? 
    - how can we guarantee log integrity?
    - how we are actually going to provide the logs to auditors?
    - can audits preserve privacy?  

### Infrastructural platforms
* eVault provisioners
    - seemlessly support single eVault on VM (1st model) and shared tenancy (current model)
* registries -- resolvers of eNames to eVault protocol endpoints
    - ideally, the data is stored on platform's eVault and can be 
* ontology collections -- resolvers of eNames to schemas
* awareness endpoints -- fan-out for awareness messages
    - awareness must respect ACLs and any other authorization mechanisms we may have
* caching layer (ora CDN-like)
    - must respect ACLs

Note: it is likely that big eVault providers are going to serve all above as well

Note2: When possible, do not create special endpoints and API, just use envelopes and platform eVaults.
For example, the registry can store eName->uri mappings as envelopes on its eVault; awareness platform can record subscriptions information on its eVault, etc.

## Resilience
* We must have at least two instances of everything

* We should have different implementations for everything.

* We should have a discovery mechanism to obtain available infra. platforms.
The multitude of them can be used for active load balancing or for failing over when primary choice is down.

## Development
* Support proper staging
* A/B testing
* Provide testing sandboxes
    - In containers
    - With accurate mocks for unit testing
