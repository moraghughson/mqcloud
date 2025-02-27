---
copyright:
  years: 2018, 2020
lastupdated: "2019-02-04"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Queue manager sizes
{: #mqoc_qm_sizes}

Learn about the characteristics of IBM queue managers available to deploy on IBM Cloud.

{:shortdesc}

## Lite queue managers

To give you a taste of MQ as a managed service, the IBM MQ on Cloud service offers a Lite queue manager. If you do not currently have a Lite queue manager deployed, you can create one by following the [creating a queue manager](/docs/services/mqcloud?topic=mqcloud-mqoc_create_qm) guide.

Note:
* You can deploy up to two Lite queue managers per Lite service instance
* You can delete an existing Lite queue manager and deploy a new one at any time
* Lite queue managers will automatically be deleted after 30 days of inactivity but you are welcome to deploy a new one
* A Lite queue manager is considered active if you have sent a message (using a non-SYSTEM queue), or have logged in to the service instance via the IBM Cloud user interface
* You will never be charged for a Lite queue manager

## Billable queue managers

The IBM MQ on Cloud service offers the following billable queue manager sizes:
* **Extra Small**: Appropriate for very light or occasional workloads with modest throughput requirements

* **Small**: Appropriate for light workloads such as supporting an individual department or application

* **Medium**: Appropriate for shared use by a number of light to moderate workload applications

* **Large**: Appropriate for heavy throughput scenarios where transaction performance is critical

A billable queue manager is charged based on the number of Virtual Processor Core (VPC) hours that it is active - for example, a Large size queue manager is charged at four times (4x) the rate of a Small size queue manager. The hourly cost is dependent on your local currency and can be seen in the Pricing Plan at the bottom of the [MQ service catalog page](https://cloud.ibm.com/catalog/services/mq), where you can select your country or region.

You can see details of your billable usage in the [IBM Cloud usage dashboard](https://cloud.ibm.com/account/usage) at the Account or Resource Group level.

**Note**: if you cannot see your MQ usage on the dashboard, but have deployed a billable queue manager, ensure you have selected to view the usage for your resource group, e.g. Under the Usage Dashboard title, in the `Group:` dropdown menu, select `Resource Groups` then `default`.

## Queue manager resources

The following table provides information about the resources available to each queue manager size and performance results based on testing:

|                               | Lite[^f1]   | Extra Small | Small | Medium | Large |
|-------------------------------|---------|-------|-------|--------|-------|
| VPC                           | -       | 0.5   | 1      | 2     |  4    |
| Memory (RAM GB)               | -       | 1     | 1      | 2     |  4    |
| Disk Size (GB)                | -       | 20    | 20     | 40    |  40  |
| Disk performance (IO operations per second - IOPS) | -       | 80    | 200    | 400   | 800  |
| TCP non-persistent message throughput [^f2] | 10000 <br> <small>per month</small> |  800 <br> <small>per second</small>| 1500 <br> <small>per second</small> |  3000 <br> <small>per second</small> | 6000 <br> <small>per second</small> |  
| TCP persistent message throughput [^f3] | 10000 <br> <small>per month</small> | 50 <br> <small>per second</small>|  100 <br> <small>per second</small>| 400 <br> <small>per second</small>| 1000 <br> <small>per second</small>
| REST non-persistent message throughput [^f4] | 10000 <br> <small>per month</small> | 250 <br> <small>per second</small> | 450 <br> <small>per second</small> | 900 <br> <small>per second</small> | 1900 <br> <small>per second</small>
| REST persistent message throughput [^f5] | 10000 <br> <small>per month</small> | 50 <br> <small>per second</small> | 100 <br> <small>per second</small> | 400 <br> <small>per second</small> | 1000 <br> <small>per second</small>
| Maximum concurrent client connections [^f6] | 20      | 30    |  50    |  100  | 200  |

---

[^f1]: Lite queue managers are allocated limited resources and should not be used for performance evaluation.

[^f2]: Performance estimates are highly dependent on the specific application logic and topology. Users are recommended to validate their own specific scenario as part of the testing process. The benchmarking scenario used for the data in this table are as described in the bullet points below.

[^f3]: Performance estimates are highly dependent on the specific application logic and topology. Users are recommended to validate their own specific scenario as part of the testing process. The benchmarking scenario used for the data in this table are as described in the bullet points below.

[^f4]: Performance estimates are highly dependent on the specific application logic and topology. Users are recommended to validate their own specific scenario as part of the testing process. The benchmarking scenario used for the data in this table are as described in the bullet points below.

[^f5]: Performance estimates are highly dependent on the specific application logic and topology. Users are recommended to validate their own specific scenario as part of the testing process. The benchmarking scenario used for the data in this table are as described in the bullet points below.

[^f6]: The IBM MQ JMS client library used for the TCP figures typically uses two client connections per application (one for the JMS Connection and one for each JMS Session) so a connection limit of 20 supports up to 10 concurrent JMS applications.

* _The benchmark applications for producing and consuming messages are scaled to drive the maximum concurrent number of connections for the given queue manager size. Single threaded or limited concurrency applications may not be able to reach the maximum capacity of the queue manager_
* _Applications are deployed in the same cloud region as the queue manager in order to minimise the latency of the connectivity. Applications deployed in different cloud locations or on-premises data centres will result in lower throughput_
* _Anonymous TLS (server-only) is configured on the MQ channels in order to protect message data and credentials as they flow over the network. Non TLS-enabled channels typically have around 10% higher throughput but are not recommended for security reasons_
* _Messages of 2KB size are used for the benchmark scenario. To a reasonable approximation IBM MQ throughput has an inverse linear correlation to the size of the message, so a 4KB message size exhibits roughly half the throughput described above_

**TCP benchmarks**

* _The TCP benchmark applications are written using the IBM MQ C client which uses one client connection per application thread. Please note the comment in footnote 6 above regarding the use of client connections by JMS applications_

**REST benchmarks**

* _The REST benchmark applications are written using a Python client with each application thread using basic authentication to generate a fresh TLS connection per request against the same message queue._
* _REST message throughput will generally be 10-15% higher if the client application uses cookie authentication. See_ [Authenticating your client to invoke REST API requests](/docs/services/mqcloud?topic=mqcloud-mqoc_qm_rest_api)
