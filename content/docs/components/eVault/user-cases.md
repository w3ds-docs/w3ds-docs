## UC-1: Create eVault

### Description
Creation of a new eVault associated with an provided eName.  
The eVault is initialized in a minimal state and registered in the system.
If an entity creates an eVault for itself, a public key is provided immediately to ensure further control over the eVault.
If an eVault is created without a key (e.g., for an organization or a child), a trusted party is assigned as its controller, and their authority is verified.

### Actors
- Request initiator
- eVault
- Registry

### Preconditions
- An eName provided in the request

### Main Scenario
1. A request to create an eVault for a given eName is received.
2. The system validates the request.
3. A new eVault is created.
4. The eVault is assigned a unique eID.
5. The eVault is linked to the provided eName.
7. The eVault is initialized in a minimal state:
   7.1 If public key is provided it is stored.
   7.2 If trusted party is provided it's eName and authority information are stored.
9. The eVault publishes information about itself to the Registry.

### Alternative Scenarios
- A1: Invalid request  
  - If the request is invalid or incomplete, the system rejects it and returns an error.

- A2: eVault already exists  
  - If an eVault is already associated with the given eName, the system may:
    - return the existing eVault, or  
    - reject the request (depending on system rules).

### Notes
- It is not yet defined who is authorized to initiate eVault creation.
- It is not yet defined how spam or excessive creation requests are prevented.
- It is not yet defined who can write to the eVault before keys are bound.
- The mechanism for publishing eVault information to the Registry requires further specification.
