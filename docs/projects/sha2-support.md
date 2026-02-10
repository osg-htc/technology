SHA-2 Compliance
================

When a certificate authority signs a certificate, it uses one of several possible hash algorithms. 
Historically, the most popular algorithms were MD5 (now retired due to security issues) and the SHA-1 family.
SHA-1 certificates are being phased out due to perceived weaknesses — as of February 2017, a practical attack for generating collisions was demonstrated by Google researchers.
 These days, the preferred hash algorithm family is SHA-2.

The certificate authorities (CAs), which issue host and user certificates used widely in the OSG, defaulted to SHA-2-based certificates on 1 October 2013; all sites will need to make sure that their software supports certificates using the SHA-2 algorithms. All supported OSG releases support SHA-2.

The table below denotes indicates the minimum releases necessary to support SHA-2 certificates.

| Component               | Version                                                                                                                                     | In Release | Notes                                                             |
|:------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------|:-----------|:------------------------------------------------------------------|
| BeStMan 2               | bestman2-2.3.0-9.osg                                                                                                                        | 3.1.13     | SHA-2 support; also see jGlobus, below                            |
| dCache SRM client       | dcache-srmclient-2.2.11.1-2.osg                                                                                                             | 3.1.22     | Major update includes SHA-2 support                               |
| Globus GRAM             | globus-gram-job-manager-13.45-1.2.osg, globus-gram-job-manager-condor-1.0-13.1.osg, globus-gram-job-manager-pbs-1.6-1.1.osg | 3.1.9      | Critical bug fixes (not SHA-2 specific)                           |
| GUMS                    | gums-1.3.18.009-15.2.osg                                                                                                                    | 3.1.13     | Switched to jGlobus 2 with SHA-2 support; also see jGlobus, below |
| jGlobus (for BeStMan 2) | jglobus-2.0.5-3.osg                                                                                                                         | 3.1.18     | Fixed CRL refresh bug (not SHA-2 specific)                        |
| VOMS                    | voms-2.0.8-1.5.osg                                                                                                                          | 3.1.17     | SHA-2 fix for voms-proxy-init                                     |

If a component does not appear in the above table, it already has SHA-2 support.

