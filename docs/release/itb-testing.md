# Testing OSG Software Prereleases on the Madison ITB Site

This document contains basic recipes for testing a OSG software prereleases on the Madison ITB site, which includes
HTCondor prerelease builds and full OSG software stack prereleases from Yum.


## Prerequisites

The following items are known prerequisites to using this recipe.  If you are not running the Ansible commands from
osghost, there are almost certainly other prerequisites that are not listed below.  And even using osghost for Ansible
and itb-submit for the submissions, there may be other prerequisites missing.  Please improve this document by adding
other prerequisites as they are identified!

* A checkout of the osgitb directory from our local git instance (not GitHub)
* Your X.509 DN in the `osgitb/unmanaged/htcondor-ce/grid-mapfile` file and (via Ansible) on `itb-ce1` and `itb-ce2`


## Gathering Information

Technically skippable, this section is about checking on the state of the ITB machines before making changes.  The plan
is to keep the ITB machines generally up-to-date independently, so those steps are not listed here.  And honestly, the
steps below are just some ideas; do whatever makes sense for the given update.

The commands can be run as-is from within the `osgitb` directory from git.

1. Check OS versions for all current ITB hosts:

        :::console
        ansible current -i inventory -f 20 -o -m command -a 'cat /etc/redhat-release'

1. Check the date and time on all hosts (in case NTP stops working):

        :::console
        ansible current -i inventory -f 20 -o -m command -a 'date'

1. Check software versions for certain hosts (e.g., for the `condor` package on hosts in the `workers` group):

        :::console
        ansible workers -i inventory -f 20 -o -m command -a 'rpm -q condor'


## Installing HTCondor Prerelease

Use this section to install a new version of HTCondor, specifically a prerelease build from the development or
upcoming-development repository, on the test hosts.

1. Obtain the NVR of the HTCondor prerelease build from OSG to test.  Do this by talking to Tim&nbsp;T. and checking
   Koji.

1. Shut down HTCondor and HTCondor-CE on prerelease machines:

        :::console
        ansible 'testing:&ces' -i inventory -bK -f 20 -m service -a 'name=condor-ce state=stopped'
        ansible 'testing:&condor' -i inventory -bK -f 20 -m service -a 'name=condor state=stopped'

1. Install new version of HTCondor on prerelease machines:

        :::console
        ansible 'testing:&condor' -i inventory -bK -f 10 -m command -a 'yum --enablerepo=osg-development --assumeyes update condor'

    or, if you need to install an NVR that is ???earlier??? (in the RPM sense) than what is currently installed:

        :::console
        ansible 'testing:&condor' -i inventory -bK -f 10 -m command -a 'yum --enablerepo=osg-development --assumeyes downgrade condor condor-classads condor-python condor-procd blahp'

1. Verify correct RPM versions across the site:

        :::console
        ansible condor -i inventory -f 20 -o -m command -a 'rpm -q condor'

1. Restart HTCondor and HTCondor-CE on prerelease machines:

        :::console
        ansible 'testing:&condor' -i inventory -bK -f 20 -m service -a 'name=condor state=started'
        ansible 'testing:&ces' -i inventory -bK -f 20 -m service -a 'name=condor-ce state=started'


## Installing a Prerelease of the OSG Software Stack

Use this section to install new versions of all OSG software from a prerelease repository in Yum.

1. Check with the Release Manager to make sure that the prerelease repository has been populated with the desired
   package versions.

