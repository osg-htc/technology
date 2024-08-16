COManage Recipes
==============================

A collection of step-by-step instructions for common actions for use by administrators of the OSG COManage.


Steps for common COManage Actions
----
This section contains some common actions administrators perform on the OSG COManage, and how to do so.

### Provisioning a CO Group in COManage
In order for a CO Group from COManage to show up in LDAP (and thus be made available for reference on Hosts),
 it must first be provisioned.

Follow these steps to provision a CO Group into LDAP:

1. #### Create CO Group in COManage and add members (or use existing CO Group)

    Skip the first bullet point if using an existing Group.

    -   Navigate to the `All Groups` page in COManage and click the `+ Add Group` button near the top-right.
        Give the group a name then click `ADD`, which will bring you to the Edit page for the new group
    -   Click on `MEMBERS`, then type in the name or identifier for a user you want to give membership to, 
        then select the user from the drop-down and click the `ADD` button. 
        Repeat as necessary for each group member.
        As the creator of the Group, you will already have both Membership in, and Ownership over, the new group.

1. #### Find lowest unclaimed OSG GID in the range of non-user GIDS

    Each group needs a unique OSG group id number or `OSG GID`, assigned from the non-user range starting at `200000`.

    -   Run the following command on a host with `ldapsearch` capability (like ap40) to find
     the highest / most recently assigned `OSG GID`.

            :::console
            sudo ldapsearch -H ldaps://ldap.cilogon.org -D uid=readonly_user,ou=system,o=OSG,o=CO,dc=cilogon,dc=org\
             -w $(sudo awk '/ldap_default_authtok/ {print $3}' /etc/sssd/conf.d/0060_domain_CILOGON.ORG.conf)\
              -b ou=groups,o=OSG,o=CO,dc=cilogon,dc=org -s one '(cn=*)' | grep "gidNumber" | sort | tail

1. #### Set OSG GID and OSG Group Name Identifiers

    Navigate back to the `PROPERTIES` tab of Edit page for the group you are trying to provision,
     then click the `+ Add Identifier` button.

    -   Add an Identifier of type `OSG GID` with a value one greater than the highest one assigned so far
     (found in the last step).
    -   Add an Identifier of type `OSG Group Name` with the group's name as it should appear in LDAP.

1.  #### Create Unix Cluster Group

    Each COManage Group needs a Unix Cluster Group in order to be provisioned. 
    
    -   On COManage, navigate to `Configuration` -> `Clusters` -> `Configure` -> `Manage Unix Cluster Groups` 
    -> `+ Add Unix Cluster Group`
    -   Select the name of the Group you are trying to provision from the drop-down menu, then click `ADD`

1.  #### Provision group

    -   In the `PROVISIONED SERVICES` tab of the Edit page for the Group,
     click the `⚙ Provision` button, then on `Provision`.
    
    If all prior steps have been completed, you should get a message that the Group was successfully provisioned.