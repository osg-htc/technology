DateReviewed: 2020-10-08

Registering with the OSG
========================

OSG uses the [COManage](https://www.internet2.edu/products-services/trust-identity/comanage/) identity management system
to register OSG contacts.
COManage is backed by the [InCommon federation](https://www.incommon.org/federation/), meaning that users can register
with the OSG using their institutional identities with familiar single sign-on forms.

Submitting an Application
-------------------------

To register with the OSG, submit an application using the self-signup process:

1.  Visit <https://opensciencegrid.org/register>

1.  You will be presented with a CILogon Single-Sign On page.
    Select your insitution and sign in with your insitutional credentials:

    ![CILogon Single-Sign On page](../img/comanage/comanage-sso.png)

1.  After you have signed in, you will be presented with the self-signup form.
    Click the "BEGIN" button:

    ![COManage self-signup form](../img/comanage/comanage-landing-page.png)

1.  Enter your name and email address.
    In most cases, your institution will provide defaults for your name and email address.
    If you prefer, you may override these values.
    Click the "SUBMIT" button:

    ![COManage enrollment form](../img/comanage/comanage-enrollment-form.png)

Verifying Your Email Address
----------------------------

After submitting your registration application, you will receive an email from <registry@cilogon.org> to verify your email
address.
Follow the link in the email and click the "Accept" button to complete the verification:

![COManage verification email](../img/comanage/comanage-email-verification-form.png)

Waiting for Approval
--------------------

After verifying your email address, your registration must be approved by OSG staff.
Once your registration has been approved, you will receive an email confirming your OSG registration:

![COManage approval email](../img/comanage/comanage-verified-email.png)

OASIS Managers: Adding an SSH Key
---------------------------------

After approval by OSG staff, [OASIS managers](https://opensciencegrid.org/docs/data/update-oasis/) must upload a public
SSH key before being able to access the OASIS login host:

1.  Visit <https://opensciencegrid.org/register> and login if prompted

1.  Click your name in the top right to get a dropdown and click the `My Profile (OSG)` button

    ![COManage profile dropdown](../img/comanage/ssh-homepage-dropdown.png)

1.  On the right-side of your profile, click the `Authenticators` link:

    ![COManage user profile](../img/comanage/ssh-edit-profile.png)

1.  On the authenticators page, click the `Manage` button:

    ![COManage authenticators](../img/comanage/ssh-authenticator-select.png)

1.  On the SSH keys page, click the `Add SSH Key` link:

    ![COManage SSH keys](../img/comanage/ssh-key-list.png)

1.  Finally, upload your public SSH key from your computer:

    ![COManage upload SSH key](../img/comanage/ssh-add-key.png)

Getting Help
------------

For assistance with the OSG contact registration process, please use
[this page](https://opensciencegrid.org/docs/common/help/).
