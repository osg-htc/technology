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
            You can view the list of Topology registrations [here](https://registry.cilogon.org/registry/co_petitions/index/co:7/sort:CoPetition.created/direction:desc/search.status:PA).)

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
