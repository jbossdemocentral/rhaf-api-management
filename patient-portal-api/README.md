# Running the Patient Portal API

## Installing Camel JBang

### Prerequisites

- JBang must be installed on your machine. 

See [instructions](https://www.jbang.dev/download/) on how to download and install the JBang.After the JBang is installed, you can verify JBang is working by executing the following command from a command shell:

```shell
jbang version
```

This outputs the version of installed JBang.

### Procedure

1. Run the following command to install the Camel JBang application:

    ```shell
    jbang app install camel@apache/camel
    ```

    This installs the Apache Camel as the camel command within JBang. This means that you can run Camel from the command line by just executing camel command.

2. Configure the Red Hat Maven repository

   ```shell
   camel config set repos=https://maven.repository.redhat.com/ga
   ```

3. Configure the Red Hat build of Apache Camel version 4

   ```shell
   camel version set 4.0.0.redhat-00027
   ```


## Creating the Camel route

1. Create a new basic route from the OpenAPI document using the camel command. 

    ```shell
    camel generate rest -i patient-portal-api.json -o patient-portal-api.camel.yaml --routes -t yaml
    ```

    This creates the file `patient-portal-api.camel.yaml` (in the current directory) with a route skeleton.

2. Add the rest of the route implementation to call the database and query for doctors. 

    ```yaml
    - route:
        from:
          uri: direct
          parameters:
            name: doctorsGET
          steps:
            - setBody:
                expression:
                  simple:
                    expression: select * from doctors order by id
            - to:
                uri: jdbc
                parameters:
                  dataSourceName: myDataSource
            - to:
                uri: kamelet:hoist-field-action
                parameters:
                  field: doctors
    - beans:
        - name: myDataSource
          properties:
            databaseName: patient_portal
            user: patient_portal
            password: secret
          type: org.postgresql.ds.PGSimpleDataSource
    ```

    

## Running the Camel route

To run the route, execute the following command:

```shell 
camel run patient-portal.camel.yaml --open-api patient-portal-api.json --deps org.postgresql:postgresql:42.7.0
```

In this command we are telling camel to run our yaml file with the route and use the provided OpenAPI as the REST DSL configuration. As we are running the Camel Main engine, we need to add the dependency to the database driver so Camel can connec to the database.

You can use HTTPie to test the API:

```shell
http :8080/doctors
```

You should see an output similar to the following:

```shell
HTTP/1.1 200 OK
Accept: */*
Accept-Encoding: gzip, deflate
Content-Type: application/json
User-Agent: HTTPie/3.2.2
transfer-encoding: chunked
```

With the following payload as JSON.

```json
{
    "doctors": [
        {
            "email": "hawkeye@example.net",
            "id": 1,
            "name": "Benjamin Pierce",
            "phone": "555-555-1001"
        },
        {
            "email": "gates@example.net",
            "id": 2,
            "name": "Beverly Crusher",
            "phone": "555-555-1002"
        },
        {
            "email": "neil@example.net",
            "id": 3,
            "name": "Doogie Howser",
            "phone": "555-555-1003"
        },
        {
            "email": "bones@example.net",
            "id": 4,
            "name": "Leonard McCoy",
            "phone": "555-555-1004"
        },
        {
            "email": "drmike@example.net",
            "id": 5,
            "name": "Michaela Quinn",
            "phone": "555-555-1005"
        },
        {
            "email": "chief@example.net",
            "id": 6,
            "name": "Miranda Bailey",
            "phone": "555-555-1006"
        }
    ]
}
```

### Export to Quarkus project

If you want to have a more complex integration you can generate a Quarkus project using the Camel route YAML file.

```shell
camel export --runtime=quarkus --gav=com.redhat.appdev:patient-portal:1.0.0 --directory patient-portal2 --deps org.postgresql:postgresql:42.5.4 --fresh 
```

This will generate a full blown Quarkus project. You can then add the required changes with everything configured to run your API implementation. 

For example, you can use the Quarkus JDBC extension instead of using the direct dependency on the database driver and configure everything from the `application.properties` file.