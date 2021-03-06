# cphtestp
Environment for creating a docker image running cph performance tests for Persistent and Non Persistent messaging.

This repository contains a set of files to help create a Docker image containing the CPH executable and a set of scripts to run an inital set of performance tests.

You will need to seperately download the MQ Client (for which license agreement is required) and copy the following files into the root directory before building your docker image:
* /lap/
*  mqlicense.sh
*  ibmmq-client_9.1.0.0_amd64.deb
*  ibmmq-runtime_9.1.0.0_amd64.deb

The MQ V9 client can be obtained from:
http://www-01.ibm.com/support/docview.wss?uid=swg24042176

The MQ V9.1 client can be obtained from:
http://www-01.ibm.com/support/docview.wss?uid=swg24044791

then perform a docker build as normal:

`docker build --tag cphtestp .`

then run in network host mode to connect and run tests against a local QM:

`docker run -it --detach --net="host" cphtestp`

The default configuration looks for a QM located on the localhost called PERF0 with a listener configured on port 1420. The clients will send and receive persistent messages. You can override a number of options by setting environment variables on the docker run command.

`docker run -it --detach --net="host" --env MQ_QMGR_NAME=PERF1 --env MQ_QMGR_HOSTNAME=10.0.0.1 --env MQ_QMGR_PORT=1414 --env MQ_QMGR_CHANNEL=SYSTEM.DEF.SVRCONN --env MQ_QMGR_QREQUEST_PREFIX=REQUEST --env MQ_QMGR_QREPLY_PREFIX=REPLY cphtestp`

In addition to the hostname, port and and QM name, the default channel can be overidden using the MQ_QMGR_CHANNEL envvar and the queue prefixes used for the testing can be set using MQ_QMGR_QREQUEST_PREFIX and MQ_QMGR_QREPLY_PREFIX.

In the latest release further configuration options have been added. The table below provides the full set:

| Envvar                  | Description                                          | Default if not set |
|-------------------------|------------------------------------------------------|--------------------|
| MQ_QMGR_NAME            | Queue Manager Name                                   | PERF0              |
| MQ_QMGR_HOSTNAME        | Hostname where QM is running                         | localhost          |
| MQ_QMGR_PORT            | Port where QM listener is running                    | 1420               |
| MQ_QMGR_CHANNEL         | Channel name to use to connect to QM                 | SYSTEM.DEF.SVRCONN |
| MQ_RESPONDER_THREADS    | Number of responder threads to run                   | 200                |
| MQ_QMGR_QREQUEST_PREFIX | Prefix of request queues to use.                     | REQUEST            |
| MQ_QMGR_QREPLY_PREFIX   | Prefix of reply queues to use.                       | REPLY              |
| MQ_NON_PERSISTENT       | QOS to be used by connecting clients                 | 0 (Persistent)     |
| MQ_USERID               | Userid to use when authenticating                    |                    |
| MQ_PASSWORD             | Password to use when authenticating                  |                    |
| MQ_CPH_EXTRA            | Additional string field to propogate to cph client   |                    |
| MQ_RESULTS              | Log results to stdout at end of tests                | TRUE               |
| MQ_RESULTS_CSV          | Log results to csv file and send to stdout at end    | FALSE              |
| MQ_TLS_CIPHER           | TLS CipherSpec to use                                |                    |
| MQ_ERRORS               | Log MQ error log at end of test                      | FALSE              |

The container will run a number of tests using different numbers of threads with messages of 2K, 20K and 200K. The scenario is a Request/Responder scenario as featured in the latest xLinux and Appliance performance reports available here:
https://ibm-messaging.github.io/mqperf/

When the testing is complete the final results will be posted to the docker logs and can be viewed in the normal way:

`docker logs <containerID>`

Here are some snippets of the human readable MQ_RESULTS:
```
CPH Test Results
Tue Oct 16 15:48:22 UTC 2018
2K
threads=1
avgRate=1150.56
CPU=1.35
Read=0.00
Write=0.00
Recv=0.06
Send=0.05
QM_CPU=2.70
```
and also of the MQ_RESULTS_CSV output:
```
# Tue Oct 16 15:47:52 UTC 2018
# Persistence, Msg Size, Threads, Rate (RT/s), Client CPU, IO Read (MB/s), IO Write (MB/s), Net Recv (Gb/s), Net Send (Gb/s), QM CPU
1,2048,1,1150.56,1.35,0.00,0.00,0.06,0.05,2.70
```

You can also obtain the available results by:

`docker cp <containerID>:/home/mqperf/cph/results .`

The output from the running responder and requester processes can be viewed by:

`docker cp <containerID>:/home/mqperf/cph/output .`

An interactive session with the running container can be access by:

`docker -ti <containerID> /bin/bash`


Logging to a CSV output file is also supported by setting MQ_RESULTS_CSV to true, and the human readable output can be disabled by setting MQ_RESULTS=FALSE. The CSV output contains the date and time of the test, and field description header (both preceeded by #) before the CSV data.

Support for TLS has now been added with the MQ_TLS_CIPHER environment variable, from which a new CCDT will be created for the MQ client to use. Set the CipherSpec to match the CipherSpec defined at the QM. You will need to place a CMS keystore named key.kdb(and its stash file) containing your QM public certificate in the /ssl directory. 

The version of cph contained in this image was taken on 23rd Jan 2018 and built on 64bit xLinux. The most up to date cph code can be found here:
https://github.com/ibm-messaging/mq-cph
