# Quarkus - Amazon Lambda with RESTEasy, Undertow, or Vert.x Web
===
As described [Quarkus - Amazon Lambda with RESTEasy, Undertow, or Vert.x Web](https://quarkus.io/guides/amazon-lambda-http).   

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

