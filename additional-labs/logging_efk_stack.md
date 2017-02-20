# Lab: Logging EFK Stack

With OpenShift an EFK (Elasticsearch, Fluentd, Kibana) stack is delivered, which collects, rotates and aggregates all log files. Kibana allows logs to be searched, filtered and graphically edited.

> [More information] (https://docs.openshift.com/container-platform/3.3/install_config/aggregate_logging.html)

** Best Practice JSON Logging on STDOUT **

Within a container the logs should be written to STDOUT, so that the platform can take care of the aggregation of the logs. If the logs are written in the JSON format, the EFK stack decomposes the logs automatically and allows filtering at the push of a button.

Java EE Example application with Log4j 2 Installing JSON Logging:

`` `
$ Oc new-app openshift / wildfly-100-centos7 ~ https://github.com/appuio/ose3-java-logging.git
$ Oc expose svc ose3-java-logging
`` `

Afterwards use the browser to invoke the application several times to generate some log entries and then select the newly created pod in the web console under Browse> Pods and then the log tab. Here, the default output of a pod of the application is now visible.

With the "View Archive" button you can switch directly to the aggregated logs of the application in the EFK stack. Here, the logs of all pods of the selected application are sorted in time, and if displayed in the JSON format, according to the individual fields:

! [Kibana Screenshot] (/ images / kibana1.png)

All fields logged by the application are now provided with a warning, since they are not yet indexed and can not be filtered, sorted, etc. afterwards. To fix this, click on the reload button! [Kibana Reload Button] (/ images / kibana2.png) under Settings> .all. You can then search for all entries with the same value by pressing a field of a log entry.

The structuring of the Log4j 2 JSON output is currently not ideal for the fields that are supported by the application:

    "ContextMap_0_key": "url",
    "ContextMap_0_value": "http://ose3-java-logging-dtschan.ose3-lab.puzzle.ch/",
    "ContextMap_1_key": "remoteAddr",
    "ContextMap_1_value": "10.255.1.1",
    "ContextMap_2_key": "freeMem",
    "ContextMap_2_value": "16598776",

I would like to:

    "Url": "http://ose3-java-logging-dtschan.ose3-lab.puzzle.ch/",
    "RemoteAddr": "10.255.1.1",
    "FreeMem": "16598776",

Which would also lead to a more comprehensible presentation within Kibana. See also: https://issues.apache.org/jira/browse/LOG4J2-623.

---

**The End **

[<< back to overview] (../README.md)
