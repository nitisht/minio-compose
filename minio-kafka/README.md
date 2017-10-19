# minio-kafka

Setup Minio + Apache Kafka for testing bucket notifications.

## Prerequisites
- Docker installed (Docker version 17.05.0-ce on Linux)
- docker-compose installed (1.8.0)

## Setup Minio and Apache Kafka with docker-compose
We'll use docker-compose to set up Minio and Kafka containers. Download the [docker-compose.yml](./docker-compose.yaml) file in your working directory. This docker-compose file declares two containers. 

First one, `kafka` runs Kafka image developed by Spotify team (it includes Zookeeper). It exposes itself under `kafka` hostname, with two environment variables set:

```
ADVERTISED_HOST: kafka
ADVERTISED_PORT: 9092
```

Those two describe to which hostname the Kafka server should respond from within the container.

Second container, minio runs latest Minio server. It needs to connect to the running Kafka server, that's why there is the `depends_on` entry in docker-compose.

### Start the environment
Once the `docker-compose.yml` file is ready, navigate to working directory via terminal and run:

```
docker-compose up
```

After running this command, you can check status of the containers by

```
docker-compose ps
```

### Test setup
After both the containers are running successfully, use [mc](https://docs.minio.io/docs/minio-client-quickstart-guide) to setup event notifications and see it notifications in action.

#### Add mc alias
Add current Minio server as host by

```sh
mc config host add myminio http://172.24.0.3:9000 minio minio123
```

#### Create a bucket

```sh
mc mb myminio/test
```

#### Set up notifications
Below event will notify Kafka of any object being created, accessed, or deleted from `test` bucket. 

```sh
mc events add myminio/test arn:minio:sqs:us-east-1:1:kafka
```
#### Upload an object
Let's upload an object to `test` bucket and see if we get the notifications in Kafka.

```sh
mc cp ./testfile myminio/test/testfile
```
#### Check notification with Kafka
You should now see the notification available in Kafka topic `minio-topic`. Check using the [Kafkacat](https://github.com/edenhill/kafkacat) tool.

```sh
kafkacat -C -b localhost:9092 -t minio-topic
```

### Stop the environment
Once you are done, turn off running containers, by

```
docker-compose down
```