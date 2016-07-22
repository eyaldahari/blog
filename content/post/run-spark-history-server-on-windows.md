At the time these lines are written Apache Spark do not contain an out of the box script to run its History Server on Windows.
The History Server is usually needed when after Spark has finished its work. 
Many times you need it after running spark-submit. 
When spark-submit is running, you can monitor spark activity through the active monitor on port 4040.
But many times you want to monitor after the fact. 
For this use-case and many others you’ll need the Spark History Server.

> Spark’s Standalone Mode cluster manager also has its own web UI. If an application has logged events over the course of its lifetime, then the Standalone master’s web UI will automatically re-render the application’s UI after the application has finished.
> If Spark is run on Mesos or YARN, it is still possible to reconstruct the UI of a finished application through Spark’s history server, provided that the application’s event logs exist.


The default Spark installation comes with built-in scripts: _start-history-server.sh_ and _stop-history-server.sh._
On Windows you’ll need to run the _.cmd_ files of Spark not _.sh_.

I was going through Sparks installation codes and according to what I saw, there is no _.cmd_ script for Spark History Server. 
So basically it needs to be run manually.

I have followed the _start-history-server.sh_ Linux script and in order to run it manually on Windows you’ll need to take
the following steps:

All history server configurations should be set at: _spark-defaults.conf_ file (remove _.template_ suffix from the file).
You should go to spark config directory and add the _spark.history.*_ and _spark.eventLog.*_ configurations 
to _%SPARK_HOME%/conf/spark-defaults.conf_. As follows:

```
spark.eventLog.enabled        true
spark.history.fs.logDirectory file:///c:/logs/dir/path
```

After configuration is finished run the following command from _%SPARK_HOME%_

```cmd
bin\spark-class.cmd org.apache.spark.deploy.history.HistoryServer
```

The output should look something like that:

```
16/07/22 18:51:23 INFO Utils: Successfully started service on port 18080.
16/07/22 18:51:23 INFO HistoryServer: Started HistoryServer at http://10.0.240.108:18080
16/07/22 18:52:09 INFO ShutdownHookManager: Shutdown hook called
```

In order to quit, use ctrl+c

Now you have history server working on the following URL (use the URLfrom your command output):

```
http://10.0.240.108:18080
```

Hope that it helps! :-)

Follow me on:

[Medium](https://medium.com/@eyaldahari) | [Twitter](https://twitter.com/EyalDahari) | [Linkedin](https://www.linkedin.com/in/eyaldahari) | [Stackoverflow](http://stackexchange.com/users/7651751/e-dahari?tab=activity) | [GitHub](https://github.com/eyaldahari)
