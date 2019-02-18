---
layout: post
title:  "Deploying a Spark Cluster: Comparing Standalone, Yarn, Mesos and DC/OS."
date:   2016-04-16 20:07:08 -0400
categories: jekyll update
published: true
---

<em>Note: I originally wrote this paper for the former BoldRadius blog in April of 2016. BoldRadius was acquired by Lightbend in May, 2016 during which time this has served as an internal resource for learning about Spark clusters. In September of 2016, I updated the DC/OS section as the product had improved significantly - and continues to improve. Before this gets too stale (The DC/OS portion which is most likely to change), I am publishing this with permission here to share publicly for all to learn.</em>

[toc]
<h2><span style="font-weight: 400;">Deploying Apache Spark</span></h2>
<span style="font-weight: 400;">While Apache Spark was only released in 2014, it has quickly become the go to solution for processing quantities of data that are so large they cannot fit onto a single machine - otherwise known as “Big Data”. Spark is orders of magnitude faster than the incumbent solution - Hadoop. It comes packaged with a developer friendly API that allows it to be applied to modern use cases that may involve processing several never ending streams of incoming data ("fast data"). Thus, Spark is now the obvious choice for handling today’s ‘Big Data’ processing applications. For more information on Apache Spark’s place in modern Big Data applications, refer to Lightbend’s white paper </span><a href="https://info.lightbend.com/COLL-20XX-Fast-Data-Big-Data-Evolved-WP_LP.html?lst=WS&amp;lsd=COLL-20XX-Fast-Data-Big-Data-Evolved-WP"><span style="font-weight: 400;">“Fast Data - Big Data Applications evolved”</span></a><span style="font-weight: 400;">.</span>

<span style="font-weight: 400;">While running Spark on a single machine is a great way to become familiar with the API, the true power is only seen when Spark is deployed in a cluster environment. Spark is a “big data” processing application after all, and if your data can be contained on a single box, Spark is overkill. To enable Spark to use a cluster of computers, Spark requires you to setup and configure a Cluster Management application.</span>
<h2><span style="font-weight: 400;">What is a Cluster Management tool and why do I need one?</span></h2>
<span style="font-weight: 400;">Computers/Servers/Boxes/Nodes - be they physical in your data centre or garage or virtual in the cloud - are not free and you want to ensure that you are getting the most out of your investment. It would make no sense to buy more servers if your existing servers are all only 25% utilized. You may also have mission critical applications that you want to ensure always have ample resources to respond to requests, and don’t become starved by lower priority applications.</span>

<span style="font-weight: 400;">You could have your operations personnel carefully monitor each of your server instances, and move around applications based on resource utilization to ensure that each application is getting the appropriate amount of memory, CPU, disk etc. Once your cluster grows beyond a few instances though, this will become impractically time consuming.</span>

<span style="font-weight: 400;">A cluster management tool effectively automates this task - ensuring that whether you have 5 or 1 million nodes in your cluster, they are being used effectively by your applications. This is especially important for distributed applications like Apache Spark - each time an application is started, they need to know where they can find the computing resources to run a given application. A cluster management tool is where they get this information.</span>
<h2><span style="font-weight: 400;">Which one should I use?</span></h2>
<span style="font-weight: 400;">Currently, Spark supports three such applications - Standalone, Yarn and Mesos. To provide some perspective, a </span><a href="http://cdn2.hubspot.net/hubfs/438089/DataBricks_Surveys_-_Content/Spark-Survey-2015-Infographic.pdf?t=1443057549926"><span style="font-weight: 400;">2015 Databricks survey</span></a><span style="font-weight: 400;">, 48% of Spark users are using the Standalone Cluster manager, 40% are using Yarn and 11% are using Mesos. You might conclude from these statistics that Standalone is the way to go or that Yarn is a better choice over Mesos, but these statistics do not tell the whole story.</span>

<span style="font-weight: 400;">Standalone is Spark’s built-in cluster manager which provides just enough to run Spark jobs over a few machines.  It’s where most people start. Yarn provides significantly more job scheduling features and comes from the Hadoop ecosystem, and thus tends to be the choice of those migrating from an existing Hadoop cluster. Mesos on the other hand, is not tied to a particular ecosystem and aims to be the ultimate arbiter for all of your cluster computing resources for all applications - not just Spark.</span>

As an interesting aside - even though Mesos is the up and comer with the lowest adoption of the three, it was also developed by Spark author Matei Zaharia at UC Berkeley.

<span style="font-weight: 400;">In this paper, I’ll discuss my experience with each of these technologies while giving a quick overview of what each brings to the table, and when you might want to use them.</span>

<span style="font-weight: 400;">To get a feel for each of these cluster managers, I set up a 4 node test cluster with resources provided by </span><a href="http://www.canarie.ca/cloud/"><span style="font-weight: 400;">CANARIE’s DAIR</span></a><span style="font-weight: 400;"> program and deployed Spark with HDFS using all three technologies. I also setup a trial with Mesosphere’s DC/OS on AWS - which while built on top of Mesos is unique enough to deserve its own evaluation.  </span>

<span style="font-weight: 400;">Note: Ansible is my provisioning tool of choice and I have left all of the my scripts </span><a href="https://github.com/boldradius/SparkExperiment"><span style="font-weight: 400;">here</span></a><span style="font-weight: 400;">. These are prototype (not high-availability / production grade) scripts, and I make no commitment to keep them up to date. Still, they may be helpful for getting started with a minimal setup.</span>
<h2><span style="font-weight: 400;">Spark’s Distributed Computing Model</span></h2>
<span style="font-weight: 400;">Before we dig into cluster managers, it helps to understand how Spark distributes applications among multiple machines (aka ‘nodes’). Note that the </span><i><span style="font-weight: 400;">italicized</span></i><span style="font-weight: 400;"> terms below represent Spark terms that will be used throughout this document.</span>

