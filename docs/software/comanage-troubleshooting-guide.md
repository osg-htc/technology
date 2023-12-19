COmanage Troubleshooting Guide
====
Introduction
----
The purpose of this guide is to be a resource for COmanage administrators to find solutions to commonly encountered problems.


COManage Troubleshooting Items
----
This section contains some of the issues you may encounter when interacting with the OSG COmanage.

Problem list:

 - [Tiger-Dex Unable to match User to COmanage CO Person identity](#tiger-dex-unable-to-match-user-to-comanage-co-person-identity)

###Tiger-Dex Unable to match User to COmanage CO Person identity

Under certain circumstances Dex may not be able to match a user to their COmanage identity,
 possibly due to incorrect and/or missing organization identities/identifiers.

####Symptoms

Symptoms include:

 - Tiger-Dex credentials not including some or all of a users group permissions.
 - Not being able to connect to the Tiger cluster.

####Next actions

- Ensure all of the user's org identities / identifiers required by LDAP exist and match across COmanage and LDAP.
    - Found at `https://registry.cilogon.org/registry/ldap_provisioner/co_ldap_provisioner_targets/edit/6`
- Setting the missing/mismatched value manually did not seem to work, 
but having the user log into COmanage with the relevant authorizer did.

####Explanation of SAML -> CILogon OIDC sub claim -> LDAP matching used in Dex.

>> P.S. How does OIDC machinery go about finding someone in LDAP when mapping LDAP attributes to OIDC claims?
>
> The OAuth2 server (which handles OIDC flows) receives appropriate SAML attributes from the upstream campus IdP.
> It then uses those to find the user in the CILogon user database and obtain the CILogon OIDC sub claim.
>
> It then takes the CILogon OIDC sub claim and does an LDAP search using (by default) the LDAP search filter:
>
> `(uid=<CILogon OIDC sub claim value>)`
>
> It retrieves configured LDAP attributes from the record it has found,
> and then sends those values out as claims with names as configured.
>
>> What determines that the uid attribute is essential for this process?
>
> Technical debt.
>
> "Real soon now" [Scott Koranda] will be upgrading COmanage Registry to version 4.1.2,
> and with that upgrade there will be a new version of the Registry plugin that allows you to configure OIDC clients.
>
> With that new version of the plugin you will be able to select an LDAP attribute other than uid to create,
> for example, an LDAP search filter like :
>
> `(voPersonSoRID=<CILogon OIDC sub claim value>)`
>
> or
>
> `(voPersonExternalID;app-cilogon=<CILogon OIDC sub claim value>)`
>
> That last example uses LDAP attribute options. See
>
> ```
> https://spaces.at.internet2.edu/display/COmanage/LDAP+Provisioning+Plugin#LDAPProvisioningPlugin-LDAPAttributeOptions
> ```
>
> and
>
> `https://github.com/voperson/voperson/blob/main/voPerson.md#voperson-attribute-options`
>
> Brian Lin and [Scott Koranda] have talked about the OSG CO moving eventually to using LDAP attribute options.
> It will give you more flexibility, 
> but you do have to be prepared for a few (relatively minor) changes in the details of the LDAP records.
