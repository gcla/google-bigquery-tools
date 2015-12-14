# Introduction #

[Maven](http://maven.apache.org/) is a build and dependency manager for Java projects.  The BigQuery client libraries are available as a Maven dependency.  This page documents how to quickly add the BigQuery client libraries to an existing Maven project.

# Details #

Add the following repository to your `<repositories>` section:
```
  <repositories>
    ...
    <repository>
      <id>google-api-services</id>
      <url>http://mavenrepo.google-api-java-client.googlecode.com/hg</url>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
    </repository>
  </repositories>
```

Then add a dependency on the BigQuery API and supporting libraries to your `<dependencies>` block:
```
  <dependencies>
    ...
    <dependency>
      <groupId>com.google.apis</groupId>
      <artifactId>google-api-services-bigquery</artifactId>
      <version>v2-1.3.3-beta</version>
    </dependency>
    <dependency>
      <groupId>com.google.oauth-client</groupId>
      <artifactId>google-oauth-client</artifactId>
      <version>1.6.0-beta</version>
    </dependency>
  </dependencies>
```

# Next Steps #

See [BigQuerySample.java](http://code.google.com/p/google-bigquery-tools/source/browse/samples/java/cmdline/src/main/java/com/google/api/services/samples/bigquery/cmdline/BigQuerySample.java) for an example of how to authenticate and issue queries.