<img class="aligncenter wp-image-749" src="http://www.jamielongmuir.com/wp-content/uploads/2016/12/image10.png" width="608" height="499" />

<span style="font-weight: 400;">A Spark </span><i><span style="font-weight: 400;">application</span></i><span style="font-weight: 400;"> consists of an independent set of processes co-ordinated by the </span><i><span style="font-weight: 400;">SparkContext</span></i><span style="font-weight: 400;"> object, which is created in your Spark application’s main class (called the </span><i><span style="font-weight: 400;">driver</span></i><span style="font-weight: 400;"> program). The SparkContext connects to a </span><i><span style="font-weight: 400;">cluster manager</span></i><span style="font-weight: 400;"> (Standalone, Yarn or Mesos) to allocate computing resources for Spark application </span><i><span style="font-weight: 400;">jobs</span></i><span style="font-weight: 400;"> to execute.</span>

<span style="font-weight: 400;">Each </span><i><span style="font-weight: 400;">application</span></i><span style="font-weight: 400;"> will consist of one or more </span><i><span style="font-weight: 400;">actions</span></i><span style="font-weight: 400;"> (save, reduce, collect, take, etc) that return the result of a computation. A </span><i><span style="font-weight: 400;">job</span></i><span style="font-weight: 400;"> refers to an </span><i><span style="font-weight: 400;">action</span></i><span style="font-weight: 400;"> and any associated data </span><i><span style="font-weight: 400;">transformations(</span></i><span style="font-weight: 400;">map, filter, etc) that can be performed in parallel across the cluster. It is these </span><i><span style="font-weight: 400;">jobs</span></i><span style="font-weight: 400;"> that our cluster manager will attempt to effectively distribute across the cluster. Note that a </span><i><span style="font-weight: 400;">job</span></i><span style="font-weight: 400;"> may further be broken down into </span><i><span style="font-weight: 400;">stages</span></i><span style="font-weight: 400;">, which are groups of transformations that can be completed without the need to shuffle data around the cluster (e.g. a groupBy(x) would require shuffling data between executors). An experienced Spark developer will try to write their application to minimize the number of these shuffle tasks. A </span><i><span style="font-weight: 400;">task </span></i><span style="font-weight: 400;">represents the instance of a </span><i><span style="font-weight: 400;">job</span></i><span style="font-weight: 400;"> on a single node in the cluster, usually executed against the subset of data stored on that node (data locality - for performance). This is explained in more detail </span><a href="http://blog.cloudera.com/blog/2015/03/how-to-tune-your-apache-spark-jobs-part-1/"><span style="font-weight: 400;">here</span></a><span style="font-weight: 400;">.</span>

Note: Although the above diagram implies that Jobs are executed serially, it is possible that Job 2’s tasks could begin to execute before Job 1 has completed. Whether this happens or not, is determined by the Cluster Manager, which the rest of this blog post will examine in detail.

&nbsp;

<hr />

<h2><span style="font-weight: 400;">Spark’s Standalone Cluster Manager</span></h2>
<span style="font-weight: 400;">This is the built-in cluster manager that Spark provides to help you get a small cluster up and running. If you’re familiar with deploying applications to Linux servers, the process is relatively straightforward. If you’ve never deployed a Spark cluster, this likely where you want to start prototyping. </span>
<h3><span style="font-weight: 400;">Test Cluster Architecture</span></h3>
<img class="aligncenter wp-image-741 " src="http://www.jamielongmuir.com/wp-content/uploads/2016/12/image02.png" width="509" height="249" />

<span style="font-weight: 400;">With the Spark Standalone cluster manager, slaves are fixed and each job will utilize all available slaves by default.</span>
<h3><span style="font-weight: 400;">Deployment</span></h3>
<span style="font-weight: 400;">In my case, I wrote a few ansible scripts to provision each node individually. Slaves simply require the master’s IP address to launch. This was straightforward, and my cluster was up and running in a couple of hours. A quick visit to the Spark console confirms that the Spark cluster is up and running with the 3 worker nodes.</span>

<img class="aligncenter wp-image-746 size-full" src="http://www.jamielongmuir.com/wp-content/uploads/2016/12/image07.png" width="1999" height="998" />

<span style="font-weight: 400;">Spark also provides a set of scripts that will allow you start/stop all of your worker nodes with a single script by listing all of your slave nodes in a configuration file (note: this requires that all nodes in your cluster are reachable via passwordless ssh). </span>
<h3><span style="font-weight: 400;">High Availability with Zookeeper</span></h3>
<span style="font-weight: 400;">While this setup only provides for one master node (a single point of failure), you can deploy multiple masters and configure Apache Zookeeper for leader election to ensure that your Spark cluster is protected against Master failure. I did not do this for the Standalone setup, but did for the Mesos setup below.</span>
<h3><span style="font-weight: 400;">Scheduling</span></h3>
<span style="font-weight: 400;">By default, all cores of the cluster are used when a Spark application is started following a strict first-in first-out mechanism. It is possible to restrict the number of cores that each application can occupy to enable some concurrency, but for the most part, new applications entering the queue will have to wait until the previous one has completed. </span>

