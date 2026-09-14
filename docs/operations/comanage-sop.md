COManage Operations
====================

OSG is using a new identity management system called COManage.
This system is used for managing contact information for OSPool and PATh Facility users, Topology site contacts, and
OSG/PATh staff.

Contact Registration
--------------------

Contact registrations must be manually approved by a COManage admin.
Follow the instructions below to approve a contact registration.

!!! note
    This page is for COManage Admins who want to approve contact registrations.
    If you are a user who wants to register with COManage,
    go to the [Registering for the OSG COManage](https://osg-htc.org/docs/common/contact-registration) page instead.

1.  Check for contact registration requests:

    -   If you are a COManage sponsor for a given group of registrants, you will receive email notifications when there
        are new registration requests.
        Check for an email from <registry@cilogon.org> saying "Petition for <NAME> changed status from
        Confirmed to Pending Approval" and visit the first link in the body.

    -   Alternatively, you can view all requests pending approval
        [here](https://registry.cilogon.org/registry/co_petitions/index/co:7/sort:CoPetition.created/direction:desc/search.status:PA).
        Click on the registrant's name to view their request.

        !!! note "Topology registrations"
            You can view the list of Topology registrations [here](https://registry.cilogon.org/registry/co_petitions/index/co:7/sort:CoPetition.created/direction:desc/search.status:PA/search.cou:55/op:search).

        !!! note
            Many groups share our COManage instance so make sure that you're only approving registration requests for
            the appropriate group, e.g. site contacts.

1.  If prompted, log in with your institutional credentials.

1.  Review the request:

    1.  Verify that the request is legitimate by asking someone affiliated with the site, collaboration, or the sponsor
        of a project to verify the registrant's affiliation.
        In the case of the OSPool, this should be the OSG Campus Coordinator
        (try searching Freshdesk for the requester's email address and look for a private note with approval).

    1.  Verify that the registrant has submitted their request using the correct form,
        e.g. OSPool users should not have submitted a request to register as a Topology contact.

1.  In the top-right corner, click the "Add comment" link and add a note indicating how you verified the request

    !!! danger "'Approver Comment' is public"
        The registrant will see notes added to the "Approver Comment" field

1.  Click the "Approve" button.
    You should see "Petition Approved" and "Petition Finalized" on top.
    The Status should now be "Finalized".

1.  Click on their name next to `CO Person` to verify that the registrant is `Active` and that they are in the expected
    groups.

1.  The user will get an email saying "Petition for <NAME> changed status from Pending Approval to Approved".

Revoking AP login access
------------------------

Login access to AP1 (PATh Facility) and AP40 (OSPool) is controlled by membership to COManage groups.
To revoke a user's login access to either of these APs, perform the following steps:

1.  Find the corresponding user in [COManage](https://registry.cilogon.org/registry/co_dashboards/search?q=&co=7) and
    revoke access to all OSG services or just the relevant AP:

    1.  If you are revoking access to all OSG services, set the user's CO Person status to `Suspended`

    1.  If you only need to revoke access to AP1 or AP40, remove the user from the `ap1-login` or `ap40-login` group,
        respectively

1.  Note the `OSG Username` identifier of the user

1.  On the AP host(s) where you are revoking access, clear the SSSD cache as root:

        :::console
        root@ap-host # sss_cache -u <OSG Username>

    Replacing `<OSG Username>` with the `OSG Username` identifier that you noted in step (2)

CO Person Merging Procedure
---------------------------

__Terms:__

__Merge Target__: The CO Person that records are being merged onto.

__Merge Material__: The CO Person that records are being pulled from, should be marked as duplicate after completion.

<br>

__Records / Identifiers that should be moved:__

- Organization Identities (OrgIds)

    Includes `cilogon_id`s for OIDC, and an`ePPN` identifier for from the users SSO.

    In the `Authenticators` page of the Merge Material, click the gear icon for the given OrgID, choose "Relink".

    Search for the Merge Target in the "Person Picker" when redirected, click the "Relink" button next to the Merge Target Co Person. 

    Repeat from the Merge Material's `Authenticators` page for each OrgID.

- Emails

    For any Emails that aren't attached to OrgIds, just add a new Email Address Record to the Merge Target directly.

- SHHKeys

    Create a public key file on local disk from the values of `Key Type`, `Key` and `Comment` (in that order) for each key associated with the Merge Material. Then, in the `Authenticators` page of the Merge Target, click "Add SHH Key" and upload each of the new public key files.

- Github Identifier

    Create a new Identifier of the type `github` on the Merge Target, with the value from the one on the Merge Material

<br>

__Records / Identifiers that *shouldn't* be moved:__

- PIDs / GIDs and OSG Usernames

    If the Merge Target already has a Unix Cluster Account and Group, the Merge Material's UID and OSG Username shouldn't be merged. If the Merge Material's Unix Cluster Account has files/directories `chown`ed to it on hosts, `chown` them to the Merge Target's Unix Cluster Account.

    If the Merge Target doesn't have a Unix Cluster Account and Group, and the Merge Material does:

    And usernames / ids don't need to be preserved:

        - Click `AutoGenerate Cluster Accounts` on the Merge Target's `Clusters` page. Provision the user and re-`chown` files on hosts as need.

    And the usernames / id do need to be preserved:

      - {re-create Unix Cluster Account of Merge Material for Merge Target}, avoid this when possible.

- OSG IDs


