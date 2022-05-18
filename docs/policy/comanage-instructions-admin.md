Approving COManage Registrations
================================

OSG is using a new identity management system called COManage.
Initially, this system will be used for managing access to OSG staff-internal monitoring webpages at UNL,
and managing access for OASIS logins.

User registrations must be manually approved by a COManage admin.
Follow the instructions below to approve a user registration.

!!! note
    This page is for COManage Admins who want to approve user registrations.
    If you are a user who wants to register with COManage,
    go to the [Registering for the OSG COManage](../policy/comanage-instructions-user.md) page instead.


1.  Check for an email from <registry@cilogon.org> saying "Petition for <NAME> changed status from
    Confirmed to Pending Approval".

1.  Click the first link in the email.

1.  Log in with your institutional credentials.
    You should see a "View CO Petition..." page.
    The Status should be "Pending Approval".
    
    !!! note
    	You can also access this page by logging into https://registry.cilogon.org/registry and
    	clicking on "CO Petitions" under the left-hand "People" menu. 

1.  Click on the name of the pending application. 
	This will take to you the page where you can see their full application and 
	have the option to approve them. 

1.  Verify that the application is legitimate.
    Ask someone affiliated with the site, VO, or the sponsor of a project to verify the applicant's affiliation.
    In the future, applicants will be asked to provide the contact information of such a person --
    similar to our existing [contact registration process](https://osg-htc.org/docs/common/registration/).

1.  Click the "Approve" button.
    You should see "Petition Approved" and "Petition Finalized" on top.
    The Status should now be "Finalized".

1.  The user will get an email saying "Petition for <NAME> changed status from Pending Approval to
    Approved".

    The links in the email are expected to be useless
    (the first link will give them "Permission Denied" and
    the second link will merely allow them to "acknowledge" that they received the email).

    If the user asks, reassure them that they are registered and no further action needs to be taken.

Troubleshooting
---------------

### The COManage petition is stuck in the "confirmed" state

1.  Under the `People` drop-down on the left,  click on `My Population`

1.  Browse to the `CO Person` record.

1.  Scroll down to the `Role`, click `Edit`, and set the status for the `Role` to `Active`.

1.  Verify that the overall status of the `CO Person` record is `Active`.  If not, change it to `Active` as well.

1.  Click on `Autogenerate Identifiers` on the right, so that the necessary identifiers are created.

1.  Now that the necessary identifiers exist for the `CO Person` record, the LDAP DN can be computed and the record
    provisioned in LDAP. To make sure, click on `Provisioned Services` and then  `Provision`.
