---
copyright:
  years: 2017, 2020
lastupdated: "2019-12-17"

subcollection: mqcloud

keywords: admin, administration, runmqsc, SSL, TLS
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Securing remote administration using runmqsc
{: #mqoc_remote_ssl_runmqsc_admin}

This document describes the process of enabling TLS for remote administration of an MQ on Cloud queue manager using the *runmqsc command line interface*.

## Prerequisites
{: #mqoc_remote_ssl_runmqsc_admin_prereq}

1. To establish a secure connection to MQ on Cloud queue manager, you must first configure TLS security on the MQ channel
  - For details on how to do this please refer to [Enabling TLS security for MQ channels in MQ on Cloud](/docs/services/mqcloud?topic=mqcloud-mqoc_configure_chl_ssl)  
  - Any queue manager at version 9.2.2 or newer that was created after the release date for version 9.2.2 will have TLS enabled by default.
  - For the purposes of this topic we assume that the CLOUD.ADMIN.SVRCONN channel has been configured with the cipherspec `ANY_TLS12_OR_HIGHER`, and SSL Authentication (SSLCAUTH) as Optional
  - An additional section at the end of this topic describes how to extend the TLS configuration to support mutual TLS authentication of the runmqsc client
2. This topic describes using a JSON format CCDT to tell runmqsc how to connect to the queue manager
  - To use JSON format CCDT you must have a runmqsc installation from IBM MQ v9.1.2 or above. Runmqsc versions prior to this do not support JSON format CCDT
  - You can also connect runmqsc to a queue manager using a binary CCDT if you wish, but the instructions provided here are specific to JSON CCDT
3. To configure the client keystore file you will need tools such as runmqakm
  - These tools are available in either a full installation of IBM MQ, or an installation of the MQ client for your operating system
  - Both full and client installations are available from the [MQ Downloads](https://ibm.biz/MQdownloads) page
  - There is also a [MacOS toolkit for Developers](https://developer.ibm.com/messaging/2019/02/05/ibm-mq-macos-toolkit-for-developers/) client that allows native use of runmqsc on MacOS


## Configuring runmqsc to use anonymous ("one way") TLS connections
{: #mqoc_remote_ssl_runmqsc_admin_tasks}

1. Download or create a JSON CCDT file that describes the queue manager you want to connect to;

  You can download a JSON CCDT from the queue manager details page by clicking on the "Connection information" button and then selecting the "JSON CCDT" format. Note that the downloaded JSON CCDT does not contain the cipher specification that you associated with the channel, so you have to include that manually by adding a transmissionSecurity definition for each channel as shown in the example below.

  Alternatively you can simply copy the example template provided here and update the host, port, queueManager and channel values to match your queue manager.
  ```
       {
         "channel": [
          {
           "name": "CLOUD.ADMIN.SVRCONN",
           "clientConnection": {
             "connection": [
             {
              "host": "qm1-1234.qm.us-south.mq.appdomain.cloud",
              "port": 31500
             }
             ],
             "queueManager": "QM1"
           },
           "transmissionSecurity": {
             "cipherSpecification": "ANY_TLS12"
           },
           "type": "clientConnection"
          }
          ]
       }

  ```
  **Note:** The cipher specification in the JSON CCDT example above has been specified as ANY_TLS12, which allows the runmqsc client to negotiate a permitted cipher with the queue manager within the family of TLS v1.2 ciphers, based on what ciphers the channel is configured to permit. For maximum flexibility the queue manager channel can also be configured as ANY_TLS12 which allows the client and server to negotiate between them.

2. Configure the necessary environment variables for the runmqsc environment;

  We will configure environment variables to instruct the runmqsc client to use the JSON CCDT to obtain the queue manager connection details, which includes the name of the channel cipher spec that we defined in the previous step.

  2.1 Open a command prompt window and navigate to the MQ "bin" directory of your installation.
    The location of this will depend on your operating system, for example;

    ```
      /var/mqm/bin  (Linux full MQ installation)
      C:\Program Files\IBM\MQ\bin  (Windows)
      ~/mytoolkit/IBM-MQ-Toolkit-Mac-x64-9.1.2.0/bin   (Linux/Mac, using the client installation)
    ```

  2.2 Set the MQCCDTURL environment variable to instruct the runmqsc client to read the JSON CCDT;
    The MQCCDTURL variable is a URL pointing to the JSON CCDT file.

    Notes:
      - It is important that you *MUST NOT* have the MQSERVER variable set, otherwise that will take precedence over the settings from the JSON CCDT
      - If you are using a binary CCDT file then set the MQCHLTAB variable instead of the MQCCDTURL
      - Please be aware that the MQCHLTAB variable does not support use of a JSON formatted CCDT

    ```
      # Linux/MacOS
      export MQCCDTURL=file:///Users/myuser/connection_info_ccdt.json    
      unset MQSERVER

      # Windows
      set MQCCDTURL=file:///c:/temp/connection_info_ccdt.json    
      set MQSERVER=        
    ```

  2.3 Set the MQSSLKEYR environment variable to allow the runmqsc client to trust the TLS certificate presented by the queue manager

    The MQSSLKEYR variable must be set to the full path from the system root to a keystore file that contains the TLS certificate presented by the queue manager.

    If you do not have already have a keystore file then you can create one by following the [Create a keystore](/docs/services/mqcloud?topic=mqcloud-mqoc_configure_chl_ssl#mqoc_chl_ssl_keystore) topic, which includes downloading the public certificate for the queue manager from the service console user interface.

    Note that the ".kdb" suffix of the file must NOT be included in the MQSSLKEYR value - so for a key store named "key.kdb", specify just "key".  

    ```
      # Linux/MacOS
      export MQSSLKEYR=/Users/myuser/key

      # Windows
      set MQSSLKEYR=c:\temp\key
    ```

3. Execute the **runmqsc** command to connect to the queue manager;

  Now that we have set up the necessary environment we can execute the runmqsc command to connect to the queue manager.

  As normal, you will need the following inputs to the runmqsc command;
  - MQ Administrator username and API key, which you obtain from the service console as described in [Gather required connection details](/docs/services/mqcloud?topic=mqcloud-mqoc_admin_mqcli#getdetails_mqoc_admin_mqexp). Note that this must an Administrator username and API key, and not an Application username and API key
  - Queue manager name, which must match what is specified in the JSON CCDT file - for example "QM1"
  ```
     <Path to MQ/bin directory>/runmqsc -c -u mqusername QM1
  ```

  All the operations on this runmqsc will now run on a secured channel.


## Configuring runmqsc to use mutual ("two way") TLS connections
The following steps describe how to extend the instructions above to configure mutual TLS connectivity between the runmqsc client and the queue manager.

In mutual TLS scenarios the client (e.g. runmqsc) presents a client certificate to the queue manager, and the queue manager must be configured to trust that incoming client certificate.

1. Generate a TLS client certificate that will identify the runmqsc client;

  In some organizations TLS client certificates are generated for you by a central certificate authority (CA). For the purposes of this example we will generate our own self-signed client certificate.

  ```
  # Generate a new private key and public certificate for the client
  # (fill in the segments of the certificate as you wish when prompted)
  openssl req -newkey rsa:2048 -nodes -keyout clientKey.pem -x509 -days 365 -out clientCert.pem

  # Combine the private key and public certificate into a single file
  cat clientKey.pem > clientCombined.pem
  cat clientCert.pem >> clientCombined.pem
  ```

2. Add the newly generated client certificate to the local keystore file;

  Import the combined client key/cert into the keystore file, making a note of the label that you use.

  ```
  runmqakm -cert -add -db key.kdb -file clientCombined.pem -label runmqsc -stashed -type pkcs12 -format ascii
  ```

3. Configure the queue manager to trust the client certificate;

  - Import the public part of the client certificate by navigating to the queue manager details page in the MQ on Cloud service console user interface, and selecting the "Trust store" tab
  - Click the "Import certificate" button, and select the file containing the public part of the client certificate, for example `clientCert.pem`
  - Click Next and pick a label for the certificate (it doesn't have to be the same label as in the client keystore file), then click Save
  - Once the certificate has been uploaded you will be requested to refresh the queue manager SSL security configuration which you can do using MQ Console, MQ Explorer or runmqsc as [described here](/docs/services/mqcloud?topic=mqcloud-mqoc_refresh_security)

4. Configure the queue manager channel to require mutual TLS;

  The TLS mode for a channel is controlled by the "SSL Authentication" attribute of the channel, which must be set to "Required" to enable mutual TLS. You can set the attribute using MQ Console or MQ Explorer via the UI, or if you are using runmqsc then you must set the attribute "SSLCAUTH" to "REQUIRED".

5. Configure runmqsc to present the client certificate when it connects to the queue manager;

  Set the "MQCERTLABL" environment variable to the label of the client certificate in the local keystore file, and then you can execute the runmqsc command to connect to the queue manager, which this time will be configured using mutual TLS authentication.

  ```
  export MQCERTLABL=runmqsc
  runmqsc -c -u mqusername QM1
  ```

## Next steps
{: #mqoc_remote_ssl_runmqsc_admin_next}

If you want to connect applications to a queue manager that has TLS enabled on the channels then you can follow the instructions in the following topic;
* [Connect securely from C MQI & JMS application](/docs/services/mqcloud?topic=mqcloud-mqoc_connect_app_ssl)
