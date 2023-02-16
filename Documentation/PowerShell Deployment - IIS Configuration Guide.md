# IIS Configuration Guide

## Introduction: 
In order to support OS deployments via HTTPS a series of configuration items must be completed. This document highlights and outlines the settings and steps known to work with PSD including steps for the server side logging feature (available in PSD version 0.2.2.8 and later).

> NOTE: Your security team and environment may require additional settings or lock down.

## Operating System Requirements for imaging via HTTP/HTTPS
We have tested the IIS Setup for PSD on the following server operating systems

       Windows Server 2016
       Windows Server 2019
       Windows Server 2022

## Install IIS and configure WebDAV
To install IIS and configure WebDAV for OSD you need to run two scripts, one for setup, and one for configuration, with a reboot in between.

To run the IIS Setup, run the first script (New-PSDWebInstance.ps1) without any parameters, and after completion, reboot the server. The New-PSDWebInstance.ps1 script is found in the Tools folder of PSD.

> NOTE: The IIS Setup script does currently NOT support a server that already has IIS installed, it has to be run on a clean Windows Server installation.

Then, to run the configuration, you run the second script (Set-PSDWebInstance.ps1), specifying your deployment folder, and the name of the virtual directory to create. The Set-PSDWebInstance.ps1 script is also located in the Tools folder of PSD. Sample syntax:

.\Set-PSDWebInstance.ps1 -psDeploymentFolder E:\PSDProduction -psVirtualDirectory PSDProduction 

## HTTPS and Certificates
To support imaging via HTTPS you need to install a proper web server certificate, and make sure the Root CA is added to WinPE. If you export your Root CA certificate to the PSDResources\Certificates folder, PSD will automatically add it to WinPE when updating the deployment share.

For lab purposes, we provide two scripts to create a self-signed certificate for your deployment server. The first script (New-PSDRootCACert.ps1) creates a local Root CA and exports the Root CA to the PSDResources\Certificates folder. The second script (New-PSDServerCert.ps1) creates a self-signed certificate for the deployment server. 

**Note:** These two scripts are replacing the New-PSDSelfSignedCert.ps1 script available in the original release of PSD.

Sample syntax for New-PSDRootCACert.ps1 script:
.\New-PSDRootCACert.ps1 -RootCAName PSDRootCA -ValidityPeriod 20 -psDeploymentFolder E:\PSDProduction

Sample syntax for the New-PSDServerCert.ps1 script:

.\New-PSDServerCert.ps1 -DNSName mdt01.corp.viamonstra.com -FriendlyName mdt01.corp.viamonstra.com -ValidityPeriod 5 -RootCACertFriendlyName PSDRootCA

## Server Side logging via BITS Upload
To support server side logging via BITS Upload, IIS need to be configured to allow that. To create a BITS Upload folder and virtual directory, run the Set-PSDLogInstance.ps1 script. Sample syntax:

.\Set-PSDWebInstance.ps1 -psDeploymentFolder E:\PSDProductionLogs -psVirtualDirectory PSDProductionLogs

In addition the following rules must be added to the CustomSettings.ini file:

LogUserDomain=ServernameOrDomain
LogUserID=AccountName
LogUserPassword=Password
SLShare=https://mdt01.corp.viamonstra.com/PSDProductionLogs

## Firewall Ports
In addition to the IIS setup and configuration the following firewall ports needs to open: 

* Port 443 for HTTPS
* Port 9800 and 9801 for MDT Event Monitoring (optional, disabled by default)

## IIS Setup Reference 
In this section you find a list of all components being added by the setup script as well as info on what the configuration script does.

* **IIS Components**

The following IIS components are required to ensure that PSD functions as expected for imaging via HTTP/HTTPS. This components are all available in the web server role.

* Common HTTP Features

     * Default Document
     * Directory Browsing
     * HTTP Errors
     * Static Content
     * HTTP Redirection
     * WebDav Publishing

* Health and Diagnostics

     * HTTP Logging
     * Custom Logging
     * Logging Tools
     * Request Monitor
     * Tracing
     
* Performance

     * Static Content Compression

* Security 

     * Request Filtering
     * Basic Authentication
     * Digest Authentication
     * URL Authorization
     * Windows Authentication
     
* Management Tools
     * IIS Management Console 
     * IIS 6 Management Compatibility
          * IIS 6 Metabase Compatability



* **Configure IIS**

 For IIS, PSD requires some configuration changes in order to function. Most of these changes have to do with configuring IIS to work properly with WebDav. If you use the configuration script these will be automatically configured for you, but shorthand, the following needs to be done:

     * Create New Virtual Directory
     * Enable Directory Browsing
     * Disable Anonymous Authentication
     * Enable Windows Authentication
     * Create and add new MIME type 
     
* **Configure WebDAV**

Finally, PSD requires some configuration changes to WebDAV, and again, if you use the configuration script these will be automatically configured for you. Shorthand, the following needs to be done:

     * Enable WebDAV
     * Create new WebDav Authoring Rule
     * Modify WebDAV Settings
         * Allow File Extension Filtering
         * Allow Hidden Segment Filtering
         * Allow Verb Filtering
     * Modify Default MIME type