<span style="font-weight: 400;">As of Spark 0.8, there is also a “Fair” scheduling mechanism where Spark assigns tasks between jobs in a ‘round-robin’ fashion, ensuring all jobs get roughly an equal share of resources. </span><span style="font-weight: 400;">This means that short jobs submitted while a long job is running can start receiving resources right away and still get good response times, without waiting for the long job to finish.</span>

<span style="font-weight: 400;">If you want more fine grained control over job scheduling, you’ll have to deploy with Yarn or Mesos.</span>
<h3><span style="font-weight: 400;">Launching Spark Applications</span></h3>
<span style="font-weight: 400;">For testing the cluster, I launched a Spark application using the spark-submit script, specifying the master address in the form: spark://&lt;host&gt;:7077/ as well as the application JAR to be executed as follows.</span>
<pre><i><span style="font-weight: 400;">spark-submit --name "SparkExp" --master spark://&lt;master host IP&gt;:7077  --class MainHDFS  my_application.jar</span></i></pre>
<span style="font-weight: 400;">The spark-submit script automatically distributes the application JAR to the worker nodes, while my data was stored in HDFS.</span>

<span style="font-weight: 400;">Because Spark is managing all of the master and slave nodes, application details such as logs, </span><span style="font-weight: 400;">direct acyclic graph </span><span style="font-weight: 400;">(DAG) visualization (below), timelines, etc. are all visible from the web UI provided you’ve configured </span><span style="font-weight: 400;">spark.eventLog.enabled to be true.</span><span style="font-weight: 400;"> For Yarn or Mesos setups, you will have to configure and launch the Spark History Server to obtain these same console views.</span>

<img class="aligncenter wp-image-740 size-full" src="http://www.jamielongmuir.com/wp-content/uploads/2016/12/image01.png" width="1999" height="1146" />
<h3><span style="font-weight: 400;">Amazon EC2</span></h3>
<span style="font-weight: 400;">While my deployment above was using servers provided by the CANARIE group, Spark provides a </span><a href="http://spark.apache.org/docs/latest/ec2-scripts.html"><span style="font-weight: 400;">handy script </span></a><span style="font-weight: 400;">for automating deployment to EC2 servers. I still favour using a tool like Ansible/Chef/Puppet, as it will allow you to provide custom configurations in your automated deployments, but this will give you a cluster faster.</span>
<h3><span style="font-weight: 400;">Overall</span></h3>
<span style="font-weight: 400;">Spark Standalone cluster manager is most often the way to go for setting up your first Spark cluster, and it is likely all you need if you do not have significant resource contention and a FIFO mechanism for job scheduling is acceptable. If you have a small cluster that is dedicated to running Spark jobs, this is may be all you need. If you are constructing a large big data application with multiple distributed applications as well as multiple producers and consumers of data, you’ll likely want something that provides finer grained management - both of your job scheduling queues, and available cluster computing resources.</span>

&nbsp;

<hr />

<h2><span style="font-weight: 400;">Yarn</span></h2>
<span style="font-weight: 400;">If you’re running a Hadoop cluster with MapReduce 2.0, you are likely already using Yarn (aka “Yet Another Resource Negotiator”) to manage your Hadoop jobs, thus configuring Spark to run on your Yarn cluster is the path of least resistance. Even if you’re not running a full Hadoop stack, if your data is stored in HDFS, Yarn will help ensure all processing takes advantage of data locality. Finally, if you’ve been using Spark’s Standalone cluster manager and want to add finer grained scheduling, prioritization, security and allocation of your resources for jobs, Yarn is still the best solution forward.</span>
<h3><span style="font-weight: 400;">Test Cluster Architecture</span></h3>
<img class="aligncenter wp-image-739 " src="http://www.jamielongmuir.com/wp-content/uploads/2016/12/image00.png" width="509" height="309" />

<span style="font-weight: 400;">The first thing you’ll notice about this compared to Standalone is that there are no Spark Slave nodes. Instead, there is a cluster of Yarn NodeManagers. When a Spark application is launched, the Spark binaries and application JAR will automatically be distributed to all of the slave nodes that will be utilized  (Yarn handles this for you) - thus it is not necessary to install Spark on the slave nodes.</span>

<span style="font-weight: 400;">In Yarn, the ResourceManager is the central authority that decides who gets what among all the application in the system. It accepts Spark job submissions and negotiates for the first available container to start executing it. Each worker node contains a NodeManager responsible for the containers on that node as well as monitoring resources usage and reporting to the ResourceManager. </span>

<span style="font-weight: 400;">Note that this configuration contains a single ResourceManager - a single point of failure in the cluster. While handy for prototyping, this is not a high-availability configuration that you’d want for a reliable production setup. For a high-availability configuration, you would want a standby ResourceManager with automatic failover provided by Zookeeper. </span>
<h3><span style="font-weight: 400;">Deployment</span></h3>
<span style="font-weight: 400;">While setting up a Yarn cluster can be a lot more involved than a Standalone Spark cluster - as there are a multitude of possible configuration possibilities, the documentation is very thorough and complete. There is also a large Hadoop community to provide help if you get stuck. As with Standalone, I had my prototype cluster working within a day. Once you have a Yarn Cluster, deploying Spark is even simpler than Standalone, as you only need to deploy Spark Master application(s) and History server to provide log viewing (no slaves) - Yarn will ensure that each worker node has everything that it needs to run your Spark jobs.</span>
<h3><span style="font-weight: 400;">Scheduling</span></h3>
<span style="font-weight: 400;">Yarn provides two off-the-shelf, highly tunable Schedulers for assigning applications to nodes - Fair Scheduler and Capacity Scheduler. There is also an API to write your own Scheduler.</span>

