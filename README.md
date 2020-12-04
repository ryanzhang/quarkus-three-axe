# Demostrate three axes of Quarkus
## Prepare/setup
* Please use jdk11
* Install Graalvm if you want to try native binary demo, See https://quarkus.io/guides/building-native-image

# Clone all submodules
```bash
git clone https://github.com/ryanzhang/quarkus-three-axe.git
cd quarkus-three-axe && git submodule update --init --recursive
```

# 1st Axe - Quarkus Build time boot
Quarkus uses build time boot idea to speed your apps startup time. It matters because:
 * Even in JVM mode, your apps startup time optimized x times (x ~ in 2-4)
 * It's essential to enable Java AOT works since AOT assume  close world assumption. And in static mode, your apps startup time optimized 100x times

 # Show me the evidence
 * 1. Check classload difference by view *-classload.txt
 * 2. Check the startup log by view *-bootstrap.txt
 * 3. Check the quarkus build log by view quarkus-build.log
 * 4. Check quarkus build class by view quarkus-build-class.txt

 Note that:

 I wrote an article about this in https://dzone.com/articles/build-time-boot-refine-java-framework . So you could find more description on the topic there.

# 2nd Axe - Quarkus Event loop core & Reactive
Quarkus uses vertx  as its http server. Therefore it is different from tradditional RR(Request & Response) thread pool model(ie worker thread), instead Quarkus uses event loop(ie IO thread).

This demo will show:
 * Quarkus make app more responsive
 * Quarkus reactive is easy.
 * Quarkus make app bigger throughouput

## Demo it
Firstly show how worker thread model stucks when huge request hit
```bash
#Suggest run them in different terminal
cd quarkus-reactive
# Start a springboot app 
java -jar server/spring-boot/target/springboot-runner.jar

#(Optional) Open jconsole to monitor the jvm thread pool behaviour
#jconsole

# Start the client, it will seige the apps with 10000 request
cd client && mvn compile exec:java

#Try to access app before the seige stopped 
http :8080/api/greeting

```
Secondly show how event loop handles huge request 
```bash
#Suggest run them in different terminal
cd quarkus-reactive
# Start a springboot app 
java -jar server/quarkus/target/quarkus-runner.jar

#(Optional) Open jconsole to monitor the jvm thread pool behaviour
#jconsole

# Start the client, it will seige the app with 10000 request
cd client && mvn compile exec:java

#Try to access app before the seige stopped 
http :8080/api/greeting

```
You will notice the quarkus is more responsive in the demo.

You can also explore the quarkus src folder for reactive example.

Open a browser to view streaming api request: http://localhost:8080/apis/stream/15

API
 * http://localhost:8080/api/compute -- calculate Pi and output some information in console
 * http://localhost:8080/api/greeting -- print a greeting msg
 * http://localhost:8080/apis/stream/15 -- steaming api show 15 numbers of compute request

Note that:
If you want to tune the springboot tomcat server's thread pool size
*server.tomcat.max-threads=<your value> # Maximum amount of worker threads.*

# 3rd Axe - Quarkus Java AOT
Quarkus can build native executable instead of bytecode jar via Java AOT(ie through graalvm toolset). This dramatically reduce the size of the binary and speed the startup.

This demo will show:
* Quarkus app can scale dramatically
* Quarkus static compile support CRUD scenario easily with optimized libraries

# Demo it
```bash
cd quarkus-todo-app
git checkout postgresql
mvn clean package -Pnative -DskipTests

#Start database
./start-database.sh

#This only apply to fish shell
for i in (seq 8000 8250); QUARKUS_HTTP_PORT=$i ../target/todo-backend-1.0-SNAPSHOT-runner > /tmp/todo-app-$i.log&; end

#watch the process 'top' data
top -bc |grep "todo-backend"

# try
http :8000/hello

#check rss for specific pid
ps -o pid,rss yourprocessid

```
