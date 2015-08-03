# Authorization


While authentication is about establishing and verifying user identity, authorization is about permissions. Is my user allowed to perform the operation it is invoking? None of the standards-based Internet authorization protocols discussed so far deals with authorization. The server and application know the permissions for each user and do not need to share this information over a communication protocol. This is why authorization is the domain of the server and application.


JAX-RS relies on the servlet and Java EE specifications to define how authorization works. Authorization is performed in Java EE by associating one or more roles with a given user and then assigning permissions based on that role. While an example of a user might be “Bill” or “Monica,” roles are used to identify a group of users—for instance, “adminstrator,” “manager,” or “employee.” You do not assign access control on a per-user basis, but rather on a per-role basis.
