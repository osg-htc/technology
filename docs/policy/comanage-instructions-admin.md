Approving COManage Registrations
================================

OSG is using a new identity management system called COManage.
This system is used for managing contact information for OSPool and PATh Facility users, Topology site contacts, and
OSG/PATh staff.

User registrations must be manually approved by a COManage admin.
Follow the instructions below to approve a user registration.

!!! note
    This page is for COManage Admins who want to approve user registrations.
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

        !!! note
            Many groups share our COManage instance so make sure that you're only approving registration requests for
            the appropriate group, e.g. site contacts.

1.  If prompted, log in with your institutional credentials.

1.  Review the request:

    1.  Verify that the request is legitimate by doing at least one of the following:

        -   Find associated support tickets by searching for their email address in Freshdesk
        -   Ask someone affiliated with the site, collaboration, or the sponsor of a project to verify the registrant's
            affiliation.
        -   Ask if other staff have been in contact with them via the `#staff` Slack channel

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

Troubleshooting
---------------

### The COManage petition is stuck in the "confirmed" state

This may happen if there are issues when confirming the user's email address.
We have seen this occur if a user clicks the confirmation link then closes the tab too quickly.

1.  Under the `People` drop-down on the left,  click on `My Population`

1.  Browse to the `CO Person` record.

1.  Scroll down to the `Role`, click `Edit`, and set the status for the `Role` to `Active`.

1.  Verify that the overall status of the `CO Person` record is `Active`.  If not, change it to `Active` as well.

1.  Click on `Autogenerate Identifiers` on the right, so that the necessary identifiers are created.

1.  Now that the necessary identifiers exist for the `CO Person` record, the LDAP DN can be computed and the record
    provisioned in LDAP. To make sure, click on `Provisioned Services` and then  `Provision`.
