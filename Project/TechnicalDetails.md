# Technical Details

## Server

1. Each team has a dedicated server on which to run their project, and each of the teammembers have been granted access to it. To access the appropriate machine: `ssh {CS_USERNAME}@fall2021-comp598-{TEAM_NUMBER}.cs.mcgill.ca`. Feel free to copy over your ssh keys so you don't have to type in your password every time you log in.
2. Check that you can access the Movie API service: `curl http://fall2020-comp598.cs.mcgill.ca:8080/user/{USER_ID}`
3. Check that docker is running: `ps aux | grep docker`
4. Check that your service is running by visiting [`http://fall2021-comp598-{TEAM_NUMBER}.cs.mcgill.ca:8082/recommend/{USER_ID}`](http://fall2021-comp598-{TEAM_NUMBER}.cs.mcgill.ca:8082/recommend/{USER_ID}). This should return a comma-separated list of 20 movie recommendations, from most to least highly recommended for `USER_ID`.

## Docker Container

1. Create a Dockerfile to set up the environment that you need: `https://docs.docker.com/get-started/`
2. Build your docker image, and run it in a container:

```bash
docker build -t {IMAGE_NAME}:{VERSION} .
docker run -it -p 8082:8082/tcp {IMAGE_NAME}:{VERSION}
```
3. Check the Kafka log to see that the replies are being received:

```bash
docker run -it bitnami/kafka kafka-console-consumer.sh --bootstrap-server fall2021-comp598.cs.mcgill.ca:9092 --topic movielog{TEAM_NUMBER}
```

## Kafka

There are many different [Kafka clients](https://docs.confluent.io/current/clients/index.html) you can use to read and write to Kafka. You are free to use any libraries you wish. We have included two in this template:

* [Apache Kafka Streams](https://kafka.apache.org/documentation/streams/) (the official client)
* [Kotka](https://github.com/blueanvil/kotka/) (a lightweight Kotlin client)

To read from Kafka, you will need to connect to the Kafka server at `fall2021-comp598.cs.mcgill.ca:9092` and stream from the topic `movielog[TEAM_NUMBER]`.

We recommend dumping a subset of the logs to disk for prototyping, then dealing with the live stream once the model is stable.

## REST API

You will need to access the user and movie services to access the corresponding features. Below are a few possible options for doing so:

* [OkHttp](https://github.com/square/okhttp) - The dependency is included, but you will need to read the documentation.
* [Moshi](https://github.com/square/moshi) - Lightweight JSON binding to Java/Kotlin classes.
* [`URL.readText()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.io/java.net.-u-r-l/read-text.html) - You will need to implement the JSON parsing and data bindings yourself.
