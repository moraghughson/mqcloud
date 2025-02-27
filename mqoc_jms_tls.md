---
copyright:
  years: 2017, 2020
lastupdated: "2018-08-06"

subcollection: mqcloud

keywords: JMS, SSL, TLS, client
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Enabling TLS between a client and a queue manager
{: #mqoc_jms_tls}

This guide details how to enable standard and mutual TLS authentication between a queue manager and a client application. This will allow messages to be sent and received securely over the network.

![Image of TLS diagram](./images/mqoc_tls_diagram.png)

Standard TLS ensures the client trusts the server that it is communicating with. Mutual TLS involves the server ensuring it trusts the client too. TLS makes use of public key cryptography to encrypt messages over the network. Public keys belonging to trusted servers and applications are stored in a trust store. Applications and servers store their own own private key in a key store.

---

## Prerequisites
{: #prereq_mqoc_jms_tls}

In order to proceed with this tutorial, it is vital that you have the following prerequisites:

1. **IBM MQ on Cloud queue manager**

  If you do not already have an IBM MQ on Cloud queue manager, you can create one by following the guided tour here:

  [Getting started with IBM MQ on Cloud](https://cloud.ibm.com/docs/services/mqcloud?topic=mqcloud-mqoc_getting_started)

  Having followed the guided tour, or the manual steps provided on the same page, you should have:
    - An MQ on Cloud queue manager
    - Connection details downloaded in a connection_info.txt file
    - An admin username and apikey downloaded in a platformApiKey.json file
    - An application username and apikey downloaded in an apiKey.json file

  You can consult **Appendix 1** at the end of this tutorial for how to get these files if you don't have them.

2. **A JMS application**

  This tutorial requires a JMS application to successfully complete it. We recommend you follow the tutorial detailed in **Appendix 2** in order to ensure you have a suitable JMS application for this tutorial.

3. **IBM's JDK**

  You will need a Java JDK to complete this tutorial. It is recommended that you use the IBM JDK. This JDK contains support for modern encryption types and higher levels of security, which are needed for this tutorial. You will make use of `java`, `javac` and `ikeycmd` tools. If you do not have these commands, follow the steps in **Appendix 3** at the end of this tutorial to download and install the IBM JDK.

4. **IBM MQ Client**

  To complete this tutorial you will need the IBM MQ command line tool *runmqsc* installed and exported to your PATH. If you do not have this command, you can get it by installing the IBM MQ Client. **Appendix 4** at the end of this tutorial details how to do this.

---

## Securing Applications with Standard (One Way) TLS
{: #oneway_mqoc_jms_tls}

Standard TLS authentication ensures that a client can only talk to trusted queue managers, while also encrypting the messages travelling between the queue manager to the client.

In this section of the document, you will first demonstrate an existing, un-secure connection between a client and a queue manager, and then transform it into a TLS enabled, secure connection.

After enabling TLS on the queue manager's channel, the JMS application will need to be configured to trust the queue manager before it can communicate with it. First you will export the public part of the queue manager's security certificate. This part of the certificate is known as a public key. MQ on Cloud comes with a default security certificate, provided by *DigiCert*. The JMS application will store the queue managers public key in a trust store. You will therefore create a trust store, and import the queue managers public key into it. The JMS application will then be updated to use the trust store.

1. **Run unencrypted JMS application**

  **Note.** You will have already run the JMS application if you have completed the prerequisites

  The JMS application must be run to confirm that is runs, though the messages are not encrypted. Use the following commands:

  - Recompile:
    - On Windows:
      - `javac -cp .\com.ibm.mq.allclient-9.0.4.0.jar;.\javax.jms-api-2.0.1.jar com\ibm\mq\samples\jms\JmsPutGet.java`
    - On Linux:
      - `javac -cp ./com.ibm.mq.allclient-9.0.4.0.jar:./javax.jms-api-2.0.1.jar com/ibm/mq/samples/jms/JmsPutGet.java`

  - Run:
    - On Windows:
      - `java -cp .\com.ibm.mq.allclient-9.0.4.0.jar;.\javax.jms-api-2.0.1.jar;. com.ibm.mq.samples.jms.JmsPutGet`
    - On Linux:
      - `java -cp ./com.ibm.mq.allclient-9.0.4.0.jar:./javax.jms-api-2.0.1.jar:. com.ibm.mq.samples.jms.JmsPutGet`

  This will be the expected output:

    ```
      Your lucky number today is 283
      SUCCESS
    ```

2. **Connect to queue manager**

  TLS communication must be enabled on the queue manager. You will use the command *runmqsc* in order to connect to the remote MQ on Cloud queue manager. *runmqsc* uses an environment variable to determine the details of the queue manager to connect to.

  Export the MQ variable with the following command:

- `export MQSERVER="CLOUD.ADMIN.SVRCONN/TCP/<HOSTNAME>(<PORT>)"`
  - `<HOSTNAME>` - this is '*hostname*' in the file connection_info.txt
  - `<PORT>` - this is '*listenerPort*' in the file connection_info.txt

3. **Using 'runmqsc'**

  Enter the following command into the console window:

- `runmqsc -c -u <ADMIN_MQ_USER> -w60 <QUEUE_MANAGER_NAME>`
  - `<ADMIN_MQ_USER>` - this is '*mqUsername*' in the file platformApiKey.json
  - `<QUEUE_MANAGER_NAME>` - this is '*queueManagerName*' in the file connection_info.txt
  - `-c` informs *runmqsc* it should connect to a remote queue manager using the *MQSERVER* variable

  - runmqsc will prompt you for a password - this is the api key in the *PlatformApiKey.json* file you downloaded as part of the guided tour.

  *runmqsc* should show that authentication was successful. The terminal will output a blank line and wait for user input.

4. **Configuring the queue manager for TLS**

Having connected to the queue manager, one way TLS communication can be configured.

- Check the current status of the queue manager with the following instruction:

`DISPLAY CHANNEL(CLOUD.APP.SVRCONN)`

This command displays the current settings of the channel you are about to change.

  - Update the channel to enable standard TLS by setting *SSLCIPH* to 'ECDHE_RSA_AES_128_CBC_SHA256' and *SSLCAUTH* to 'OPTIONAL'. 'OPTIONAL' will disable the use of mutual authentication, and a combination of OPTIONAL with the setting of Cipher Spec will enable standard TLS. Run the command:

  `ALTER CHL('CLOUD.APP.SVRCONN') CHLTYPE(SVRCONN) SSLCIPH(ECDHE_RSA_AES_128_CBC_SHA256) SSLCAUTH(OPTIONAL)`

  - Use the following command to ensure that the channel settings have changed:

  `DISPLAY CHANNEL(CLOUD.APP.SVRCONN)`

  - Exit *runmqsc* with:

  `end`

5. **Rerun the JMS application**

  Now run JMS application again. This time it should fail because the queue manager is enforcing TLS, but the client application is not yet configured to use TLS.

  - On Windows:
    - `java -cp .\com.ibm.mq.allclient-9.0.4.0.jar;.\javax.jms-api-2.0.1.jar;. com.ibm.mq.samples.jms.JmsPutGet`

  - On Linux:
    - `java -cp ./com.ibm.mq.allclient-9.0.4.0.jar:./javax.jms-api-2.0.1.jar:. com.ibm.mq.samples.jms.JmsPutGet`

  This is the expected error shown in the last Java backtrace:

  ```
    Caused by: com.ibm.mq.jmqi.jmqiException: CC=2;RC=2397;AMQ9641: Remote CipherSpec error for channel 'CLOUD.APP.SVRCONN' to  host ''. [3=CLOUD.APP.SVRCONN]
  ```

6. **Export the queue manager's public key certificate**

  The JMS application needs to trust the queue manager for one way TLS. The public key must first be exported from the server so that it can be imported into the application trust store and used by the JMS application. Follow these steps:

  - Click on the queue manager being configured - this will display more details about this queue manager.

  - Click on 'Key store'. This is where the security certificates on the queue manager are stored.

  ![Image of queue manager key store](./images/mqoc_tls_qmkeystore.png)

  - The default security certificate labelled 'Default: qmgrcert' can be seen:

  ![Image of security certificate](./images/mqoc_tls_qmcert.png)

  - The public part of the certificate must be downloaded. Click on the three dots in the upper right of the certificate:

  - This will bring up a toolbar. Click on 'Download public certificate' to download the public key of the certificate.

  ![Image of download certificate](./images/mqoc_tls_qmdowncert.png)

7. **Create a trust store**

  The following command creates a key database our application can use as a trust store. Navigate to the same folder as the JMS application and execute this command:

- `ikeycmd -keydb -create -db trust -pw <trustStorePassword> -type jks -expire 0`
  - `<trustStorePassword>` - choose a password and remember it for future use.

8. **Import the queue manager's public key into the trust store**

  Enter the following command to add the certificate to our new trust store:

- `ikeycmd -cert -add -db trust.jks -file <path_to_cert>/qmgrcert.pem -label DigiCertRootCA -pw <trustStorePassword>`
  - `<trustStorePassword>` - this is the password created in the previous step.
  - `<path_to_cert>` - this is the location you downloaded the queue managers public key to

  A key database has now successfully been created to use as a trust store, and the queue manager's public key has been imported to our new trust store.

9. **Configure the JMS application to use the trust store for TLS**

  The JMS application now needs to be configured to use the new trust store. Open the JMS application with a text editor, and navigate to the following section of code:

  ```
    try {
    // Create a connection factory
    JmsFactoryFactory ff = JMSFactoryFactory.getInstance(WMQConstants.WMQ_PROVIDER);
    JmsConnectionFactory cf = ff.createConnectionFactory();
  ```

  Insert the following immediately after the code shown above:

  ```
    cf.setStringProperty(WMQConstants.WMQ_SSL_CIPHER_SPEC,"ECDHE_RSA_AES_128_CBC_SHA256");
    cf.setStringProperty(WMQConstants.WMQ_CHANNEL, "CLOUD.APP.SVRCONN");
    cf.setIntProperty(WMQConstants.WMQ_CONNECTION_MODE, WMQConstants.WMQ_CM_CLIENT);
    System.setProperty("javax.net.ssl.trustStore", "trust.jks");
    System.setProperty("javax.net.ssl.trustStorePassword", "<trustStorePassword>");
  ```

  - `<trustStorePassword>` - this is the same password created to protect the key database.

10. **Rerun the JMS application**

  Once these changes have been made, save the program and re-compile it and follow by running the following command:

  - On Windows:
    - `javac -cp .\com.ibm.mq.allclient-9.0.4.0.jar;.\javax.jms-api-2.0.1.jar com\ibm\mq\samples\jms\JmsPutGet.java`

  - On Linux:
    - `javac -cp ./com.ibm.mq.allclient-9.0.4.0.jar:./javax.jms-api-2.0.1.jar com/ibm/mq/samples/jms/JmsPutGet.java`

  Run the program again using the commands listed below:

  - On Windows:
    - `java -cp .\com.ibm.mq.allclient-9.0.4.0.jar;.\javax.jms-api-2.0.1.jar;. com.ibm.mq.samples.jms.JmsPutGet`

  - On Linux:
    - `java -cp ./com.ibm.mq.allclient-9.0.4.0.jar:./javax.jms-api-2.0.1.jar:. com.ibm.mq.samples.jms.JmsPutGet`

  This time, the program should be able to successfully 'put' and 'get' messages from the queue manager.

  The messages travelling over the network are now encrypted and the client now trusts the server.

---

## Securing Applications with Mutual (Two Way) TLS
{: #twoway_mqoc_jms_tls}

Standard TLS is commonly used when connecting to servers or websites. Using standard TLS, a client can be sure that it is connecting to the trusted server. In addition, mutual TLS ensures that the server only accepts connections from trusted clients. This is again achieved through the use of security certificates.

Before starting this section, you should have completed the Standard TLS section above. The JMS client application should currently be communicating with a queue manager using standard TLS authentication. You will now enable mutual TLS.

1. **Connect to the queue manager**

  First, the queue manager needs to be configured and mutual authentication must be enforced. Run the following command to connect to the queue manager:

- `runmqsc -c -u <ADMIN_MQ_USER> -w60 <QUEUE_MANAGER_NAME>`

  - `<ADMIN_MQ_USER>` - this is '*mqUsername*' in the file platformApiKey.json
  - `<QUEUE_MANAGER_NAME>` - this is '*queueManagerName*' in the file connection_info.txt

2. **Configure the queue manager**

  Enter the command below. Notice now that *SSLCAUTH* is changed to 'REQUIRED'. This is configuring the queue manager to only accept communication from trusted clients:

  `ALTER CHL('CLOUD.APP.SVRCONN') CHLTYPE(SVRCONN) SSLCIPH(ECDHE_RSA_AES_128_CBC_SHA256) SSLCAUTH(REQUIRED)`

3. **Rerun the JMS application**

  Rerun the JMS application. The application will fail as the queue manager is configured for mutual TLS, but the client is still configured for standard TLS. Run the JMS application with the following command:

  - On Windows:

  `java -cp .\com.ibm.mq.allclient-9.0.4.0.jar;.\javax.jms-api-2.0.1.jar;. com.ibm.mq.samples.jms.JmsPutGet`

  - On Linux:

  `java -cp ./com.ibm.mq.allclient-9.0.4.0.jar:./javax.jms-api-2.0.1.jar:. com.ibm.mq.samples.jms.JmsPutGet`

4. **Create the keystore**

  A key store must now be created to store a private certificate for the JMS application to use. Navigate to the same folder as the JMS application and execute the following command:

  - `ikeycmd -keydb -create -db key -pw <keyStorePassword> -type jks -expire 0`
    - `<keyStorePassword>` - choose a password and remember it for future use.

  A key database has now successfully been created to use as a key store.

  Confirm a new file can be seen called *key.jks* in the file system:

  `ls`

5. **Create a self signed public/private key pair for JMS application**

  Create a self-signed certificate for the client to use:

  - `ikeycmd -cert -create -db key.jks -pw <keyStorePassword> -sig_alg SHA256WithRSA -label clientcert -dn "O=<Your Organisation>, C=<Your Country>"`
    - `<keyStorePassword>` - pick a password for your key store file
    - `<Your Organisation>` - your company may have specified a particular value to use, otherwise use your own input
    - `<Your Country>` - your company may have specified a particular value to use, otherwise use your own input

6. **Extract the self signed public key**

  Extract the public key of the security certificate from the key store:

  - `ikeycmd -cert -extract -db key.jks -pw <keyStorePassword> -label clientcert -target clientcert.pem`
    - `<keyStorePassword>` - this is the password entered above to protect the key store.

7. **Configure the JMS application to use a key store for mutual TLS**

  Open the JMS application with a preferred text editor, and navigate to the following section of code:

  ```
    try {
    // Create a connection factory
    JmsFactoryFactory ff = JMSFactoryFactory.getInstance(WMQConstants.WMQ_PROVIDER);
    JmsConnectionFactory cf = ff.createConnectionFactory();

    ...
  ```

  Insert the following code immediately after the code shown above:

  ```
    System.setProperty("javax.net.ssl.keyStore", "key.jks");
    System.setProperty("javax.net.ssl.keyStorePassword", "<keyStorePassword>");
  ```

  - `<keyStorePassword>` - this is the password created to protect the key store.

8. **Import the JMS public key into a queue manager trust store**

  In order for the queue manager to trust the client, the next step is to import the public part of the JMS client’s security certificate into the queue manager’s trust store; This action ensures that the queue manager trusts the client. This can be done from the web user interface:

  - Click on the queue manager name from the available list:

  ![Image showing the list of queue managers](./images/mqoc_tls_qmview.png)

  - Click on the 'Trust store' tab as shown below:

  ![Image of queue manager details](./images/mqoc_tls_qmtruststore.png)

  - Click the 'Import certificate' button:

  ![Image showing the import certificate option](./images/mqoc_tls_qmimportcert.png)

  - In the window that opens, browse for the public certificate extracted in the earlier step.

  - In order for the queue manager to refresh its security cache and pick up these changes, refresh the SSL security on the queue manager. Using the *runmqsc* command to connect to the queue manager, execute the following:

  **NOTE: This will end all connections to this queue manager, so consider the most appropriate time to use this command**

  `REFRESH SECURITY TYPE(SSL)`

  - The console will output the following message as a notification of the change:

  ```
    AMQ8560I: IBM MQ security cache refreshed.
  ```

  - When this operation has finished, disconnect runmqsc from the queue manager by typing:

  `end`

  The queue manager has now been configured to enforce mutual authentication and the client’s public key has been added to the queue manager’s trust store. Therefore, the client application and server will now be able to communicate through encrypted channels.

9. **Rerun the JMS application**

  To demonstrate mutual TLS has been correctly configured between the client and the server, recompile and run the program using the same Java commands as usual:

  - Recompile:
    - On Windows:
      - `javac -cp .\com.ibm.mq.allclient-9.0.4.0.jar;.\javax.jms-api-2.0.1.jar com\ibm\mq\samples\jms\JmsPutGet.java`

    - On Linux:
      - `javac -cp ./com.ibm.mq.allclient-9.0.4.0.jar:./javax.jms-api-2.0.1.jar com/ibm/mq/samples/jms/JmsPutGet.java`

  - Run:
    - On Windows:
      - `java -cp .\com.ibm.mq.allclient-9.0.4.0.jar;.\javax.jms-api-2.0.1.jar;. com.ibm.mq.samples.jms.JmsPutGet`

    - On Linux:
      - `java -cp ./com.ibm.mq.allclient-9.0.4.0.jar:./javax.jms-api-2.0.1.jar:. com.ibm.mq.samples.jms.JmsPutGet`

  The tutorial has successfully been completed.

---

## APPENDIX
{: #appendix_mqoc_jms_tls}

### Appendix 1: Downloading the IBM MQ on Cloud files

  This section describes how to gather the various files that would have been downloaded during the guided tour, but which might have since been lost.

#### Connection_info.json
To retrieve the connection_info.txt file containing queue manager connection details:
  - Login to the IBM Cloud service instance by clicking on the relevant service shown in the table:
  ![Image of service instance](./images/mqoc_tls_si1.png)
  - This will open the queue manger view:
  ![Image of queue manager details](./images/mqoc_tls_qmview.png)
  - Select the queue manager to be configured to use TLS on to open the next window. Select the 'Connection information' button:
  ![Image of queue manager connection information](./images/mqoc_tls_qmconinfo.png)
  - Download this file in the format of 'Plain text for human use'

#### PlatformApiKey.json
To create or reset your administrator api key:
  1. Login to the IBM Cloud service instance by clicking on the relevant service shown in the table
  ![Image showing service instance](./images/mqoc_tls_si1.png)
  2. This will open the queue manager view. Select the queue manager you wish to retrieve the connection info from
  ![Image showing list of queue managers](./images/mqoc_tls_qmview.png)
  3. Next, select the **Administration** tab
  ![Image showing queue manager Administration tab highlighted](./images/mqoc_ams_administration_select.png)
  4. Now click the **Reset IBM Cloud API Key**
    - **Note:** The previous admin API key for this MQ Username will **no longer be valid**

  ![Image showing administration API key reset button highlighted](./images/mqoc_ams_admin_reset.png)
    - **Note:** If the button says **Create IBM Cloud API Key**, then you have not created an api key in this way before. Click the **Create IBM Cloud API Key** button.
  5. Click **Download** to download platformApiKey.json containing an admin username and apikey
  ![Image showing the Download button for the admin new API key highlighted](./images/mqoc_ams_admin_download.png)

#### apiKey.json
To create a new application api key:
  1. Login to the IBM Cloud service instance by clicking on the relevant service shown in the table
  ![Image showing service instance](./images/mqoc_tls_si1.png)
  2. This will open the queue manager view. Select the **Application permissions** tab
  ![Image showing list of queue managers with Application permissions circled](./images/mqoc_ams_application_select.png)
  3. Now click the 3 dots next to the application you will use for the AMS tutorial
  ![Image showing list of application permissions currently set](./images/mqoc_ams_application_tab.png)
  4. Click **Add new API key**
    - Note. Existing app API keys will continue to work

  ![Image showing Add new API key circled for an application](./images/mqoc_ams_app_new_apikey.png)
  5. Click **Download** to download apiKey.json containing an app username and apikey
  ![Image showing the Download button for the app new API key highlighted](./images/mqoc_ams_app_download.png)

### Appendix 2: JMS Application tutorial

Use the tutorial provided at the link below to create a JMS application which can connect to a queue manager and send and receive simple messages.

Several variables will need changing from the tutorial in order to configure the JMS application to connect to the on cloud queue manager you are using for this TLS tutorial. The values that need changing in the JMS application are below:

**NOTE** - most of the values required can be found in the *connection_info.txt* file; this should have been downloaded by following the guided tour. If you do not have the file, consult Appendix 1.

**NOTE** - the guided tour details how to create the ‘APP_USER’ and ‘APP_PASSWORD’ variables and download them in a file called ‘apiKey.json’, the ‘APP_PASSWORD’ refers to the API key. If you do not have this file, consult Appendix 1.

- HOST         = listed as ‘*Hostname:*’ in the ‘connection_info.txt’ file
- PORT         = listed as ‘*Listener port:*’ in the ‘connection_info.txt’ file
- CHANNEL      = change this value to: ‘CLOUD.APP.SVRCONN’
- QMGR         = listed as ‘*Queue manager name:*’ in the ‘connection_info.txt’ file
- APP_USER     = listed as ‘*mqUsername:*’ in the ‘apiKey.json’ file
- APP_PASSWORD = listed as ‘*apiKey:*’ in the ‘apiKey.json’ file
- QUEUE_NAME   = do NOT change this value

To create the basic, unsecured JMS application follow:

  https://developer.ibm.com/components/ibm-mq/tutorials/mq-develop-mq-jms

### Appendix 3: **Downloading and installing the IBM SDK**

  - Download JDK

  You should download the IBM JDK from [here](https://www.ibm.com/support/pages/java-sdk-downloads-version-80). Select the right download for your platform and install.
  Depending on which package you downloaded you need to run it to install. For example, if you downloaded ibm-java-x86_64-sdk-8.0-5.17.bin, run:

  `./ibm-java-x86_64-sdk-8.0-5.17.bin`

  For Windows you need to download the Eclipse package that contains the JDK.

  - Add to PATH

  Ensure that the JDK is on the system path:

  On Windows:

  `set PATH=C:\IBM_Eclipse\eclipseDevelopmentPackage\ibm_sdk80\bin;%PATH%`

  On Linux

  `export PATH=<install location>/ibm-java-x86_64-80/jre/bin:<install location>/ibm-java-x86_64-80/bin:$PATH`

### Appendix 4: ** Downloading the IBM MQ Client**
The IBM client is required to connect to the queue manager using *runmqsc*.

If you do not have the IBM MQ Client command line tool you can download it from [here](http://www-01.ibm.com/support/docview.wss?uid=swg24042176#1)

1. Select the latest package as shown below, the latest version at time of writing being 9.0.5
![Image showing IBM MQ client versions](./images/mqoc_ams_mqclient1.png)
2. Select the 'IBM MQC redistributable client for <Your Operating System>' by ticking the box on the left of the package as shown below. It should have **Redist** in the file name. This tutorial was created using the Linux Ubuntu Operating system
![Image showing selecting the redistributable MQ C client compatible with an Operating System](./images/mqoc_ams_mqclient2.png)
3. Select to download via HTTPS, this will allow you to download the client directly through your browser as shown below
![Image showing a number of download options with HTTPS being selected](./images/mqoc_ams_mqclient3.png)
  - **Note**: if you do not have this option, try in an alternative browser.
4. After clicking on continue. You will be redirected to screen shown below. Click on the symbol as shown by the red circle to begin your download
![Click on the image indicated by the red circle](./images/mqoc_ams_mqclient4.png)
5. Once downloaded, extract the file to a directory of your choice `<PATH_TO_MQCLIENT_DIR>`
  - `tar -xvzf <IBM-MQC-Redist>.tar.gz <PATH_TO_MQCLIENT_DIR>`
6. Add commands to path
  - `export PATH=$PATH:<PATH_TO_MQCLIENT_DIR>/bin`
