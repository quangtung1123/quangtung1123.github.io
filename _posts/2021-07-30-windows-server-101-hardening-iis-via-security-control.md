---
layout: post
title:  "Windows Server 101 Hardening IIS via Security Control Configuration"
date:   2021-07-30 15:10:00
permalink: 2021/07/30/windows-server-101-hardening-iis-via-security-control
tags: Security IIS Windows_server
category: Security
img: /assets/windows-server-101-hardening-iis-via-security-control/Hinh1.png
summary: "Windows Server 101 Hardening IIS via Security Control Configuration"

---

IIS, the web server that’s available as a role in Windows Server, is also one of the most used web server platforms on the internet. Hardening IIS involves applying a certain configuration steps above and beyond the default settings. The default settings on IIS provide a mix of functionality and security. As with any hardening operation, the harder you make a configuration, the more you reduce functionality and compatibility.

The two important third party guides for hardening IIS are the OWASP guide and the Center for Internet Security guide. You can access these guides here:

- Center for [Internet Security IIS 10 Benchmark](https://www.cisecurity.org/cis-benchmarks/)

The CIS IIS 10 benchmark is more fleshed out at the time of writing and is an approximately 140 page PDF with 55 separate security recommendations. The OWASP guide is shorter and provides approximately 23 separate security recommendations.

Table 1.1 provides a high level list of the CIS IIS 10 benchmarks. For more detail on how to implement and check each security control, download the CIS IIS 10 benchmark file from the above website.

 

Table 1.1: High Level Center for Internet Security IIS 10 Security Controls

1.       Basic Configurations

   1.1.    Ensure web content is on non-system partition

   1.2.    Ensure 'host headers' are on all sites

   1.3.    Ensure 'directory browsing' is set to disabled

   1.4.    Ensure 'Application pool identity' is configured for all application pools

   1.5.    Ensure 'unique application pools' is set for sites

   1.6.    Ensure 'application pool identity' is configured for anonymous user identity

   1.7.    Ensure WebDav feature is disabled

2.       Configure Authentication and Authorization

   2.1.    Ensure 'global authorization rule' is set to restrict access

   2.2.    Ensure access to sensitive site features is restricted to authenticated principals only

   2.3.    Ensure 'forms authentication' requires SSL

   2.4.    Ensure 'forms authentication' is set to use cookies

   2.5.    Ensure 'cookie protection mode' is configured for forms authentication

   2.6.    Ensure transport layer security for 'basic authentication' is configured

   2.7.    Ensure 'passwordFormat' is not set to clear

   2.8.    Ensure 'credentials' are not stored in configuration files

3.       ASP.NET Configuration Recommendations

   3.1.    Ensure 'deployment method retail' is set

   3.2.    Ensure 'debug' is turned off

   3.3.    Ensure custom error messages are not off

   3.4.    Ensure IIS HTTP detailed errors are hidden from displaying remotely

   3.5.    Ensure ASP.NET stack tracing is not enabled

   3.6.    Ensure 'httpcookie' mode is configured for session state

   3.7.    Ensure 'cookies' are set with HttpOnly attribute

   3.8.    Ensure 'MachineKey validation method - .Net 3.5' is configured

   3.9.    Ensure 'MachineKey validation method - .Net 4.5' is configured

   3.10.  Ensure global .NET trust level is configured

   3.11.  Ensure X-Powered-By Header is removed

   3.12.  Ensure Server Header is removed

4.       Request Filtering and other Restriction Modules

   4.1.    Ensure 'maxAllowedContentLength' is configured

   4.2.    Ensure 'maxURL request filter' is configured

   4.3.    Ensure 'MaxQueryString request filter' is configured

   4.4.    Ensure non-ASCII characters in URLs are not allowed

   4.5.    Ensure Double-Encoded requests will be rejected

   4.6.    Ensure 'HTTP Trace Method' is disabled

   4.7.    Ensure Unlisted File Extensions are not allowed

   4.8.    Ensure Handler is not granted Write and Script/Execute

   4.9.    Ensure ‘notListedIsapisAllowed’ is set to false

   4.10.  Ensure ‘notListedCgisAllowed’ is set to false

   4.11.  Ensure ‘Dynamic IP Address Restrictions’ is enabled

5.       IIS Logging Recommendations

   5.1.    Ensure Default IIS web log location is moved

   5.2.    Ensure Advanced IIS logging is enabled

   5.3.    Ensure ‘ETW Logging’ is enabled

6.       FTP Requests

   6.1.    Ensure FTP requests are encrypted

   6.2.    Ensure FTP Logon attempt restrictions is enabled

7.       Transport Encryption

   7.1.    Ensure HSTS Header is set

   7.2.    Ensure SSLv2 is Disabled

   7.3.    Ensure SSLv3 is Disabled

   7.4.    Ensure TLS 1.0 is Disabled

   7.5.    Ensure TLS 1.1 is Disabled

   7.6.    Ensure TLS 1.2 is Enabled

   7.7.    Ensure NULL Cipher Suites is Disabled

   7.8.    Ensure DES Cipher Suites is Disabled

   7.9.    Ensure RC4 Cipher Suites is Disabled

   7.10.  Ensure AES 128/128 Cipher Suite is Disabled

   7.11.  Ensure AES 256/256 Cipher Suite is Enabled

   7.12.  Ensure TLS Cipher Suite Ordering is Configured

 

Table 1.2 provides a high level overview of the OWASP benchmarks
 
Table 1.2: OWASP IIS 10 Security Configuration Controls

1.1   Basic Configuration

   1.1.1         Disable directoryBrowsing

   1.1.2         Avoid wildcard host headers

   1.1.3         Ensure applicationPoolIdentity is configured for all application ppols

   1.1.4         Use an unique applicationPool per site

   1.1.5         Disable IIS detailed error page from displaying remotely

1.2   Request Filtering

   1.2.1         Configure maxAllowedContentLength

   1.2.2         Configure maxURL request filter

   1.2.3         Configure maxQueryString request filter

   1.2.4         Reject non-ASCII characters in URLs

   1.2.5         Reject double-encoded requests

   1.2.6         Disable HTTP trace requests

   1.2.7         Disallow unlisted file extensions

   1.2.8         Enable Dynamic IP Address Restrictions

1.3   Transport Encryption

   1.3.1         SSL/TLS settings are controlled at the SChannel level. They are set machine wide and IIS respects these values

   1.3.2         A list of recommendations for IIS

   1.3.2.1    Disable SSL v2/v3

   1.3.2.2    Disable TLS 1.0

   1.3.2.3    Disable TLS 1.1

   1.3.2.4    Ensure TLS 1.2 is enabled

   1.3.2.5    Disable weak cipher suites (NULL cipher suites, DES cipher suites, RC4 cipher suites, Triple DES, etc)

   1.3.2.6    Ensure TLS cipher suites are correctly ordered

1.4   HSTS support

   1.4.1         IIS recently (Windows Server 1709+) added turnkey support for HSTS

1.5   CORS support

   1.5.1         Implement OWASP IIS CORS configuration module if your application does not natively handle CORS.

 

When hardening IIS, review each control and determine its appropriateness to your existing deployment. With any hardening strategy, you need to be incremental in your approach, applying and testing each new security control in a development or test environment before deploying it into a production environment. As wonderful as it is to have a secure deployment, it’s not so wonderful if the web application your IIS server is hosting no longer works because you made everything a little too secure.

**Tài liệu tham khảo**

- [microsoft](https://techcommunity.microsoft.com/t5/itops-talk-blog/windows-server-101-hardening-iis-via-security-control/ba-p/329979)
- [calcomsoftware](https://www.calcomsoftware.com/hardening-iis-server-guide/)

---
