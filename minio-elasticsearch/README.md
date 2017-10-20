# minio-elasticsearch

Setup Minio + Elasticsearch for testing bucket notifications.

## Prerequisites
- Docker installed (Docker version 17.05.0-ce on Linux)
- docker-compose installed (1.8.0)

## Setup Minio and Elasticsearch with docker-compose
We'll use docker-compose to set up Minio and Elasticsearch containers. Download the [docker-compose.yml](./docker-compose.yaml) file in your working directory. This docker-compose file declares two containers.

First one, `elasticsearch` runs official image. It exposes itself under `elasticsearch` hostname, with two environment variables set:

```
discovery.type=single-node
xpack.security.enabled=false
```

`discovery.type` describe that this is a single node Elasticsearch setup. `xpack.security.enabled` set to `false` indicates `xpack` is disabled in this instance. Read more [here](https://www.elastic.co/guide/en/x-pack/current/xpack-settings.html).

Second container, minio runs latest Minio server. It needs to connect to the running Elasticsearch server, that's why there is the `depends_on` entry in docker-compose.

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
Below event will notify Elasticsearch of any object being created, accessed, or deleted from `test` bucket. 

```sh
mc events add myminio/test arn:minio:sqs:us-east-1:1:elasticsearch
```
#### Upload an object
Let's upload an object to `test` bucket and see if we get the notifications in Elasticsearch.

```sh
mc cp ./testfile myminio/test/testfile
```
#### Check notification with curl
You should now see the notification available in Elasticsearch index topic `minio_index`.

```sh
curl "http://localhost:9200/minio_index/_search?pretty=true"
```

### Stop the environment
Once you are done, turn off running containers, by

```
docker-compose down
```