<span style="font-weight: 400;">Capacity Scheduler (default) supports a set of hierarchical queues to allow for more predictable sharing of resources. It allows for the partitioning of the cluster’s resources among different organizations, providing a minimum capacity guarantee for each organization. Each queue can be allocated a fraction of the cluster’s capacity while having its own security policies for ensuring only authorized users can submit/view requests for their queues.</span>

There is also a feature called ‘Node Labels’ which allows for individual nodes in the cluster to be reserved for particular queues.

<span style="font-weight: 400;">Fair Scheduler ensures that all applications receive, on average, an equal share of resources over time. </span><span style="font-weight: 400;">When there is a single job running, that job uses the entire cluster. When other jobs are submitted, freed up resources are assigned to them, so that each job eventually gets roughly the same amount of resources. </span><span style="font-weight: 400;">Fair Scheduler supports application priorities via weights that</span><span style="font-weight: 400;"> are used to determine the fraction of total resources that each app should get.</span>

<img class="aligncenter wp-image-747 size-full" src="http://www.jamielongmuir.com/wp-content/uploads/2016/12/image08.png" width="1999" height="844" />
<h3><span style="font-weight: 400;">Launching Spark Applications</span></h3>
<span style="font-weight: 400;">Running a Spark application with Yarn is even simpler than Standalone, as Spark is able to use the Yarn configuration (specified in an environment variable) to connect to the Yarn ResourceManager and let Yarn launch jobs in the cluster. The Yarn queue to be used can be specified in the submit command. The Spark binaries and application JAR are automatically distributed to the chosen worker nodes (as specified or scheduler determined). Yarn also works directly with HDFS to ensure optimal performance by taking advantage of data locality.</span>
<pre><span style="font-weight: 400;">spark-submit --name "SparkExp"  </span><b>--master yarn-client</b><span style="font-weight: 400;">  --class MainHDFS --queue JamieQueue hdfs://hdfs_server/path/to/my/jar/application.jar</span></pre>
<span style="font-weight: 400;">Unlike the Spark Standalone cluster, with Yarn and Mesos you will need to start the Spark History server to view completed application details (logs, DAG, timelines, etc). The Spark History server creates a web interface on port 18080 that will allow viewing of the same logs and job visualizations that were viewable directly from the Spark console in Standalone or local mode.</span>

<img class="aligncenter wp-image-751 size-full" src="http://www.jamielongmuir.com/wp-content/uploads/2016/12/image12.png" width="1999" height="653" />
<h3><span style="font-weight: 400;">Apache Slider</span></h3>
<span style="font-weight: 400;">While it’s still in the incubator phase, the </span><a href="http://slider.incubator.apache.org/"><span style="font-weight: 400;">Apache Slider</span></a><span style="font-weight: 400;"> project aims to make it much easier to port existing distributed applications to Yarn. This would allow Yarn to manage far more than just your Spark and Hadoop jobs - and take away one of the major arguments for Mesos. There are already projects for </span><a href="https://github.com/DataTorrent/koya"><span style="font-weight: 400;">Kafka on Yarn</span></a><span style="font-weight: 400;"> and </span><a href="https://github.com/hkropp/slider-cassandra"><span style="font-weight: 400;">Cassandra on Yarn</span></a><span style="font-weight: 400;"> in development, and likely more to come. This is still very early days though, with Mesos still representing the more robust total cluster management solution. </span>
<h3><span style="font-weight: 400;">Overall</span></h3>
<span style="font-weight: 400;">I was impressed with how quick and easy it was to set up a Yarn cluster and start experimenting with the many configuration variables while launching Spark applications. Yarn is the way to go if you have an existing Hadoop based cluster or require scheduling, job isolation and additional security features. With detailed documentation and plentiful community support, Yarn is also currently the most field-tested cluster manager for Spark. On the downside, as we’ll see, it does not offer as fine-grained resource utilization as efficiently as Mesos does, nor does it provide a framework for managing your entire data centre (Pending Apache Slider). Thus, if you want your cluster manager to take into account other distributed applications - such as Kafka and Cassandra, Mesos starts to become much more attractive.</span>

&nbsp;

<hr />

<h2><span style="font-weight: 400;">Mesos</span></h2>
<span style="font-weight: 400;">While Yarn provides a solid mechanism for controlling how Spark distributes jobs around your cluster - especially if using other Hadoop technologies such as HDFS, it is (at the moment) limited to managing your Spark (or Hadoop) jobs. </span>

<span style="font-weight: 400;">Modern big data applications are more likely to involve multiple distributed applications from a variety of vendors - incoming data may be queued up with a distributed queuing mechanism such as Kafka, while processed data may be stored in a distributed database such as Cassandra. With multiple distributed applications in play, you’ll want to ensure that each of them is making effective use of available resources. This is where Mesos aims to shine.</span><span style="font-weight: 400;">
</span>

<span style="font-weight: 400;">Not only does Mesos manage individual Spark jobs, it aims to be the manager of ALL of your applications in the cluster - including HDFS, Cassandra, Kafka, web servers or any application that you may want a piece of your cluster resources. Lightbend is even working on an integration to allow for easy deployment of Play, Akka and Lagom applications using ConductR as a Mesos framework.</span>

<span style="font-weight: 400;">The Mesos vision is that it should be able to ensure the efficient utilization of every core or memory bank in your cluster - be it by a Spark job or a web server handling a burst of traffic. Of course, this level of sharing comes with its own overhead that must be taken into account. There is also a significant amount of configuration and tuning to ensure that your applications are able to acquire the necessary resources to operate effectively.</span>
<h3><span style="font-weight: 400;">Test Cluster Architecture</span></h3>
<img class="aligncenter wp-image-748 " src="http://www.jamielongmuir.com/wp-content/uploads/2016/12/image09.png" width="489" height="285" />