1. Make sure that software is generally up-to-date on the hosts&nbsp;??? see
   [the Madison ITB Site doc](https://docs.google.com/document/d/11Njz9YMWg67f_TMzcrbdD7anZRIsf9-wiXx-inWhO4U/edit#bookmark=id.4d34og8) for more details

    It may be desirable to update only non-OSG software at this stage, in which case one could simply disable the OSG
    repositories by adding command-line options to the `yum update` commands.

1. Install new software on prerelease hosts:

        :::console
        ansible testing -i inventory -bK -f 20 -m command -a 'yum --enablerepo=osg-prerelease --assumeyes update'

1. Read the Yum output carefully, and follow up on any warnings, etc.

1. If the `osg-configure` package was updated on any host(s), run the `osg-configure` command on the host(s):

        :::console
        ansible testing -i inventory -bK -f 20 -m command -a 'osg-configure -v' -l [HOST(S)]
        ansible testing -i inventory -bK -f 20 -m command -a 'osg-configure -c' -l [HOST(S)]

1. Verify OSG software updates by inspecting the Yum output carefully or examining specific package versions:

        :::console
        ansible current -i inventory -f 20 -o -m command -a 'rpm -q osg-wn-client'

    Use an inventory group and package names that best fit the situation.


## Running Tests

For the first two test workflows, use your personal space on `itb-submit`.  Copy or checkout the `osgitb/htcondor-tests`
directory to get the test directories.

### Part ???: Submitting jobs directly

1. Change into the `1-direct-jobs` subdirectory

1. If there are old result files in the directory, remove them:

        :::console
        make distclean

1. Submit the test workflow

        :::console
        condor_submit_dag test.dag

1. Monitor the jobs until they are complete or stuck

    In the initial test runs, the entire workflow ran in a few minutes.  If the DAG or jobs exit immediately, go on
    hold, or otherwise fail, then you have some troubleshooting to do!  Keep trying steps 2 and 3 until you get a clean
    run (or one or more HTCondor bug tickets).

1. Check the final output file:

        :::console
        cat count-by-hostnames.txt

    You should see a reasonable distribution of jobs by hostname, keeping in mind the different number of cores per
    machine and the fact that HTCondor can and will reuse claims to process many jobs on a single host.  Especially
    watch out for a case in which no jobs run on the newly updated hosts (at the time of writing: `itb-data[456]`).

1. (Optional) Clean up, using the `make clean` or `make distclean` commands.  Use the `clean` target to remove
   intermediate result and log files generated by a workflow run but preserve the final output file; use the `distclean`
   target to remove all workflow-generated files (plus Emacs backup files).


### Part ???: Submitting jobs using HTCondor-C

If direct submissions fail, there is probably no point to doing this step.

1. Change into the `2-htcondor-c-jobs` subdirectory

1. If there are old result files in the directory, remove them:

        :::console
        make distclean

1. Get a proxy for your X.509 credentials

        :::console
        voms-proxy-init

1. Submit the test workflow

        :::console
        condor_submit_dag test.dag

1. Monitor the jobs until they are complete or stuck

    In the initial test runs, the entire workflow ran in 10 minutes or less; generally, this test takes longer than the
    direct submission test, because of the layers of indirection.  Also, status updates from the CEs back to the submit
    host are infrequent.  For direct information about the CEs, log in to `itb-ce1` and `itb-ce2` to check status; don???t
    forget to check both `condor_ce_q` and `condor_q` on the CEs, probably in that order.

    If the DAG or jobs exit immediately, go on hold, or otherwise fail, then you have some troubleshooting to do!  Keep
    trying steps 2 and 3 until you get a clean run (or one or more HTCondor bug tickets).

1. Check the final output file:

        :::console
        cat count-by-hostnames.txt

    Again, look for a reasonable distribution of jobs by hostname.

1. (Optional) Clean up, using the `make clean` or `make distclean` commands.

### Part ???: Submitting jobs from a GlideinWMS VO Frontend

For this workflow, use your personal space on `glidein3.chtc.wisc.edu`.  Copy or checkout the `osgitb/htcondor-tests`
directory to get the test directories.  Again, if previous steps fail, do not bother with this step.

1. Change into the `3-frontend-jobs` subdirectory

1. If there are old result files in the directory, remove them:

        :::console
        make distclean

1. Submit the test workflow

        :::console
        condor_submit_dag test.dag

1. Monitor the jobs until they are complete or stuck

    This workflow could take much longer than the first two, maybe an hour or so.  Also, unless there are active
    glideins, it will take 10 minutes or longer for the first glideins to appear and start matching jobs.  Thus it is
    helpful to monitor `condor_q -totals` until all of the jobs are submitted (there should be 2001), then switch to
    monitoring `condor_status` until glideins start appearing.  After the first jobs start running and finishing, it is
    probably safe to ignore the rest of the run.  If the jobs do not appear in the local queue, if glideins do not
    appear, or if jobs do not start running on the glideins, it is time to start troubleshooting.

1. Check the final output file:

        :::console
        cat count-by-hostnames.txt

    The distribution of jobs per execute node may be more skewed than in the first two workflows, due to the way in
    which pilots ramp up over time and how HTCondor allocates jobs to slots.

1. (Optional) Clean up, using the `make clean` or `make distclean` commands.
