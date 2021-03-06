## The simple JMH-based benchmark of Spring MVC vs EJB vs CDI RESTful web-service implementations

A very popular delusion in the Java world is the **Spring Framework** provides better performance than good old **Enterprise Java Beans**. The
best approach to compare technologies is just to get a benchmark. This repository contains three implementations of the same simple
RESTful web-service based on **Spring MVC**, **EJB** and **CDI**, respectively. The service design is the following: resource providers are injected 
into a "service" class and the "service" is injected into a controller. This is the usual design of an enterprise application: data-access 
objects are injected into services and the services are injected into facades or controllers. Different technologies are used for *Dependency Injection* 
as well as *RESTfulization*.

Let's have a look at the EJB version.

Resource:

```java
@Stateless
public class EJBResourceA {

    public String message() {
        return "A#" + System.currentTimeMillis();
    }
}
```

Service:

```java
@Stateless
public class MessageService {

    @EJB
    private EJBResourceA aresource;
    
    @EJB
    private EJBResourceB bresource;
    
    public String message() {
        return aresource.message() + bresource.message();
    }
}

``` 

REST Controller:

```java
@Stateless
@Path("/")
public class MessageController {

    @EJB
    private MessageService service;
    
    @GET
    @Path("/message")
    @Produces({"text/plain"})
    public String message() {
        return service.message();
    }
}

```

The service just returns a string built by some letters and current time concatination:

![Services return the following result](http://1.bp.blogspot.com/-2ZHUvA7OSoo/Vn1t0F9m9yI/AAAAAAAADtk/6SWbm_pKXCQ/s1600/service-result.png)

**[JMH](http://openjdk.java.net/projects/code-tools/jmh/ "OpenJDK JMH Tool")** is a Java harness for building, running, and analysing 
nano/micro/milli/macro benchmarks written in Java and other languages targetting the JVM.

The benchmark code contained by the [file](rest-benchmark/src/main/java/psamolysov/demo/restws/benchmark/RestImplementationsBenchmark.java).

Now it is time to destroy the myth!

***

### Benchmark running

```
# java -jar rest-benchmark/target/rest-benchmarks.jar ".*Benchmark" -f 4 -wi 20 -i 20 -t 4 
-si true -gc true
```

Where:

- `-f [int]` - How many times to forks a single benchmark.
- `-wi <int>` - Number of warmup iterations to do.
- `-i <int>` - Number of measurement iterations to do.
- `-si [bool]` - Synchronize iterations?
- `-t <int>` - Number of worker threads to run with.
- `-gc [bool]` - Should JMH force GC between iterations?


Allowed parameters:

- `server` - application server's host, `localhost` is the default value.
- `port` - application server's port.
- `path` - path for the resource. Every path is relative to the `/api/` catalog.
- `implementation` - technology used for the RESTful service implementation. The passed values are just substituted
   in the `rest-ws-{implementation}` template for web-applications context-roots.

Example:

```
# java -jar rest-benchmark/target/rest-benchmarks.jar ".*Benchmark" -f 4 -wi 20 -i 20 -t 4 
-si true -gc true -p port=14633 -p implementation=ejb,cdi
```
***

### Results

### Summary Diagram

The test results data - troughput - are normalized (the Spring Framework is 100%, **bigger value is better**):

![Summary Diagram](http://2.bp.blogspot.com/-XzR4h_VLOa0/VoEWInXvPSI/AAAAAAAADuc/iKYTeEEKLlk/s1600/ghraphical-results.png)

*EJB* is **up to 15%** faster than the *Spring Framework* while *CDI* is **up to 19%** slower. 


#### WebSphere Application Server 8.5.5.4 for z/OS

<table cellspacing="0" cellpadding="0">
  <tbody>
    <tr>
      <td align="center" width="200"><img alt="Mainframe logo" src="http://2.bp.blogspot.com/-fEALuKC-JGs/Vn1qDftkrhI/AAAAAAAADtU/MJPqzLx5FHk/s1600/zBC12-small.jpg"/></td>
      <td valign="top">IBM Mainframe <a href="http://www-03.ibm.com/systems/z/hardware/zenterprise/zbc12.html" title="IBM zEnterprise Business Class 12">
            zBC 12</a>: 2 General CP + 1 zIIP, 32 GB RAM, z/OS 2.1, IBM J9 VM R26 SR7 based on Oracle 7u55-b13</td>
    </tr>
  </tbody>
</table>

Operations per second, the picture is clickable:

![Mainframe results][zBC results]

[zBC results]: http://1.bp.blogspot.com/-VUcmJ1MQ0uw/Vn2VAIpan6I/AAAAAAAADuA/XdHP4UeWkYQ/s1600/zBC12-result-t-4.png


#### WebSphere Application Server 8.5.5.8 Network Deployment

<table cellspacing="0" cellpadding="0">
  <tbody>
    <tr>
      <td align="center" width="200"><img alt="Mainframe logo" src="http://2.bp.blogspot.com/-fEALuKC-JGs/Vn1qDftkrhI/AAAAAAAADtU/MJPqzLx5FHk/s1600/zBC12-small.jpg"/></td>
      <td valign="top">IBM Mainframe <a href="http://www-03.ibm.com/systems/z/hardware/zenterprise/zbc12.html" title="IBM zEnterprise Business Class 12">
            zBC 12</a>: 2 IFL, 16 GB RAM, SUSE Linux Enterprise Server 12 s390x, IBM J9 VM R27 SR3 based on Oracle 7u85-b15</td>
    </tr>
  </tbody>
</table>

Operations per second, the picture is clickable:

![Mainframe Linux for z Systems results][zBC linux results]

[zBC linux results]: http://4.bp.blogspot.com/-Q_LOdUS0U1c/Vn2R6VJ_OII/AAAAAAAADt0/QMbyPLkwVLA/s1600/zBC12-result-linux-t-4.png


#### WebSphere Application Server 8.5.5.8 Network Deployment

<table cellspacing="0" cellpadding="0">
  <tbody>
    <tr>
      <td align="center"><img alt="Lenovo logo" src="http://2.bp.blogspot.com/-L65s__CNEdk/Vn1qDWGmO7I/AAAAAAAADtQ/-3NhFdheggo/s1600/Lenovo-small.jpg"/></td>
      <td valign="top">Lenovo ThinkPad T440: 2 Intel i5-4300U @ 1.90 GHz, 8 GB RAM, Windows 7 x64, IBM J9 VM R27 SR3 based 
        on Oracle 7u85-b15</td>
    </tr>
  </tbody>
</table>

Operations per second, the picture is clickable:

![Lenovo results][Lenovo results]

[Lenovo results]: http://2.bp.blogspot.com/-pGUHfVt3rzw/VoEUKd75DMI/AAAAAAAADuQ/Jpoh7WDyGbA/s1600/Lenovo-result-t-4.png


Which result have you gone? Please, share it with me: <samolisov@gmail.com>