<span style="font-weight: 400;">The Mesos Master manages slave daemons and </span><i><span style="font-weight: 400;">Frameworks</span></i><span style="font-weight: 400;"> that run Mesos </span><i><span style="font-weight: 400;">tasks</span></i><span style="font-weight: 400;"> on these slaves. For Spark, the spark-submit or spark-shell would be the framework.</span>

<span style="font-weight: 400;">It’s important to note that Mesos has a very different (some would say inverse) approach to distributing applications compared to Yarn. </span>

<span style="font-weight: 400;">While Yarn uses a central authority (ResourceManager) for distributing applications to nodes as they become available, Mesos’s Master continually sends out resource “offers” that list resources (CPUs, memory, etc.) that are available to run tasks. If an offer aligns with the needs of a given framework (e.g. your Spark job), it can accept the offer and provide the Mesos Master with enough information to launch that framework’s tasks on the offered resources. More on Mesos’ resource offer architecture </span><a href="http://mesos.apache.org/documentation/latest/architecture/"><span style="font-weight: 400;">here</span></a><span style="font-weight: 400;">.</span>

<span style="font-weight: 400;">Browsing to the Mesos console, you can view all of the frameworks running on your cluster - below I have Marathon, Chronos and a SparkContext started, though they are not currently occupying any of the cluster’s resources.</span>

<span style="font-weight: 400;">Marathon describes itself as a container orchestration system that enables you to run, monitor and scale distributed applications across the Mesos cluster. It can be thought of as analogous to upstart, systemd and systemv for Linux, except applied to a cluster of applications. Similarly, Chronos provides a distributed ‘Cron’ like system. Both applications are heavily used as part of Mesosphere’s DC/OS, which we’ll talk about after focusing exclusively on Mesos.</span>

<img class="aligncenter wp-image-750 size-full" src="http://www.jamielongmuir.com/wp-content/uploads/2016/12/image11.png" width="1999" height="704" />
<h3><span style="font-weight: 400;">Deployment</span></h3>
<span style="font-weight: 400;">To deploy a Mesos cluster, you can either download the binaries from Apache and build them yourself (can take greater than 1 hour) or use pre-built packages from 3rd parties like </span><a href="https://mesosphere.com/downloads/"><span style="font-weight: 400;">Mesosphere</span></a><span style="font-weight: 400;">. I chose the later route. </span>

<span style="font-weight: 400;">Deploying Spark with Mesos is easily the most complex of the three possible cluster managers.  While Mesosphere provides </span><a href="https://open.mesosphere.com/getting-started/install/"><span style="font-weight: 400;">detailed instructions</span></a><span style="font-weight: 400;"> to help get you started, it is a lengthy procedure that involves installing both Mesos Master and Slaves services on all nodes, then configuring the Slaves individually and removing the already started Master services. There are several possible configuration gotchas to fall into along the way, and not nearly the same level of documentation or community support as are available for Yarn or Standalone setups. </span><a href="http://data-scientist-in-training.blogspot.ca/2015/03/apache-spark-cluster-deployment-part-2.html"><span style="font-weight: 400;">This blog post </span></a><span style="font-weight: 400;">is probably the best summary of the possible traps you can fall into when installing Spark with Mesos - in particular, I struggled with the ‘hadoop home’ issue for several hours. My ansible scripts are </span><a href="https://github.com/boldradius/SparkExperiment"><span style="font-weight: 400;">here</span></a><span style="font-weight: 400;"> as well.</span>

<span style="font-weight: 400;">The installation instructions include Marathon, Chronos and Zookeeper (for providing leader election among multiple masters). These are not required for deploying a Spark cluster, but will come in very handy for managing other applications with Mesos.</span>
<h3><span style="font-weight: 400;">Scheduling</span></h3>
<span style="font-weight: 400;">Spark can schedule jobs using two modes in Mesos. “Coarse-grained” mode is similar to the static resource allocation of Standalone and Yarn, where each Spark job is given a maximum amount of resources that it will hold onto until it is completed. This allows for much lower startup overhead at the cost of reserving resources for the entire duration of the application.</span>

<span style="font-weight: 400;">In “Fine-grained” - which is unique to Mesos, each Spark </span><i><span style="font-weight: 400;">task</span></i><span style="font-weight: 400;"> runs as a separate Mesos </span><i><span style="font-weight: 400;">task</span></i><span style="font-weight: 400;">. This allows for dynamic sharing of CPUs. While memory allocations are still fixed, when an application is not running a task on a machine, other applications may run tasks on those cores. This is useful if you expect a large number of not-overly-active applications, where the tradeoff of less predictable latency (because a core may not be immediately available when work is to be done) is acceptable.</span>

<span style="font-weight: 400;">Like Yarn, Mesos provides a mechanism - “roles”,  for giving preference or reserving resource offers to one or more frameworks. This can allow the cluster to be divided among different application types or organizations. Mesos also provides “weights” as a means of specifying the relative share of cluster resources that is offered to different roles.  </span>

