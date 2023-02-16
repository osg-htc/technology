# Containers in the OSPool

Tim C. started this document in February 2023 to remember things from hallway discussions.
If the scope of this page broadens, be sure to update the title and/or this description.

## Slot Ad Attributes for Containers

There are two seemingly similar Slot Ad attributes in the OSPool related to Singularity:

*   `HasSingularity` is
    [an HTCondor attribute](https://htcondor.readthedocs.io/en/latest/classad-attributes/machine-classad-attributes.html#HasSingularity)
    that indicates whether jobs can run within Singularity containers.
    It is set to `true` based on a test that HTCondor performs at start-up,
    although subsequent container invocations could revoke the value upon certain failure conditions.

*   `HAS_SINGULARITY` is an OSG pool attribute that indicates whether user payload jobs
    can _and will_ run within Singularity containers.
    Only Singularity container runtimes are supported.
    It is set to `true` based on a test that is run in the pilot scripts.
    Note that the test is also baked into the OSPool Backfill Containers.
    The pilot-script test checks more conditions than the HTCondor `HasSingularity` one.
    Periodic scripts in the pilot (i.e., `STARTD_CRON`) retest some of these conditions;
    thus, `HAS_SINGULARITY` may start out `true` but become `false` later in the pilot’s lifetime.

When an OSPool job asks to run inside a container,
`requirements` are set to check that both `HasSingularity` and `HAS_SINGULARITY`
are true for any matching Slot Ad.

## Containers and Pilots

Obviously, Backfill Container “pilots” run inside a container runtime,
although the specific choice of container runtime technology is up to the site.

Within a GlideinWMS pilot, the pilot scripts determine whether a functioning Singularity container is available,
which could come from a local install on each Execution Point or from CVMFS.
If detected, then `HAS_SINGULARITY` is set to `true` and
_all_ user payloads will be run in containers.
(It is possible to override this behavior through site-specific pilot hackery.)

For Backfill Containers, the container itself includes an installation of Apptainer (née Singularity)
and it will always be used (in unprivileged mode) instead of any system or other installation.

The user job is a vanilla job.
Today, though, a pilot runs the “user job wrapper” script,
which is a replacement for the user’s executable that does some stuff and
then runs the user’s executable.
If the `HAS_SINGULARITY` attribute is `true` in the environment of the wrapper,
then it runs the actual user payload job in Singularity (or Apptainer).

## PID Namespaces

PID namespaces are a key technology that enables containers to isolate from each other.
See, for example,
[this Ubuntu copy of the man page](https://manpages.ubuntu.com/manpages/bionic/man7/pid_namespaces.7.html)
for `pid_namespaces`.
The `root` user always has the ability to create PID namespaces, so a privileged container runtime
(i.e., not unprivileged Singularity) can always do this.

There is a special feature for _user PID namespaces_, which can be created by unprivileged processes.
To work, there is a certain kernel setting that must be set,
and then user PID namespaces are available to all.

Today, when the GlideinWMS user job wrapper is about to start a user payload job in a Singularity container,
the default is to pass a flag to Singularity to use user PID namespaces.
However, if user PID namespaces are not available (say, due to the kernel setting),
the pilot start-up scripts do not detect this condition automatically.
So, there is a special envirnoment variable that can be set on the outer container
(e.g., the Backfill Container) to disable this flag:

```SINGULARITY_DISABLE_PID_NAMESPACES=1```

While set on the outer container, it affects only the user job wrapper and
how it invokes the inner Singularity container.

Mat says that HTCondor 10.2.2 has a feature to detect the lack of user PID namespaces and,
in such a case, to avoid using them, but it is not clear that that feature will help anything
as long as the GlideinWMS user job wrapper is being used.

## Containers and GPUs

Most, but certainly not all, OSPool payload jobs that request GPUs also request to run within a container.
This is probably due to GPU-using software stacks often being complex,
and so users often turn to containers for their runtime environment,
including ones that we and others pre-build.

Note that the Backfill Container images themselves do not include the NVIDIA CUDA drivers
(although, we are experimenting with that).
Instead, the CUDA drivers that are installed on the host system are mounted inside the running Backfill Container.
This scheme, which works at most OSPool sites, allows GPU payload jobs to work
without specifically requesting a container,
although see above for why many do so anyway.
