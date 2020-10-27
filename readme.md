# Process Servicecall Exception Handling

## Process definition

![ServiceCall](./src/main/resources/com/redhat/demos/ServiceCall-svg.svg)

## Use the [SignallingTaskHandlerDecorator](https://github.com/kiegroup/jbpm/blob/master/jbpm-bpmn2/src/main/java/org/jbpm/bpmn2/handler/SignallingTaskHandlerDecorator.java)
jBPM comes with a special Work Item handler Wrapper that can be used to decorate the [REST Service Task](https://github.com/kiegroup/jbpm/blob/master/jbpm-workitems/jbpm-workitems-rest/src/main/java/org/jbpm/process/workitem/rest/RESTWorkItemHandler.java) to send a Signal to the process instance when an exception occurs. To use it change the [kie-deployment-descriptor.xml](src/main/resources/META-INF/kie-deployment-descriptor.xml) to register it:

```xml
     <work-item-handlers>
        <work-item-handler>
            <resolver>mvel</resolver>
            <identifier>new org.jbpm.bpmn2.handler.SignallingTaskHandlerDecorator(new org.jbpm.process.workitem.rest.RESTWorkItemHandler(classLoader), "Error-serviceErrorSignal")</identifier>
            <parameters/>
            <name>Rest</name>
        </work-item-handler>
    </work-item-handlers>
```
> Note: make sure you set the `HandleResponseErrors` parameter to `true` in the *Rest Service Task* (`RESTWorkItemHandler`). This parameter is used to properly handle the exception. Otherwise it will just log the Http request error and return the error code in the `Result` output parameter.


## Testing
After building and deploying your kjar you can test it via Rest API with:

```bash
curl --location --request POST 'http://localhost:8080/kie-server/services/rest/server/containers/kie-process-template/processes/ServiceCall/instances' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic <base64 credentials>' \
--data-raw '{
    "serviceUrl": "https://api.covid19api.com/summary/",
    "serviceRequestData": {
        "java.util.HashMap": {
            "data": "test..."
        }
    }
}'
```