<span style="font-weight: 400;">It should be noted, that like Yarn, Mesos’ internal scheduling algorithms are pluggable - with several out there, and a developer guide for writing your own.</span>
<h3><span style="font-weight: 400;">Launching Spark Applications</span></h3>
<span style="font-weight: 400;">Launching applications (or spark-shell) on Mesos is similar to the Standalone in that you specify the Mesos Master address. </span>
<pre><i><span style="font-weight: 400;">spark-submit --name "SparkExp"  --master </span></i><i><span style="font-weight: 400;">mesos://zk://&lt;host&gt;:2181</span></i><i><span style="font-weight: 400;">  --class MainHDFS hdfs://1</span></i><i><span style="font-weight: 400;">&lt;host&gt;</span></i><i><span style="font-weight: 400;">:9000/path/to/jar/my_application.jar</span></i></pre>
<span style="font-weight: 400;">Unlike Yarn, you will need to ensure that all Mesos slaves can access a common version of the Spark binaries (I suggest saving a copy to a shared storage storage location such as HDFS). Running Spark in Cluster-Mode (where the Spark Driver App is started from a worker), also requires that you start an additional </span><span style="font-weight: 400;">MesosClusterDispatcher application.</span>

<span style="font-weight: 400;">Once an application is launched, Spark will look at resource offers coming from Mesos until it receives one with enough resources to launch the application’s jobs. These requirements are part of Spark’s executor configurations. If Spark does not receive a sufficient resource offer, you’ll see the following message:</span>
<blockquote><i><span style="font-weight: 400;">Initial job has not accepted any resources; check your cluster UI to ensure that workers are registered and have sufficient resources</span></i></blockquote>
<span style="font-weight: 400;">Because my cluster’s 3 worker nodes were small instances (1 core/1g mem), the resources offers from Mesos were too small for Spark’s default memory/core requirements and thus I saw that message. After adjusting these settings (discussed in this </span><a href="http://pkoperek.github.io/braindump/2015/02/18/spark-not-accepting-resources/"><span style="font-weight: 400;">blog post</span></a><span style="font-weight: 400;">), I was able to start both spark-shell and spark-submit application.</span>

<span style="font-weight: 400;">Like Yarn, you will have to launch and use the Spark history server to view the details of your completed jobs.</span>
<h3><span style="font-weight: 400;">Resource Offers</span></h3>
<span style="font-weight: 400;">It’s worth noting that a common reason an application may fail to start in Mesos (and by extension Marathon and DC/OS) is a lack of the available resource offers. If you find that applications are stuck in “Waiting…” in Marathon or that your Spark task stays queued, you may see entries like the following in your Marathon logs:</span>
<blockquote><span style="font-weight: 400;">Aug 18 07:58:37 m1.dcos java[23145]: [2016-08-18 07:58:37,826] INFO Offer [e4118d63-322a-40e3-9cb8-44c966b6665d-O900]. Considering unreserved resources with roles {*}. Not all basic resources satisfied: cpus SATISFIED (1.0 &lt;= 1.0), mem SATISFIED (2048.0 &lt;= 2048.0), disk including volumes NOT SATISFIED (10000.0 &gt; 5746.0) </span></blockquote>
<span style="font-weight: 400;">This indicates the application (Apache Zeppelin in this case) is not able to run because it has not received any offers that contain enough disk space - 10g in this case.</span>
<h3><span style="font-weight: 400;">Apache Myriad</span></h3>
<span style="font-weight: 400;">It’s worth mentioning that it doesn’t have to be an either/or choice between Yarn and Mesos. </span><a href="http://myriad.incubator.apache.org/"><span style="font-weight: 400;">Apache Myriad</span></a><span style="font-weight: 400;"> is an ongoing project developing a Mesos framework for Yarn. This may be particularly useful if you have a finely tuned Yarn cluster that you wish to take advantage of for using Spark with HDFS, yet want to start leveraging Mesos’ ability to manage your entire data centre.</span>
<h3><span style="font-weight: 400;">Overall</span></h3>
<span style="font-weight: 400;">It took longer to get Spark running on a Mesos cluster compared with Yarn, with a lengthy setup process and more technical gotchas along the way. I also found that Mesos’ system of resource offers to frameworks may be suboptimal for a small cluster setup. That said, for a small cluster, Standalone is the way to go. </span>

<span style="font-weight: 400;">Mesos’ real selling point is not that it efficiently will let you distribute Spark jobs, but rather that it will allow you to make efficient use of many nodes in a large data centre - even if your Spark jobs represent a fraction of your workload. To realize these benefits, you would use Marathon and/or Chronos to launch several distributed applications over many machines that are large enough to ensure that the resource offers provided to your applications are sufficient for them to run.</span>

&nbsp;

<hr />

<h2><span style="font-weight: 400;">Mesosphere’s DC/OS</span></h2>
<span style="font-weight: 400;">Recognizing the power in having a single platform to manage all of your cluster resources - and the challenges for many in getting started (see above), Mesosphere developed a product called DC/OS (Data Centre Operating System) that wraps Mesos and Marathon to deploy, monitor and manage all of your applications over a cluster of nodes (ie an operating system for the data centre). Mesosphere both provides a command-line interface and web gui that let you manage your cluster as if it was a single computer.</span>

<span style="font-weight: 400;">It should be noted that while Mesosphere’s DC/OS is still in its early days as a product, it has seen massive improvements in the 6 months since our first setup. The most notable change during that time is the open sourcing of DC/OS in April of 2016, which will no doubt ensure that DC/OS continues to evolve to become more robust and offer even better support for distributed applications such as Spark, Kafka, Cassandra and more.</span>

