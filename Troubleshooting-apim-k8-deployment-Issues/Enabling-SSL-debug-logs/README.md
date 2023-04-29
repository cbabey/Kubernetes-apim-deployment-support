# How to enable the ‘SSL debug’ logs for the WSO2 products which are running on the docker containers

You might require to enable SSL debug logs in the WSO2 products in order to troubleshoot some SSL communication-related issues. If your deployment is not a container-based deployment, the following command can be easily used to enable SSL handshake debug logs at the startup of the wso2 product instance. You can either provide this JVM parameter along with the startup script flags or add this JVM parameter to the startup script.
```sh
./api-manager.sh -Djavax.net.debug=ssl:handshake
```
If you are required to capture the complete debug logs other than the SSL handshake-related logs, you can use the following command.

```sh
./api-manager.sh -Djavax.net.debug=ssl:all
```

In the containerized APIM deployment, you can define the above JVM parameter in the ```docker-entrypoint.sh```. However, it will be required additional effort of rebuilding the docker image by updating the default ```docker-entrypoint.sh``` or will be required to have a config map to mount the updated docker-entrypoint.sh into the APIM pod if it is k8 based deployment. 

You can Inject the JVM parameter to the APIM server by either using  ```Shell script arguments ``` or using the  ```“JAVA_OPTS” ``` environment variable. 

## Shell Script arguments

The default  ```docker-entrypoint.sh``` file is implemented to accept the shall script argument to start the server, it can be used to pass the JVM parameters during the server startup. 

```
# start WSO2 Carbon server
echo "Start WSO2 Carbon server" >&2
if [[ -z "${PROFILE_NAME}" ]]
then
  # start the server with the provided startup arguments
  sh ${WSO2_SERVER_HOME}/bin/api-manager.sh "$@"
else
  # start the server with the specified profile and provided startup arguments
  sh ${WSO2_SERVER_HOME}/bin/api-manager.sh -Dprofile=${PROFILE_NAME} "$@"
fi
```
You can provide the JVM parameters as below:

```sh
docker run -it -p 8280:8280 -p 8243:8243 -p 9443:9443  --name api-manager wso2/wso2am:4.1.0-multiarch "-Djavax.net.debug=ssl"
```
In a Kubernetes-based deployment, you can provide the argument that needs to inject into the startup command by using the ‘args’ variable as shown below. 

```
    spec:
      containers:
        - name: wso2am
          image: wso2/wso2am:4.0.0
          imagePullPolicy: Always
          args: ["-Djavax.net.debug=ssl:all"]
```

## ```“JAVA_OPTS”``` environment variable. 


JAVA_OPTS is an environment variable that you can set to pass custom settings to the Java Virtual Machine (JVM) that runs Liquibase.

You can define JAVA_OPTS in the environment as below in order to enable the SSL debug logs.
```sh
docker run -it -p 8280:8280 -p 8243:8243 -p 9443:9443  --env JAVA_OPTS="-Djavax.net.debug=ssl" --name api-manager wso2/wso2am:4.1.0-multiarch 
```
In a k8 deployment, you can define the environment variable as below.

```
       env:
            - name: NODE_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            - name: JVM_MEM_OPTS
              value: "-Xms1024m -Xmx1024m"
            - name: JAVA_OPTS
              value: "-Djavax.net.debug=ssl:all"

```
