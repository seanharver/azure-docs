---
title: Azure Toolkit for IntelliJ – Debug Spark applications remotely through ssh | Microsoft Docs
description: Step-by-step guidance on how to use the HDInsight Tools in Azure Toolkit for IntelliJ to debug applications remotely on HDInsight clusters through ssh
keywords: debug remotely intellij, remote debugging intellij, ssh, intellij, hdinsight, debug intellij, debugging, Executor
services: hdinsight
documentationcenter: ''
author: jejiang
manager: DJ
editor: Jenny Jiang
tags: azure-portal

ms.assetid: 
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.workload: big-data
ms.tgt_pltfrm: 
ms.devlang: 
ms.topic: article
ms.date: 08/24/2017
ms.author: Jenny Jiang

---
# Remotely debug Spark applications on an HDInsight cluster with Azure Toolkit for IntelliJ through SSH

This article provides step-by-step guidance on how to use the HDInsight Tools in Azure Toolkit for IntelliJ to debug applications remotely on an HDInsight cluster. You can follow a [video](https://channel9.msdn.com/Series/AzureDataLake/Debug-HDInsight-Spark-Applications-with-Azure-Toolkit-for-IntelliJ) to debug your project.

**Prerequisites:**

* **HDInsight Tools in Azure Toolkit for IntelliJ**. This is part of the Azure Toolkit for IntelliJ. For more information, see [Installing the Azure Toolkit for IntelliJ](https://docs.microsoft.com/en-us/azure/azure-toolkit-for-intellij-installation).
* **Azure Toolkit for IntelliJ**. Use this to create Spark applications for HDInsight cluster. For more information, follow the instructions at [Use Azure Toolkit for IntelliJ to create Spark applications for HDInsight cluster](https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-apache-spark-intellij-tool-plugin).
* **HDInsight SSH service with username and password management.** For more information, see [Connect to HDInsight (Hadoop) using SSH](https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-hadoop-linux-use-ssh-unix) and [Use SSH Tunneling to access Ambari web UI, JobHistory, NameNode, Oozie, and other web UIs](https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-linux-ambari-ssh-tunnel). 
 

## Create a Spark Scala application and configure it for remote debugging
1. Launch IntelliJ IDEA, and then create a project. In the new project dialog box, make the following choices, and then click **Next**. This article uses a **Spark on HDInsight Cluster Run Sample (Scala)** as an example.

     ![Create a debug project](./media/hdinsight-apache-spark-intellij-tool-debug-remotely/hdinsight-create-projectfor-debug-remotely.png)
   - In the left pane, select **HDInsight**.
   - In the right pane, select a Java or Scala template based on your preference. Choose between the following options: 
      - **Spark on HDInsight (Scala)**
      - **Spark on HDInsight (Java)**
      - **Spark on HDInsight Cluster Run Sample (Scala)**
   - Build tool: Scala project creation wizard support Maven or SBT managing the dependencies and building for scala project. You select one according need.
2. In the next window, provide the project details.

   ![Selecting the Spark SDK](./media/hdinsight-apache-spark-intellij-tool-debug-remotely/hdinsight-new-project.png)
   - Provide a project name and project location.
   - For **Project SDK**, use **Java 1.8** for **Spark 2.x** cluster or **Java 1.7** for **Spark 1.x** cluster.
   - For **Spark Version**, the Scala project creation wizard integrates the correct version for Spark SDK and Scala SDK. If the spark cluster version is lower than 2.0, choose **Spark 1.x**. Otherwise, select **Spark 2.x.** This article uses **Spark 2.0.2 (Scala 2.11.8)** as an example.
3. Select **src** > **main** > **scala** to open your code in the project. This article uses the **SparkCore_wasbloTest** script as an example.
4. To access the **Edit Configurations** menu, select the icon in the upper right corner. From this menu, you can create or edit the configurations for remote debugging.

   ![Edit configurations](./media/hdinsight-apache-spark-intellij-tool-debug-remotely/hdinsight-edit-configurations.png) 
5. In the **Run/Debug Configurations** window, click the plus sign (**+**). Then select the **Submit Spark Job** option.

   ![Edit configurations](./media/hdinsight-apache-spark-intellij-tool-debug-remotely/hdinsight-add-new-Configuration.png)
6. Enter information for **Name**, **Spark cluster**, and **Main class name**. Then select **Advanced configuration**. 

   ![Run debug configurations](./media/hdinsight-apache-spark-intellij-tool-debug-remotely/hdinsight-run-debug-configurations.png)

   We support debugging of both driver and executor code depending on where the breakpoint is set.
   - **numExecutors**: The default value is 5 executors. For better results, do not set the value higher than 3.
7. In the **Spark Submission Advanced Configuration** dialog box, select **Enable Spark remote debug**. Enter the SSH user name or password, or use a private key file. To save it, select **Ok**.

   ![Enable Spark remote debug](./media/hdinsight-apache-spark-intellij-tool-debug-remotely/hdinsight-enable-spark-remote-debug.png)
8. The configuration is now saved with the name you provided. To view the configuration details, select the configuration name. To make changes, select **Edit Configurations**. 
9. After you complete the configurations settings, you can run the project against the remote cluster or perform remote debugging.

## Learn how to perform remote debugging
### Scenario 1: Perform remote debugging
In this section, we show you how to debug driver and executors.

    import org.apache.spark.{SparkConf, SparkContext}

    object LogQuery {
      val exampleApacheLogs = List(
        """10.10.10.10 - "FRED" [18/Jan/2013:17:56:07 +1100] "GET http://images.com/2013/Generic.jpg
          | HTTP/1.1" 304 315 "http://referall.com/" "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1;
          | GTB7.4; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.04506.648; .NET CLR
          | 3.5.21022; .NET CLR 3.0.4506.2152; .NET CLR 1.0.3705; .NET CLR 1.1.4322; .NET CLR
          | 3.5.30729; Release=ARP)" "UD-1" - "image/jpeg" "whatever" 0.350 "-" - "" 265 923 934 ""
          | 62.24.11.25 images.com 1358492167 - Whatup""".stripMargin.lines.mkString,
        """10.10.10.10 - "FRED" [18/Jan/2013:18:02:37 +1100] "GET http://images.com/2013/Generic.jpg
          | HTTP/1.1" 304 306 "http:/referall.com" "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1;
          | GTB7.4; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.04506.648; .NET CLR
          | 3.5.21022; .NET CLR 3.0.4506.2152; .NET CLR 1.0.3705; .NET CLR 1.1.4322; .NET CLR
          | 3.5.30729; Release=ARP)" "UD-1" - "image/jpeg" "whatever" 0.352 "-" - "" 256 977 988 ""
          | 0 73.23.2.15 images.com 1358492557 - Whatup""".stripMargin.lines.mkString
      )
      def main(args: Array[String]) {
        val sparkconf = new SparkConf().setAppName("Log Query")
        val sc = new SparkContext(sparkconf)
        val dataSet = sc.parallelize(exampleApacheLogs)
        // scalastyle:off
        val apacheLogRegex =
          """^([\d.]+) (\S+) (\S+) \[([\w\d:/]+\s[+\-]\d{4})\] "(.+?)" (\d{3}) ([\d\-]+) "([^"]+)" "([^"]+)".*""".r
        // scalastyle:on
        /** Tracks the total query count and number of aggregate bytes for a particular group. */
        class Stats(val count: Int, val numBytes: Int) extends Serializable {
          def merge(other: Stats): Stats = new Stats(count + other.count, numBytes + other.numBytes)
          override def toString: String = "bytes=%s\tn=%s".format(numBytes, count)
        }
        def extractKey(line: String): (String, String, String) = {
          apacheLogRegex.findFirstIn(line) match {
            case Some(apacheLogRegex(ip, _, user, dateTime, query, status, bytes, referer, ua)) =>
              if (user != "\"-\"") (ip, user, query)
              else (null, null, null)
            case _ => (null, null, null)
          }
        }
        def extractStats(line: String): Stats = {
          apacheLogRegex.findFirstIn(line) match {
            case Some(apacheLogRegex(ip, _, user, dateTime, query, status, bytes, referer, ua)) =>
              new Stats(1, bytes.toInt)
            case _ => new Stats(1, 0)
          }
        }
        
        dataSet.map(line => (extractKey(line), extractStats(line)))
          .reduceByKey((a, b) => a.merge(b))
          .collect().foreach{
          case (user, query) => println("%s\t%s".format(user, query))}

        sc.stop()
      }
    }

1. Set up breaking points, and then select the **Debug** icon.

    ![Click the debug icon](./media/hdinsight-apache-spark-intellij-tool-debug-remotely/hdinsight-debug-icon.png)
2. When the program execution reaches the breaking point, you see a **Driver** tab and two **Executor** tabs in the **Debug** pane.   Click the **Resume Program** icon to continue running the code, which then reaches the next breakpoint and focuses on the corresponding **Executor** tab. You can review the execution logs in the corresponding **Console** tab. 

   ![Debugging tab](./media/hdinsight-apache-spark-intellij-tool-debug-remotely/hdinsight-debugger-tab.png)

### Scenario 2: Perform remote debugging and bug fixing
In this section, we show you how to dynamically update the variable value by using the IntelliJ debugging capability for a simple fix. For the following code example, an exception is thrown because the target file already exists.
  
        import org.apache.spark.SparkConf
        import org.apache.spark.SparkContext

        object SparkCore_WasbIOTest {
          def main(arg: Array[String]): Unit = {
            val conf = new SparkConf().setAppName("SparkCore_WasbIOTest")
            val sc = new SparkContext(conf)
            val rdd = sc.textFile("wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")

            // Find the rows that have only one digit in the sixth column.
            val rdd1 = rdd.filter(s => s.split(",")(6).length() == 1)

            try {
              var target = "wasb:///HVACout2_testdebug1";
              rdd1.saveAsTextFile(target);
            } catch {
              case ex: Exception => {
                throw ex;
              }
            }
          }
        }


#### To perform remote debugging and bug fixing
1. Set up two breaking points, and then click the **Debug** icon to start the remote debugging process.

2. The code stops at the first breaking point, and the parameter and variable information are shown in the **Variable** window. 

3. Click the **Resume Program** icon to continue. The code stops at the second point. The exception is being caught as expected.

  ![Throw error](./media/hdinsight-apache-spark-intellij-tool-debug-remotely/hdinsight-throw-error.png) 

4. Click the **Resume Program** icon again. The **Console** panel displays a job run failed error.

  ![Error submission](./media/hdinsight-apache-spark-intellij-tool-debug-remotely/hdinsight-error-submission.png) 

5. To dynamically update the variable value using the IntelliJ debugging capability, select **Debug** again. The variable windows appears again. 

6. Right-click the target on the **Debug** tab, and then select **Set Value**. Next, input a new value for the variable. Then select **Enter** to save the value. 

  ![Set value](./media/hdinsight-apache-spark-intellij-tool-debug-remotely/hdinsight-set-value.png) 

7. Click the **Resume Program** icon to continue running the program. This time, the exception is not being caught. You can see that the project runs successfully without any exceptions.

  ![Debug without exception](./media/hdinsight-apache-spark-intellij-tool-debug-remotely/hdinsight-debug-without-exception.png)

## <a name="seealso"></a>See also
* [Overview: Apache Spark on Azure HDInsight](hdinsight-apache-spark-overview.md)

### Demo
* Create Scala Project (Video): [Create Spark Scala Applications](https://channel9.msdn.com/Series/AzureDataLake/Create-Spark-Applications-with-the-Azure-Toolkit-for-IntelliJ)
* Remote Debug (Video): [Use Azure Toolkit for IntelliJ to debug Spark applications remotely on HDInsight Cluster](https://channel9.msdn.com/Series/AzureDataLake/Debug-HDInsight-Spark-Applications-with-Azure-Toolkit-for-IntelliJ)

### Scenarios
* [Spark with BI: Perform interactive data analysis using Spark in HDInsight with BI tools](hdinsight-apache-spark-use-bi-tools.md)
* [Spark with Machine Learning: Use Spark in HDInsight for analyzing building temperature using HVAC data](hdinsight-apache-spark-ipython-notebook-machine-learning.md)
* [Spark with Machine Learning: Use Spark in HDInsight to predict food inspection results](hdinsight-apache-spark-machine-learning-mllib-ipython.md)
* [Spark Streaming: Use Spark in HDInsight for building real-time streaming applications](hdinsight-apache-spark-eventhub-streaming.md)
* [Website log analysis using Spark in HDInsight](hdinsight-apache-spark-custom-library-website-log-analysis.md)

### Create and run applications
* [Create a standalone application using Scala](hdinsight-apache-spark-create-standalone-application.md)
* [Run jobs remotely on a Spark cluster using Livy](hdinsight-apache-spark-livy-rest-interface.md)

### Tools and extensions
* [Use HDInsight Tools in Azure Toolkit for IntelliJ to create and submit Spark Scala applicatons](hdinsight-apache-spark-intellij-tool-plugin.md)
* [Use Azure Toolkit for IntelliJ to debug applications remotely on HDInsight Spark through VPN](hdinsight-apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [Use HDInsight Tools for IntelliJ with Hortonworks Sandbox](hdinsight-tools-for-intellij-with-hortonworks-sandbox.md)
* [Use HDInsight Tools in Azure Toolkit for Eclipse to create Spark applications](hdinsight-apache-spark-eclipse-tool-plugin.md)
* [Use Zeppelin notebooks with a Spark cluster on HDInsight](hdinsight-apache-spark-zeppelin-notebook.md)
* [Kernels available for Jupyter notebook in Spark cluster for HDInsight](hdinsight-apache-spark-jupyter-notebook-kernels.md)
* [Use external packages with Jupyter notebooks](hdinsight-apache-spark-jupyter-notebook-use-external-packages.md)
* [Install Jupyter on your computer and connect to an HDInsight Spark cluster](hdinsight-apache-spark-jupyter-notebook-install-locally.md)

### Manage resources
* [Manage resources for the Apache Spark cluster in Azure HDInsight](hdinsight-apache-spark-resource-manager.md)
* [Track and debug jobs running on an Apache Spark cluster in HDInsight](hdinsight-apache-spark-job-debugging.md)
 