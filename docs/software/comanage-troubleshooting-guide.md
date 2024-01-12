COManage Troubleshooting Guide
==============================

A resource for COManage administrators to find solutions to commonly encountered problems.


COManage Troubleshooting Items
----
This section contains some of the issues you may encounter when interacting with the OSG COManage.


### Valid Tiger User is unauthorized with Dex credentials

Under certain circumstances Dex may not be able to match a user to their COManage identity,
 possibly due to incorrect and/or missing organization identities/identifiers.

#### Symptoms

 - Tiger-Dex credentials missing some or all of a users group permissions.
 - User unable to connect to the Tiger cluster.
 - Error message from Tiger involving an unauthorized user.
    - Example error message: 

    >kubectl get pods -n osg-dev

    >Error from server (Forbidden): pods is forbidden: User "user@email.edu" cannot list resource "pods" in API group "" in the namespace "osg-dev""
 
 - Empty Dex response

#### Next actions

1. Ensure that the user's LDAP record contains the attribute `uid`,
 which shares the same value as the COManage identifier used as a source for the LDAP provisioner target.
    - LDAP attributes and their COManage sources can be found [here](https://registry.cilogon.org/registry/ldap_provisioner/co_ldap_provisioner_targets/edit/6)
1. Have the user log in
1. Have the user fetch a new Dex token  

#### Explanation of SAML -> CILogon OIDC sub claim -> LDAP matching used in Dex.

How the OAuth2 server does the following:

1. Receives appropriate SAML attributes from the campus Identity provider.
1. Uses those attributes to find the user in the CILogon user database,
 from which the CILogon OIDC sub claim is obtained.
1. Performs an LDAP search using a filter of `(uid=<CILogon OIDC sub claim value>)`.
1. Retrieves configured LDAP attributes from the record it has found,
 and sends those values out as claims with names as configured.
