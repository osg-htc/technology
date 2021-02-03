The OSG and NSF Campus Cyberinfrastructure
==========================================

The NSF Campus Cyberinfrastructure (CC\*) program invests in coordinated campus-level cyberinfrastructure
improvements, innovation, integration, and engineering for science applications and distributed research projects, 
including  enhancements to campus networking and computing resources.

The [Open Science Grid](https://www.opensciencegrid.org) (as part of the 
[Partnership to Advance Throughput Computing (PATh)](https://path-cc.io/)), is here to help you with your 
Campus Cyberinfrastructure (CC\*) proposal!  Please contact us at 
[cc-star-proposals@opensciencegrid.org](mailto:cc-star-proposals@opensciencegrid.org)

We have significant experience working with CC\* applicants and awardees, offering letters of collaboration and 
consulting for:

- bringing the power of the OSG to YOUR researchers
- gathering science drivers and planning local computing resources 
- CC*-required resource sharing for the Campus Compute category\*, and other options for integrating local resources into OSG

\*In the most recent call for proposals ([NSF 21-528](https://www.nsf.gov/pubs/2021/nsf21528/nsf21528.htm)), joining the
OSG is mentioned as a potential path to sharing resources with
the wider research community:

> Proposals are required to commit to a minimum of 20% shared time on the cluster and describe their approach to making 
> the cluster available as a shared resource external to the campus, [...]
> **One possible approach to implementing such a federated distributed computing solution is joining a multi-campus or 
> national federated system such as the Open Science Grid.**


Sharing Resources via the OSG
-----------------------------

The OSG consortium provides standard services and support for computational resource providers (i.e., "sites") using a
[distributed fabric](https://map.opensciencegrid.org) of high throughput computating (HTC) technologies.
These distributed-HTC (dHTC) services communicate with the site's local resource management (e.g. "queueing") systems to provision
resources for OSG users.
The OSG itself does not own resources, but provides software and services that enable the sharing of resources by many sites, 
and enable users to take advantage of these from submission points (whether via an OSG-operated submission point, like 
[OSG Connect](https://www.osgconnect.net/), or a locally-managed one).

To contribute computational resources to the OSG, the following will be needed:

- An existing compute cluster running on a [supported operating system](https://opensciencegrid.org/docs/release/supported_platforms/)
  with a supported resource management system:
  [Grid Engine](http://www.univa.com/products/),
  [HTCondor](https://research.cs.wisc.edu/htcondor/),
  [LSF](https://www.ibm.com/us-en/marketplace/hpc-workload-management),
  [PBS Pro](https://www.altair.com/pbs-professional/)/[Torque](https://adaptivecomputing.com/cherry-services/torque-resource-manager/),
  [Slurm](https://slurm.schedmd.com/), and some local cloud provisioners.
- Outbound network connectivity from the cluster's worker nodes
- SSH access to your local cluster's submit node from a known IP address
- [Temporary scratch space](https://opensciencegrid.org/docs/worker-node/using-wn/#the-worker-node-environment) on each
  worker node and shared home directories on each cluster node
- Installation of some additional packages on the local cluster, IF the site would like to maximize its ability to support
  users, including those with large per-job data, containerized software, and/or GPU jobs.

(There ARE some exceptions to the above. [Contact us](mailto:support@opensciencegrid.org) to discuss them!)

!!!success "Next steps"
    If you are interested in OSG-offered services, please [contact us](mailto:support@opensciencegrid.org) for a
    consultation, even if your site does not meet all the conditions as outlined above!

Additional Materials
--------------------

If you are interested in learning more about the dHTC, OSG, and what it means to share resources via OSG services, consider
reviewing the following presentations from our [October 2020 workshop on dHTC and OSG services for campuses](https://indico.fnal.gov/event/45998/timetable/#20201022) ([YouTube Playlist](https://www.youtube.com/playlist?list=PLBWb4iScSWcPGfjvJztz_IeHCKS7f3u1k)):

- Intro to dHTC and PATh Services for Campuses ([slides](https://indico.fnal.gov/event/45998/contributions/199894/attachments/136489/169747/20201022OSGCampusWorkshop_dHTCServiceOverview_LMichael.pdf),[YouTube](https://www.youtube.com/watch?v=hYc6RaWL33g&list=PLBWb4iScSWcPGfjvJztz_IeHCKS7f3u1k&index=2&t=5s))
- How OSG Works ([slides](https://indico.fnal.gov/event/45998/contributions/199896/attachments/136496/169757/ccstarOctober2020-whatisosg.pdf), [YouTube](https://www.youtube.com/watch?v=kwIs7oa56t8&list=PLBWb4iScSWcPGfjvJztz_IeHCKS7f3u1k&index=4))
- Intro to OSG Resource Sharing ([slides](https://indico.fnal.gov/event/45998/contributions/199898/attachments/136497/169758/ccstarOctober2020-resourcesharing.pdf), [YouTube](https://www.youtube.com/watch?v=GuR8gxrVRew&list=PLBWb4iScSWcPGfjvJztz_IeHCKS7f3u1k&index=5))
- Resource Sharing Technology, Security, System Requirements, Setup Process ([slides](https://indico.fnal.gov/event/45998/contributions/199903/attachments/136555/169835/OSG-Sharing-Resources-CCStar-2020.pdf), [YouTube](https://www.youtube.com/watch?v=t5YtIIs7bqg&list=PLBWb4iScSWcPGfjvJztz_IeHCKS7f3u1k&index=7))
