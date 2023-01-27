Software Support
================

This document describes how OSG Technology Team members should support the OSG Software Stack, including triage duty
responsibilities and when to transition from direct support inquiries to a ticketing system such as Freshdesk or Jira.

Considerations
--------------

When providing support for our users, remember the following:

-   We are a small community and we need to take good care of our users.
    Please be friendly and patient even when the user is frustrated or lacking in knowledge.
-   Always sign your ticket with your full name, so people know who is responding.
    If it's easy for you, include a signature at the bottom of your response.
-   If you need to collect information about a problematic host, ask users to run `osg-system-profiler`.
    It can shorten the number of times you ask for information because it collects quite a bit for you.
-   If you run across a problem that has a chance of being hit by other users:
    -   Is there a bug we should fix in the software? Open a Jira ticket.
    -   Is there something we could improve in the software? Notify the Software Area Coordinator.
    -   Is there a way to improve our documentation? Notify the Software Area Coordinator.
    -   Can you extend our troubleshooting documents to help people track this down more quickly?
        Consider the troubleshooting documents to be as much for us as for our users.
    -   Is this something that other Technology Team members should be aware of?
        Note it during the support discussion during the weekly OSG Technology meeting, or email the Technology Team if
        it seems more urgent.

Triage Duty
-----------

