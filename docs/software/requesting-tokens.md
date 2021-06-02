How to Request Tokens
=====================

As part of the [GridFTP and GSI migration](../policy/gridftp-gsi-migration.md), the OSG will be transitioning authentication
away from X.509 certificates to the use of bearer tokens such as [SciTokens](http://scitokens.org/) or
[WLCG JWT](https://twiki.cern.ch/twiki/bin/view/LCG/WLCGAuthorizationWG).
This document is intended as a guide for OSG developers for requesting tokens necessary for software development.

Before Starting
---------------

Before you can request the appropriate tokens, you must have the following:

-   A [WLCG INDIGO IAM](https://wlcg.cloud.cnaf.infn.it/) account belonging to the `wlcg`, `wlcg/pilots`, and `wlcg/xfers`
    groups.
-   An installation of [oidc-agent](https://indigo-dc.gitbook.io/oidc-agent/) available as an RPM from the OSG
    repositories

Requesting Tokens
-----------------

`oidc-agent` is similar to SSH agent except that it works with OpenID Connect token providers.

1. Start the agent and add the appropriate variables to your environment:

        :::console
        eval `oidc-agent`

1. Generate a local client profile and follow the prompts:

        :::console
        oidc-gen -w device <CLIENT NAME>

    1. Specify the WLCG INDIGO IAM instance as the client issuer:

            Issuer [https://iam-test.indigo-datacloud.eu/]: https://wlcg.cloud.cnaf.infn.it/

    1. Request `wlcg`, `offline_access`, and other scopes for the capabilities that you need:

        | **Capability**   | **Scope**                     |
        |------------------|-------------------------------|
        | HTCondor `READ`  | `compute.read`                |
        | HTCondor `WRITE` | `compute.modify compute.cancel compute.create` |
        | XRootD read      | `read:/`                      |
        | XRootD write     | `write:/`                     |

        For example, to request HTCondor `READ` and `WRITE` access, specify the following scopes:

            This issuer supports the following scopes: openid profile email address phone offline_access wlcg iam wlcg.groups
            Space delimited list of scopes or 'max' [openid profile offline_access]: wlcg offline_access compute.read compute.modify compute.cancel compute.create
    
        Note that, prior to HTCondor 8.9.7, the server also needed `condor:/ALLOW` in all cases.

    1. When prompted, open <https://wlcg.cloud.cnaf.infn.it/device> in a browser, enter the code provided by `oidc-gen`,
       and click "Submit".

    1. On the next page, verify the scopes and client profile name, and click "Authorize".

    1. Enter a password to encrypt your local client profile.
       You'll need to remember this if you want to re-use this profile in subsequent sessions.

1. Request a token using the client name that you used above with `oidc-gen`:

        :::console
        oidc-token <CLIENT NAME>

1. Copy the output of `oidc-token` into a file on the host where you need SciToken authentication, e.g. an HTCondor or
   XRootD client.

Troubleshooting Tokens
----------------------

You can inspect the payload  by copy-pasting the token into the "Encoded" text box here: <http://jwt.io/>.
Mouse over the fields and values for details.