<span style="font-weight: 400;">To help you get started, Mesosphere supports a growing list of possible deployment infrastructures - including AWS, Azure and custom installs on Centos and Vagrant VMs. During our initial experimentation, the community edition was limited to deploying on AWS. The community edition of DC/OS will let you setup a cluster on AWS and only pay for the standard AWS usage fees. Keep in mind, the basic 5 node cluster (was 8 at the time), will generate considerable AWS fees, so if you’re just experimenting, I recommend starting with the </span><a href="https://dcos.io/docs/1.7/administration/installing/local/"><span style="font-weight: 400;">DCOS Vagrant setup</span></a><span style="font-weight: 400;">, then moving over to AWS when you’re looking to deploy a usable prototype.</span>
<h3><span style="font-weight: 400;">AWS Cluster Architecture</span></h3>
<span style="font-weight: 400;">After running Mesosphere’s AWS CloudDeploy template and installing HDFS and Spark, we had a cluster that looks roughly like the following with each node being an M3.xLarge. Apart from having 5 worker nodes and the HDFS cluster being designed for high-availability (hence multiple NameNodes and 3 JournalNodes), it is similar to the manually provisioned clusters earlier.</span>

<img class="aligncenter wp-image-744" src="http://www.jamielongmuir.com/wp-content/uploads/2016/12/image05.png" alt="" width="502" height="383" />
<h3><span style="font-weight: 400;">Deployment</span></h3>
<span style="font-weight: 400;">There are now several methods for installing the community edition of DC/OS. There’s the original method which uses AWS Cloud Formation, as well as Azure Resource Manager, Packet Terraform as well as custom installations using a </span><a href="https://mesosphere.com/blog/2016/03/30/dcos-gui-installer/"><span style="font-weight: 400;">web GUI</span></a><span style="font-weight: 400;">, CLI commands or manually. We’ll focus on the original AWS Cloud Formation installation below.</span>

<span style="font-weight: 400;">To setup a community DC/OS cluster on AWS, you will need to provide Mesosphere with a Google, Github or Azure account and contact information. A few mouse clicks and a cup of coffee later - your cluster is baked! </span>

<span style="font-weight: 400;">Once the setup completes, you are directed your DC/OS web dashboard, which looks like this.</span>

<img class="aligncenter wp-image-743 size-full" src="http://www.jamielongmuir.com/wp-content/uploads/2016/12/image04.png" width="1999" height="1001" />

<span style="font-weight: 400;">From there, you can install the DC/OS CLI (bottom left sign post icon) on your local machine to begin installing services like HDFS and Spark which are both done with single commands. Custom install options are specified with an options.json configuration file which can be added when installing via the CLI.</span>

<span style="font-weight: 400;">It should be noted that for Spark - the History Server (discussed above) is an optional install, and configured via the options.json file. It also requires HDFS to provide a shared volume. It would be specified in an options.json as follows:</span>
<pre><span style="font-weight: 400;">{</span>
<span style="font-weight: 400;">   "history-server": {</span>
<span style="font-weight: 400;">     "enabled": true</span>
<span style="font-weight: 400;">   }</span>
<span style="font-weight: 400;">}</span></pre>
<span style="font-weight: 400;">Once installation has started, you can go to the Marathon dashboard to view your application’s installation progress. From marathon, you can start/stop application deployments, as well as scale to multiple nodes.</span>

<img class="aligncenter wp-image-745 size-full" src="http://www.jamielongmuir.com/wp-content/uploads/2016/12/image06.png" width="1999" height="853" />

<span style="font-weight: 400;">While this simple installation is convenient, the Mesosphere wrapped distributed applications have resulted in some challenges in trying to customise deployments and configurations. The documentation here is improving rapidly, though it’s still in its early days. Luckily, Mesosphere provides timely live support via their Slack channel if you have any questions.</span>

<span style="font-weight: 400;">It’s also worth noting that many services - especially community driven ones, are currently in a beta or alpha phase, so you may see noticed like the following when you go to install a particular service.</span>

<span style="font-weight: 400;">“Note that the service is alpha and there may be bugs, including possible data loss, incomplete features, incorrect documentation or other discrepancies.”</span>

<span style="font-weight: 400;">Once your applications are installed, you will see your processes on the DC/OS dashboard under services.</span>

<img class="aligncenter wp-image-742 size-full" src="http://www.jamielongmuir.com/wp-content/uploads/2016/12/image03.png" width="1999" height="931" />

<span style="font-weight: 400;">From there, you should be off to the races, ready to start integrating your different distributed applications.</span>

<span style="font-weight: 400;">When we did our initial experimentation in March of 2016, we ran into challenges ensuring that Spark workers were able to communicate with the DC/OS HDFS installation. Documentation at the time was quite limited, though seems to have improved significantly and no doubt will continue to do so.</span>

<b><i>Warning to avoid an unexpected AWS bill:</i></b><span style="font-weight: 400;"> If, like me, you are experimenting with DC/OS, be sure to shut down your cluster using the Cloud Formation Management page and selecting “Delete Stack”. Anything else (stopping, terminating EC2s) will result in the stack being recreated and your continued billing at $50-60+/day! Unfortunately, you will have to setup your entire cluster from scratch, which does not appear to be automatable(see previous paragraph).</span>
<h3><span style="font-weight: 400;">Local Setup with Vagrant</span></h3>
<span style="font-weight: 400;">As mentioned above, there are multiple ways to get started with DC/OS. If you have a relatively beefy local computer ( &gt;= 12G of RAM FREE), the Vagrant based local setup is a great way to get a feel for DC/OS before deploying to a live cluster that will start running up server bills.</span>