The OSG uses [Freshdesk](#freshdesk) to track support issues so you will need a Freshworks account with agent privileges
(contact the OSG Software Team Area Coordinator for access).

!!!tip "Logging in as an agent"
    Don't enter your credentials directly into the [login page](https://support.opensciencegrid.org/support/login)!
    Click the agent login link instead so that you don't have to enter your credentials twice.

During normal work hours, the OSG Technology Team splits responsibilities for managing incoming OSG Software support
requests based upon a [weekly rotation](https://github.com/opensciencegrid/osg-triage-assignments/blob/master/rotation.txt).
If you are on triage duty, your responsibilities are as follows:

-   **Watch for new software tickets:** review the
    [Unresolved Software Tickets](https://support.opensciencegrid.org/a/tickets/filters/5000319518) and
    [All Unassigned Tickets](https://support.opensciencegrid.org/a/tickets/filters/5000322056) filters at least three
    times daily for new OSG Software-related tickets.
    For any such unassigned tickets, assign it as follows:
    -   *If you can handle an incoming ticket,* assign it to yourself.
        Inasmuch as possible, you should strive to handle the easier tickets and not pass them off to other people.
    -   *If you cannot handle an incoming ticket,* collect initial details such as relevant versions, logs, etc., and
        assign the ticket to the most appropriate Technology Team member.
        Where appropriate, CC people from other OSG teams, sites, or collaborations.

    !!! important "New sites interested in joining the OSG"
        For support requests inquiring about joining the OSG, assign the ticket to the `Campus Facilitation` group,
        and specifically `Lauren Michael` as the agent.

-   **Review assigned software tickets.**
    For tickets that are not being handled in a timely fashion (pay special attention to `OVERDUE` and `Customer
    Responded` tickets):
    -   *If the ticket is pending and the assignee has not responded in > 2 business days,*
        notify the ticket assignee via private note that they need to revisit the ticket.
    -   *If the ticket is waiting on the customer or a third party and they have not responded in > 1 week,*
        reply to the ticket asking if they've had the time to review the Technology Team's latest response(s).
    -   *If the ticket was opened by the customer, is waiting on the customer and they have not responded in > 2 weeks,*
        close the ticket and let the customer know that they can re-open it by responding whenever they're ready to
        tackle the issue again.
-   **Re-assign non-software tickets:**
    Tickets that have been mistakenly assigned to the Software group should be re-assigned to the appropriate group.
-   **Merge duplicate tickets:**
    Responses to a ticket sometimes results in creation of a new ticket; these new tickets should be merged into the
    original ticket.
    See [this documentation](<https://support.freshdesk.com/support/solutions/articles/80180-merging-two-or-more-tickets-together>).
-   **Clean up spam:**
    Mark the ticket as spam and block the user.
    See [this documentation](<https://support.freshdesk.com/support/solutions/articles/217539-spam-and-trash>).
-   **Clean up automated replies:**
    announcements are often sent with `Reply-to: help@opensciencegrid.org` so automated replies (e.g. Out of Office,
    mailing list moderation) will generate tickets.
    These tickets can be closed.

!!!question
    If you have questions concerning a ticket, consult the OSG Software Team Manager and/or the `#software` channel in
    the OSG Slack.

### Updating the triage calendar ###

The current triage duty schedule can be found in the OSG Software calendar, hosted on Tim Cartwright’s Google account.
If you need privileges to edit the calendar, ask the OSG Software Team Manager.
To update the triage duty schedule:

1.  Clone the [git repo](https://github.com/opensciencegrid/osg-triage-assignments)
1.  Generate next rotation:

        ./triage.py --generateNextRotation > rotation.txt

1.  Check and update assignments according to team member outages
1.  Load triage assignments into Google Calendar:

        ./triage.py --load rotation.txt

To subscribe to this calendar in your calendar program, use the iCal URL:
`https://www.google.com/calendar/ical/h5t4mns6omp49db1e4qtqrrf4g%40group.calendar.google.com/public/basic.ics`

<iframe src="https://www.google.com/calendar/embed?height=600&showPrint=0&wkst=1&bgcolor=%23FFFFFF&src=h5t4mns6omp49db1e4qtqrrf4g%40group.calendar.google.com&color=%232F6309&ctz=America%2FChicago" style=" border-width:0 " width="800" height="600" frameborder="0" scrolling="no"></iframe>

Ticket Systems
--------------

The OSG Technology Team uses the [Freshdesk](https://support.opensciencegrid.org/) and
[Jira](https://jira.opensciencegrid.org) ticketing systems to track support and all other work, respectively.
This section describes the differences between the two as well as some OSG Technology Freshdesk conventions.

### Direct Email ###

Sometimes users may email you directly with support inquiries.
If someone emails you directly for support, you have the choice of when to move it to a ticket.
The recommended criteria are:

-   If it's easy to handle and you can definitely do it yourself, leave it in email.
-   If there's a chance that you can't do it in a timely fashion, turn it into a ticket.
-   If there's a chance that you might lose track of the email, turn it into a ticket.
-   If there's a chance that you might need help from others, turn it into a ticket.
-   If it's an unusual topic and other people would benefit from seeing the ticket (now or in the future), turn it into
    a ticket.

### Freshdesk ###

!!!info "Freshdesk access"
    The OSG uses [Freshdesk](#freshdesk) to track support issues so you will need a Freshworks account with agent
    privileges (contact the OSG Software Team Manager for access).

Freshdesk tickets are for user support, i.e. this is where we help users debug, understand their problems, etc.
When replying to or otherwise updating a Freshdesk ticket, there are a few things to note:

-   Freshdesk auto-populates the contact's name when replying through the web interface, e.g. `Hi Brian`.
    Ensure that the name is correct, especially if there are multiple parties involved in a single ticket.
    If the auto-populated name looks incorrect, e.g. `Hi blin.wisc`, fix the contact's First and Last name fields.
-   Make sure to set the state of the ticket, which is helpful for those on triage:

    | **State**              | **Description**                                                             |
    |------------------------|-----------------------------------------------------------------------------|
    | Open                   | OSG staff is responsible for next actions, including when ticket has not yet been assigned (initial ticket state) |
    | Waiting on Customer    | Assignee needs the reporter to respond                                      |
    | Waiting on Third Party | Assignee needs a response from a CC                                         |
    | Resolved               | Support is complete or the user is unresponsive. [See above](#triage-duty). |
    | Closed                 | **DO NOT SET MANUALLY**. Terminal ticket state that is set by Freshdesk.    |

If actionable Technology Team tasks arise from a Freshdesk ticket, [Jira](#jira) ticket(s) should be created to track
that work.
Resultant Jira tickets should include a link to the original Freshdesk ticket, a description of the problem or feature
request, and a proposed solution or implementation.

After the relevant Jira tickets have been created, ask the user if they would be ok with tracking the issue via Jira. 
If they say yes, close the Freshdesk ticket.

### Jira ###

Jira is for tracking our work and it's meant for internal usage, not for user support.
In general, users should not ask for support via Jira.
A single user support ticket might result in zero, one, or multiple Jira tickets.
