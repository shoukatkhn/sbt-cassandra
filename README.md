sbt-cassandra-plugin
====================

This is a work in progress project.  The goal is to allow launching [Cassandra](http://cassandra.apache.org) during tests, and test your application against it (much like the [plugin for maven](http://mojo.codehaus.org/cassandra-maven-plugin)).
at this pre-mature phase, only the very basic functionality works. API is not final, and might (probably will) change down the road.
However, the plugin is already usable as is.

## Installation ##
Add the following to your `project/plugins.sbt` file:
```scala
addSbtPlugin("com.github.hochgi" % "sbt-cassandra-plugin" % "0.1-SNAPSHOT")
```
Until i'll get this plugin hosted, you can build it yourself, and use `sbt publish-local` to have it available in your local `~/.ivy`.

## Usage ##
### Basic: ###
```scala
import com.github.hochgi.sbt.cassandra._
    
seq(CassandraPlugin.cassandraSettings:_ *)
   
test in Test <<= (test in Test).dependsOn(startCassandra)
``` 
### Advanced: ##
To choose a specific version of cassandra (default is 2.0.6), you can use:
```scala
cassandraVersion := "2.0.6"
```
Now, assuming your project depends on cassandra (well duh...), you probably have your dependencies declared like:
```scala
libraryDependencies += "org.apache.cassandra" % "apache-cassandra" % "2.0.6"
```
~~a nifty tweak to set the correct version automatically, could look something like:~~(currently not working, due to a cyclic reference)
```scala
cassandraVersion <<= (libraryDependencies) (_.collect{case ModuleID("org.apache.cassandra","apache-cassandra",version,_,_,_,_,_,_,_,_) => version}.head)
```
to stop cassandra after the tests finished and clean the data directory, you could use:
```scala
testOptions in Test <+= (cassandraPid, cassandraHome) map {
  case (pid, cassHome) => Tests.Cleanup(() => {
    s"kill $pid" !
    val data = (cassHome / "data").getAbsolutePath
    s"rm -r $data" !
  })
}
```
to use special configuration files suited for your use case, use:
```scala
cassandraConfigDir := "/path/to/your/conf/dir"
```
to intialize cassandra with your custom cassandra-cli commands, use:
```scala
cassandraCliInit := "/path/to/cassandra-cli/commands/file"
```
~~to intialize cassandra with your custom cql commands, use:~~(not implemented yet)
```scala
cassandraCqlInit := "/path/to/cassandra-cql/commands/file"
```

##### IMPORTANT NOTES #####
Regarding the CQL commands, the file must contain `exit;` as the last line.
your file might look something like:
```
create keyspace Data with ... ;
use Data;
create column family MoreData with ... ;
exit;
```