<span style="font-weight: 400;">Like all of DC/OS now, the </span><a href="https://github.com/dcos/dcos-vagrant/"><span style="font-weight: 400;">DC/OS Vagrant project</span></a><span style="font-weight: 400;"> is open source with comprehensive documentation to get your local DC/OS setup going. Keep in mind, as the Vagrant VM instances are utilizing your local machine’s memory and CPU, thus the amount of work you can actually perform is quite limited. For example, I wasn’t able to setup a cluster large enough to run DC/OS’s HDFS installation on my local cluster, but I was able to get Apache Spark running with some of sample Spark jobs that come with the distribution (e.g. PiSpark). This setup is still great for learning about DCOS and seeing how easy it is to setup a Mesos cluster and install distributed applications. </span>
<h3><span style="font-weight: 400;">Launching Spark Applications</span></h3>
<span style="font-weight: 400;">Spark applications are started using the DC/OS CLI Spark command. This command wraps the spark-submit script. </span>

<span style="font-weight: 400;">It’s worth noting that while DCOS does not support running Spark-shell as a means of running interactive Spark sessions, it does support </span><a href="https://zeppelin.apache.org/"><span style="font-weight: 400;">Apache Zepplin</span></a><span style="font-weight: 400;">, which provides a web-UI for creating interactive data analysis session with Spark as well as other database backends.</span>

<span style="font-weight: 400;">For monitoring your Spark applications, DC/OS provides a custom Spark UI on port as well that looks like a variation of the Standalone GUI.</span>

<img class="aligncenter wp-image-752 size-full" src="http://www.jamielongmuir.com/wp-content/uploads/2016/12/image13.png" width="1999" height="859" />

<span style="font-weight: 400;">While I was able to get the example Spark app to run (PiSpark), I ran into several challenges attempting to launch the same application I used to test the other cluster managers on DC/OS (as can be seen by all of the failed jobs above). After launching the app with the DC/OS Spark command, the console showed the state as “TASK_FAILED”. Using the ‘Services’ tab in the DC/OS web gui, I was able to browse the logs for the failed Spark app, showing the error details - my Spark job was not able to contact HDFS. This provided enough information to start debugging - and eventually, to contact Mesosphere’s support team.</span>

<span style="font-weight: 400;">The Mesosphere web GUI itself contain an chat icon - called “</span><a href="https://support.mesosphere.com/hc/en-us/articles/206959716"><span style="font-weight: 400;">private intercom</span></a><span style="font-weight: 400;">” that will allow you send messages to their support engineers who aim to respond within a few hours. Mesosphere engineers also maintain an active presence on their public </span><a href="http://chat.mesosphere.com/"><span style="font-weight: 400;">Slack channel</span></a><span style="font-weight: 400;">, though this does require a registration (separate from your Mesosphere account). Finally, there’s good old StackOverflow, where several Mesosphere engineers are also present.</span>

<span style="font-weight: 400;">With help from Mesosphere engineers, I was able to get the Spark job to complete successfully on my DC/OS cluster, albeit with Spark event logging disabled. This technology was still very much in its early stages when we did this experiment though, so I have no doubts that these hiccups will be smoothed over (if they are not already).</span>
<h3><span style="font-weight: 400;">Overall</span></h3>
<span style="font-weight: 400;">While it’s important to note that Mesosphere’s DC/OS is still an emerging technology (as are all of its competitors), I’ve been very impressed by the improvements in both features and documentation in the first 7 months of 2016. The concept of a data centre OS that abstracts all of your resources is extremely powerful, and one that will become critical as applications strive to become reactive, and therefore distributed with ever growing cluster sizes. I truly believe that technologies like Mesosphere's DC/OS represent the future of large cluster management.</span>

<span style="font-weight: 400;">Mesosphere’s DC/OS is by far the quickest way to deploy a distributed computing cluster, and the integrated dashboards provide a built-in mechanism for easily monitoring the resource utilization of your cluster. Their online support personnel have also shown that just because you are an earlier adopter, does not mean that you will not be hung out to dry when your use case goes off of the documentation or you run into early adopter hick ups.</span>

&nbsp;

<hr />

<h2><span style="font-weight: 400;">Final Summary - Standalone, Yarn, Mesos, DC/OS</span></h2>
<span style="font-weight: 400;">As mentioned at the beginning, according to a </span><a href="http://cdn2.hubspot.net/hubfs/438089/DataBricks_Surveys_-_Content/Spark-Survey-2015-Infographic.pdf?t=1443057549926"><span style="font-weight: 400;">2015 Databricks survey</span></a><span style="font-weight: 400;">, 48% of Spark users are using the Standalone Cluster manager, 40% are using Yarn and 11% are using Mesos. Some of the Standalone users may still be prototyping Spark and may not have a production deployment yet, nevertheless for a small Spark cluster - Standalone is a great place to start. For those starting with a Hadoop cluster, or those that want more job scheduling, isolation or security features - Yarn still represents the most mature mechanism for deploying a Spark cluster. </span>

<span style="font-weight: 400;">That said, Yarn will not manage your entire cluster. It is fairly limited as you move beyond Hadoop and Spark jobs. As you move into the modern era of distributed </span><a href="http://www.reactivemanifesto.org/"><span style="font-weight: 400;">reactive applications</span></a><span style="font-weight: 400;">, the need to provide cluster management for multiple distributed applications grows, and the value from an overall cluster manager like Mesos becomes essential. Mesosphere’s DC/OS, while still an early-stage technology, will save you significant time on the deployment side for a Mesos cluster. If you are planning to build a modern reactive application that will grow to tens, hundreds or thousands of servers, in which Spark is only one of multiple distributed application, Mesosphere’s DC/OS is the way to go.</span>