# Resource Manager Module

- Before the use of a resource, a process has to invoke a **resource manager** to acquire the required resource.
- When a process releases a resource, the state of other processes waiting for the resource must be set to READY.
- There are 2 functions in the module here: one to acquire the resource, and the other to release the resource.