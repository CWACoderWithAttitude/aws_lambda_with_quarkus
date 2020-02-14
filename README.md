# Quarkus - Amazon Lambda with RESTEasy, Undertow, or Vert.x Web

I'm a huge fan of [quarkus-framework](https://quarkus.io/). And i like serverless coding pretty much.
This is a comprehensive description of the steps i took to make it work.
Luckily the geniuses over at [Quarkus.io](https://quarkus.io/) have published a great documentation.    
So, this describes ma followinbg the description [Quarkus - Amazon Lambda with RESTEasy, Undertow, or Vert.x Web](https://quarkus.io/guides/amazon-lambda-http).   
Filling in the gaps was fun. Great fun - see for yourself:   

## Compile

Generated, `mvn install`. Guess what.    
It wouldn't even compile :-(   
Since this module is under development i had to adjust the version number.
Consulting [Mvn Repository](https://mvnrepository.com/artifact/io.quarkus/quarkus-core-deployment) showed i should probably update to quarkus v1.2.0.Final.   
`mvn install` worked.   

## Local test

I tried to fire up the lamba code locally in a mocked lambda env:
```bash
$> sam local start-api --template sam.jvm.yaml
```

Next error showed up:

```bash
Error: [InvalidResourceException('Quarkus_lambdaFunction', 'Logical ids must be alphanumeric.')] ('Quarkus_lambdaFunction', 'Logical ids must be alphanumeric.')
```

I appears aws have changed the naming policy and forbids hyphens and underscores.  [Post by antonnaws, Jan 31, 2020 7:21 AM](https://forums.aws.amazon.com/thread.jspa?threadID=312718)

Removing the underscore frm the functins name saves your day:
```
$> sam local start-api --template sam.jvm.yaml
Mounting QuarkusLambdaFunction at http://127.0.0.1:3000/{proxy+} [DELETE, GET, HEAD, OPTIONS, PATCH, POST, PUT]
You can now browse to the above endpoints to invoke your functions. You do not need to restart/reload SAM CLI while working on your functions, changes will be reflected instantly/automatically. You only need to restart SAM CLI if you update your AWS SAM template
2020-02-13 22:01:30  * Running on http://127.0.0.1:3000/ (Press CTRL+C to quit)
```

Test correct function by calling the REST endpoint:
```bash
$> http http://127.0.0.1:3000/hello
HTTP/1.0 200 OK
Content-Length: 11
Content-Type: text/plain;charset=UTF-8
Date: Thu, 13 Feb 2020 21:09:50 GMT
Server: Werkzeug/0.16.0 Python/3.7.6

hello jaxrs
```

Here I use [httpie](https://httpie.org/) to call the rest endpoint. But curl / firefox etc. would also get the job done.

## Deploy to AWS

Deploying required my to update my `sam-cli` and `aws-cli` tools.

```bash
$> brew upgrade aws/tap/aws-sam-cli
...
==> Upgrading 1 outdated package:
aws/tap/aws-sam-cli 0.40.0 -> 0.41.0
==> Upgrading aws/tap/aws-sam-cli
...
$> echo 'export PATH="/usr/local/opt/sqlite/bin:$PATH"' >> ~/.zshrc
$> source ~/.zshrc
$> brew upgrade awscli
...
==> Upgrading 1 outdated package:
awscli 1.16.180 -> 2.0.0
==> Upgrading awscli
...   

```

But after updating those tools i was able to deploy the demo resource toi AWS:

```bash
$> sam deploy --guided --template-file sam.jvm.yaml


Configuring SAM deploy
======================

	Looking for samconfig.toml :  Not found

	Setting default arguments for 'sam deploy'
	=========================================
	Stack Name [sam-app]:
	AWS Region [us-east-1]: eu-central-1
	#Shows you resources changes to be deployed and require a 'Y' to initiate deploy
	Confirm changes before deploy [y/N]: y
	#SAM needs permission to be able to create roles to connect to the resources in your template
	Allow SAM CLI IAM role creation [Y/n]:
	Save arguments to samconfig.toml [Y/n]:

	Looking for resources needed for deployment: Not found.
	Creating the required resources...
	Successfully created!

		Managed S3 bucket: aws-sam-cli-managed-default-samclisourcebucket-1s9zc500wkwxa
		A different default S3 bucket can be set in samconfig.toml

	Saved arguments to config file
	Running 'sam deploy' for future deployments will use the parameters saved above.
	The above parameters can be changed by modifying samconfig.toml
	Learn more about samconfig.toml syntax at
	https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-config.html

	Deploying with following values
	===============================
	Stack name                 : sam-app
	Region                     : eu-central-1
	Confirm changeset          : True
	Deployment s3 bucket       : aws-sam-cli-managed-default-samclisourcebucket-1s9zc500wkwxa
	Capabilities               : ["CAPABILITY_IAM"]
	Parameter overrides        : {}

Initiating deployment
=====================
Uploading to sam-app/e9223e9e453018324cc7a8be96b3108e  20989287 / 20989287.0  (100.00%)
Uploading to sam-app/5a47e03573d22bbf44b71a4bd6731cce.template  973 / 973.0  (100.00%)

Waiting for changeset to be created..

CloudFormation stack changeset
------------------------------------------------------------------------------------------------
Operation                        LogicalResourceId                ResourceType
------------------------------------------------------------------------------------------------
+ Add                            QuarkusLambdaFunctionGetResour   AWS::Lambda::Permission
                                 cePermissionProd
+ Add                            QuarkusLambdaFunctionRole        AWS::IAM::Role
+ Add                            QuarkusLambdaFunction            AWS::Lambda::Function
+ Add                            ServerlessRestApiDeployment275   AWS::ApiGateway::Deployment
                                 42d00aa
+ Add                            ServerlessRestApiProdStage       AWS::ApiGateway::Stage
+ Add                            ServerlessRestApi                AWS::ApiGateway::RestApi
------------------------------------------------------------------------------------------------

Changeset created successfully. arn:aws:cloudformation:eu-central-1:807403674325:changeSet/samcli-deploy1581631152/95caa8bb-a776-41eb-b18b-cbd871e8ebd2


Previewing CloudFormation changeset before deployment
======================================================
Deploy this changeset? [y/N]: y

2020-02-13 22:59:59 - Waiting for stack create/update to complete

CloudFormation events from changeset
-------------------------------------------------------------------------------------------------
ResourceStatus           ResourceType             LogicalResourceId        ResourceStatusReason
-------------------------------------------------------------------------------------------------
CREATE_IN_PROGRESS       AWS::IAM::Role           QuarkusLambdaFunctionR   -
                                                  ole
CREATE_IN_PROGRESS       AWS::IAM::Role           QuarkusLambdaFunctionR   Resource creation
                                                  ole                      Initiated
CREATE_COMPLETE          AWS::IAM::Role           QuarkusLambdaFunctionR   -
                                                  ole
CREATE_IN_PROGRESS       AWS::Lambda::Function    QuarkusLambdaFunction    -
CREATE_IN_PROGRESS       AWS::Lambda::Function    QuarkusLambdaFunction    Resource creation
                                                                           Initiated
CREATE_COMPLETE          AWS::Lambda::Function    QuarkusLambdaFunction    -
CREATE_IN_PROGRESS       AWS::ApiGateway::RestA   ServerlessRestApi        -
                         pi
CREATE_IN_PROGRESS       AWS::ApiGateway::RestA   ServerlessRestApi        Resource creation
                         pi                                                Initiated
CREATE_COMPLETE          AWS::ApiGateway::RestA   ServerlessRestApi        -
                         pi
CREATE_IN_PROGRESS       AWS::Lambda::Permissio   QuarkusLambdaFunctionG   -
                         n                        etResourcePermissionPr
                                                  od
CREATE_IN_PROGRESS       AWS::ApiGateway::Deplo   ServerlessRestApiDeplo   -
                         yment                    yment27542d00aa
CREATE_IN_PROGRESS       AWS::ApiGateway::Deplo   ServerlessRestApiDeplo   Resource creation
                         yment                    yment27542d00aa          Initiated
CREATE_IN_PROGRESS       AWS::Lambda::Permissio   QuarkusLambdaFunctionG   Resource creation
                         n                        etResourcePermissionPr   Initiated
                                                  od
CREATE_COMPLETE          AWS::ApiGateway::Deplo   ServerlessRestApiDeplo   -
                         yment                    yment27542d00aa
CREATE_IN_PROGRESS       AWS::ApiGateway::Stage   ServerlessRestApiProdS   -
                                                  tage
CREATE_IN_PROGRESS       AWS::ApiGateway::Stage   ServerlessRestApiProdS   Resource creation
                                                  tage                     Initiated
CREATE_COMPLETE          AWS::ApiGateway::Stage   ServerlessRestApiProdS   -
                                                  tage
CREATE_COMPLETE          AWS::Lambda::Permissio   QuarkusLambdaFunctionG   -
                         n                        etResourcePermissionPr
                                                  od
CREATE_COMPLETE          AWS::CloudFormation::S   sam-app                  -
                         tack
-------------------------------------------------------------------------------------------------

Stack sam-app outputs:
-------------------------------------------------------------------------------------------------
OutputKey-Description                            OutputValue
-------------------------------------------------------------------------------------------------
QuarkusLambdaApi - URL for application           https://qcj91rsgpi.execute-api.eu-
                                                 central-1.amazonaws.com/Prod/
-------------------------------------------------------------------------------------------------

Successfully created/updated stack - sam-app in eu-central-1
```

## Test HTTP-API

```bash
$> http https://qcj91rsgpi.execute-api.eu-central-1.amazonaws.com/Prod/hello
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 11
Content-Type: text/plain;charset=UTF-8
Date: Thu, 13 Feb 2020 22:02:55 GMT
X-Amzn-Trace-Id: Root=1-5e45c78b-fe7b746c09ea245838baa02c;Sampled=0
x-amz-apigw-id: H2wdzGvwliAFktw=
x-amzn-Remapped-Content-Length: 11
x-amzn-RequestId: 84405056-a778-4459-983c-03971d34f22a

hello jaxrs
```

So far - so good.

## Adding OpenAPI 3.0

APIs should be documented. The defacto standard way to do this is OpenAPI - aka Swagger.
OpenAPI is supported by Quarkus OOTB. You just have to add the appropriate module:
 
```bash
$> mvn quarkus:add-extension -Dextension="quarkus-smallrye-openapi"
```

This adds two new endpoints to your app:
* /openapi
* /swagger-ui

### OpenAPI

This produces a yaml file describing the deployed endpoints.

```yaml
http http://127.0.0.1:9090/openapi
HTTP/1.1 200 OK
Access-Control-Allow-Credentials: true
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Methods: GET, HEAD, OPTIONS
Access-Control-Allow-Origin: *
Access-Control-Max-Age: 86400
Content-Type: application/yaml;charset=UTF-8
content-length: 243

---
openapi: 3.0.1
info:
  title: Generated API
  version: "1.0"
paths:
  /hello:
    get:
      responses:
        "200":
          description: OK
          content:
            text/plain:
              schema:
                type: string
```

### Swagger-UI

This shows an interface to test and debug your endpoints.



