##### Run compose file and verify :-

1. Run using `docker compose up -d`
2. Verify running containers `docker compose ps`
3. Verify accessibility to container endpoints.

##### Instrument application with agent

1.  Download the java agent from the below url:- https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/latest/download/opentelemetry-javaagent.jar
2.  Move the agent to /var/lib/jenkins/
3.  Change ownership to jenkins, `sudo chown jenkins:jenkins /var/lib/jenkins/opentelemetry-javaagent.jar` 
4. Add below block to your jenkins pipeline before docker build step.
```groovy
stage("ADD OTL AGENT") {
        steps {
            sh "cp /var/lib/jenkins/opentelemetry-javaagent.jar ."
        }
    }
```

5. Make below changes to your dockerfile
```Dockerfile
#Add the following env variable.

ENV OTEL_SERVICE_NAME="service_name" 
ENV OTEL_RESOURCE_ATTRIBUTES="service=service_name,env=dev"
ENV OTEL_EXPORTER_OTLP_ENDPOINT="http://server_ip:4317"

#Add the agent to your image

ADD opentelemetry-javaagent.jar path


#Instrument the applciation using the below command

"-javaagent:/path/opentelemetry-javaagent.jar"
"-Dotel.exporter.otlp.protocol=grpc"

#FOR EXAMPLE

CMD ["/usr/bin/java",
"-Dserver.address=0.0.0.0",
"-javaagent:/path/opentelemetry-javaagent.jar",
"-Dotel.exporter.otlp.protocol=grpc",
"-jar",
"-Dfile.encoding=UTF-8",
"/path/to/application/app.jar"]

```

That's it, just build!