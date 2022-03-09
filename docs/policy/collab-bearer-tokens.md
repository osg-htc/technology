Collaborations and Bearer Tokens
================================

Sites in the OSG grant access to their grid services based on client's association with a specific collaboration
(i.e. VO) instead of granting access on a per-user basis.
In the past, this type of access was provided through X.509 proxies with VOMS attributes to demonstrate assocation with
a collaboration:

-   Users (both human and robots) would request that their collaboration sign their X.509 proxies
    (usually through a VOMS server),
-   Or in the case of automated services (i.e. pilot job submission) a collaboration could directly create and sign
    proxies themselves

Now, sites can authenticate and authorize clients presenting bearer tokens, such as SciTokens or WLCG tokens.
This document describes how collaborations can issue bearer tokens for the aforementioned use cases in ways that are
compatible with OSG sites.

Issuers
-------

To generate bearer tokens, a collaboration must adminster at least one "token issuer" to issue tokens to their users.
In addition to generating and signing tokens, token issuers provide a public endpoint that can be used to validate an
issued token,
e.g. an OSG Compute Entrypoint (CE) will contact the token issuer to authorize a bearer token used for pilot job
submission.

!!! attention "Token issuer uptime"
    Due to the centralized nature of bearer token validation, token issuers should be treated as critical, highly
    available services.
    Otherwise, a token issuer outage will result in OSG sites being unable to authenticate a collaboration's tokens,
    meaning an interruption in pilot job submission and authenticated data transfers.

Choose one of the token issuer types below, depending on the needs of your collaboration.

### Simple issuer ###

If your collaboration centrally administers all services requiring bearer tokens and your users do not need to directly
manage bearer tokens, consider running a simple token issuer.
A simple token issuer consists of a public/private certificate keypair where the private key is used to issue tokens
directly and the public certificate is made available through a web server.

