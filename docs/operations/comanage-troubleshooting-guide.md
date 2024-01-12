COManage Troubleshooting Guide
==============================

A resource for COManage administrators to find solutions to commonly encountered problems.


COManage Troubleshooting Items
----
This section contains some of the issues you may encounter when interacting with the OSG COManage.

### The COManage petition is stuck in the "confirmed" state

This may happen if there are issues when confirming the user's email address.
We have seen this occur if a user clicks the confirmation link then closes the tab too quickly.

1.  Under the `People` drop-down on the left,  click on `My Population`

1.  Browse to the `CO Person` record.

1.  Scroll down to the `Role Attributes`, click the gear icon, select `Edit`, and set the status for the `Role` to `Active`.

1.  Verify that the overall status of the `CO Person` record is `Active`.  If not, change it to `Active` as well.

1.  Click on `Autogenerate Identifiers` on the right, so that the necessary identifiers are created.

1.  Now that the necessary identifiers exist for the `CO Person` record, the LDAP DN can be computed and the record
    provisioned in LDAP. To make sure, click on `Provisioned Services` and then  `Provision`.


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
 
 - Empty groups in the Dex response after logging in with SSO

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
