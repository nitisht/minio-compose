# minio-redis

Setup Minio + Redis for testing bucket notifications.

## Prerequisites
- Docker installed (Docker version 17.05.0-ce on Linux)
- docker-compose installed (1.8.0)

## Setup Minio and Redis with docker-compose
We'll use docker-compose to set up Minio and Redis containers. Download the [docker-compose.yml](./docker-compose.yaml) file in your working directory. This docker-compose file declares two containers.

First one, `redis_db` runs official image. It exposes itself under `redis_db` hostname, with environment variable `REDIS_PASS_FILE` set to a file containing redis password.

Second container, `minio` runs latest Minio server. It needs to connect to the running Redis server, that's why there is the `depends_on` entry in docker-compose.

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
Below event will notify Redis of any object being created, accessed, or deleted from `test` bucket. 

```sh
mc events add myminio/test arn:minio:sqs:us-east-1:1:redis
```

#### Set up redis-cli
Install `redis-cli` using

```sh
sudo apt-get install redis-tools
```

Then start it to monitor the `redis` instance we launched earlier

```sh
redis-cli -a yoursecret
127.0.0.1:6379> monitor
OK
```

#### Upload an object
Open another terminal and upload an object to `test` bucket and see if we get the notifications in Redis.

```sh
mc cp ./testfile myminio/test/testfile
```

You should see the notifications sent out to `redis` in the `redis-cli` monitoring terminal.


### Stop the environment
Once you are done, turn off running containers, by

```
docker-compose down
```