For example, the OSPool (née OSG VO) serves its public certificate through
[GitHub pages](https://github.com/scitokens/osg-connect/) and uses the private key to sign tokens used for pilot job
submission as well as automatically generating tokens to accompany user jobs so that they can access their private
storage areas.

### OAuth2/OpenID Connect ###

If your collaboration distributes administrative responsibility or your users need to request and manage their own
tokens,
you should administer an OAuth2/OpenID Connect (OIDC) service (e.g., [INDIGO IAM](https://indigo-iam.github.io/v/v1.7.2/))
or work with an existing OAuth2/OIDC provider (e.g., [CILogon](https://www.cilogon.org/subscribe)).

For example, Fermilab uses CILogon as their OIDC provider combined with an
[HTVault server](https://github.com/fermitools/htvault-config) used to streamline the OIDC process for users and
integrate with their local Kerberos.

Claims
------

Bearer tokens are self-describing credentials that enumerate their capabilities as "claims" and different token
"profiles" enumerate common sets of claims.
OSG sites support the following bearer token profiles:

-   [SciTokens](https://scitokens.org/technical_docs/Claims)
-   [WLCG tokens](https://github.com/WLCG-AuthZ-WG/common-jwt-profile/blob/master/profile.md)

Claims are further described below with recommendations to ensure the greatest compatibility with OSG sites.

### Scope ###

The `scope` claim is a space-separated list of authorizations that should be granted to the bearer.
Scopes utilized by OSG services include the following:

| **Capability**   | **SciTokens scope** | **WLCG scope**                                 |
|------------------|---------------------|------------------------------------------------|
| HTCondor `READ`  | `condor:/READ`      | `compute.read`                                 |
| HTCondor `WRITE` | `condor:/WRITE`     | `compute.modify compute.cancel compute.create` |
| XRootD read      | `read:<PATH>`       | `storage.read:<PATH>`                          |
| XRootD write     | `write:<PATH>`      | `storage.modify:<PATH>`                        |

Replacing `<PATH>` with a path to the storage location that the bearer should be authorized to access.

### Issuer ###

The issuer URL, or the `iss` claim, indicates the endpoint to use for authenticating a given token.
A collaboration may have more than one token issuer but a single token issuer should never serve more than one
collaboration.
In other words, given the token issuer, the site can determine the collaboration that issued the token.
The following collaborations have registered token issuers with the OSG:

| **Collaboration** | **Issuers**                                                                                   |   |
|-------------------|-----------------------------------------------------------------------------------------------|---|
| ATLAS             | <https://github.com/opensciencegrid/topology/blob/master/virtual-organizations/ATLAS.yaml>    |   |
| CLAS12            | <https://github.com/opensciencegrid/topology/blob/master/virtual-organizations/CLAS12.yaml>   |   |
| CMS               | <https://github.com/opensciencegrid/topology/blob/master/virtual-organizations/CMS.yaml>      |   |
| DES               | <https://github.com/opensciencegrid/topology/blob/master/virtual-organizations/DES.yaml>      |   |
| DUNE              | <https://github.com/opensciencegrid/topology/blob/master/virtual-organizations/DUNE.yaml>     |   |
| EIC               | <https://github.com/opensciencegrid/topology/blob/master/virtual-organizations/EIC.yaml>      |   |
| Fermilab          | <https://github.com/opensciencegrid/topology/blob/master/virtual-organizations/Fermilab.yaml> |   |
| CHTC (née GLOW)   | <https://github.com/opensciencegrid/topology/blob/master/virtual-organizations/GLOW.yaml>     |   |
| Gluex             | <https://github.com/opensciencegrid/topology/blob/master/virtual-organizations/Gluex.yaml>    |   |
| IceCube           | <https://github.com/opensciencegrid/topology/blob/master/virtual-organizations/IceCube.yaml>  |   |
| LIGO              | <https://github.com/opensciencegrid/topology/blob/master/virtual-organizations/LIGO.yaml>     |   |
| Mu2e              | <https://github.com/opensciencegrid/topology/blob/master/virtual-organizations/Mu2e.yaml>     |   |
| OSPool            | <https://github.com/opensciencegrid/topology/blob/master/virtual-organizations/OSG.yaml>      |   |
| SBND              | <https://github.com/opensciencegrid/topology/blob/master/virtual-organizations/SBND.yaml>     |   |
| gm2               | <https://github.com/opensciencegrid/topology/blob/master/virtual-organizations/gm2.yaml>      |   |

The OSG distributes the `osg-scitokens-mapfile` RPM package that includes issuer and [subject](#subject) to default user
mappings for use by OSG CEs.

### Subject ###

Subjects (i.e., the `sub` claim) should be unique, stable identifiers that correspond to a user or service
(e.g. pilot job submission).
In other words, subjects combined with a token issuer can be used for suspending access for a given collaboration user,
user-level accounting, monitoring, auditing, or tracing.
In tandem with a token issuer URL (i.e., the `iss` claim), subjects can be used by site HTCondor-CE or XRootD services
to map to a local identity.

!!! tip "Privacy considerations"
    Depending on your collaboration's userbase and contributing sites, you may have to take privacy concerns, such as
    the GDPR into account when assigning subjects to users.
    Thus, it may be preferable to assign user a randomly-generated string as their `sub`.

### Audience ###

To take advantage of the improved security posture of bearer tokens, we recommend that the `aud` claim be set to the
intended host.
For example, tokens used for submission to an HTCondor-CE should set the following:

```
aud = <CE FQDN>:<CE PORT>
```

### WLCG groups ###

WLCG tokens may have the `wlcg.groups` claim consisting of a comma and space separated list of collaboration groups.
The format of these groups are similar to VOMS FQANs: `/<collaboration>[/<group>][/Role=<role>]`,
replacing `<collaboration>`, `<group>`, and `<role>` with the collaboration, group, and role, respectively, where the
group and role are optional.
For example, the following groups and roles have been used by the ATLAS and CMS collaborations:

```
/atlas/
/atlas/usatlas
/cms/Role=pilot
/cms/local/Role=pilot
```

Traditionally, sites have made local accounting and scheduling decisions based on the first VOMS FQAN so collaborations
should set the first group/role in `wlcg.groups` to the most specific group or role.
For example:

```
wlcg.groups = /cms/Role=pilot, /cms
```

Instead of:

```
wlcg.groups = /cms, /cms/Role=pilot
```

Help
----

To get assistance, please use the [this page](https://opensciencegrid.org/docs/common/help/).
