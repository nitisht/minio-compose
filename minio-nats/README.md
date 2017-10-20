# minio-nats

Setup Minio + Nats for testing bucket notifications.

## Prerequisites
- Docker installed (Docker version 17.05.0-ce on Linux)
- docker-compose installed (1.8.0)

## Setup Minio and Nats with docker-compose
We'll use docker-compose to set up Minio and Nats containers. Download the [docker-compose.yml](./docker-compose.yaml) file in your working directory. This docker-compose file declares two containers.

First one, `nats` runs official image. It exposes itself under `nats` hostname, with port `4222` exposed. Nats username and password is set to `user` and `password`.

Second container, `minio` runs latest Minio server. It needs to connect to the running Nats server, that's why there is the `depends_on` entry in docker-compose.

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
Below event will notify Nats of any object being created, accessed, or deleted from `test` bucket. 

```sh
mc events add myminio/test arn:minio:sqs:us-east-1:1:nats
```

#### Test notifications
Use the file [nats.go](./nats.go) to test events. Run the file using

```go
go run nats.go
```

You should see the notifications sent out to this terminal after files are updated.

#### Upload an object
Open another terminal and upload an object to `test` bucket and see if we get the notifications in Nats.

```sh
mc cp ./testfile myminio/test/testfile
```

### Stop the environment
Once you are done, turn off running containers, by

```
docker-compose down
```