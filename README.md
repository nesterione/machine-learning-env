# machine-learning-env

## Description 

This repository contains ready-to-use docker-compose files with different useful tools for machine learning teams.

* `minio` - self-hosted bucket service with s3-like API
* `mlflow` - preconfigured mlflow traking with minio as artefact storage [TODO]
* `airflow` - workflow management system

## Examples 

* https://github.com/nesterione/dvc-example - example of usage DVC (used minio as artefact storage)
* https://github.com/nesterione/mlflow-example - example of usage MLFlow

## How to use MLFlow container? 

Example of docker compose file, before usage put your values `MINIO_ACCESS_KEY` and `MINIO_SECRET_KEY`

This set up uses `minio` insted of s3 and `sqlite` to store metadata.

```
version: '3.0'

services:
  minio:
    image: minio/minio:RELEASE.2020-07-14T19-14-30Z
    volumes:
      - ./data/minio:/data
    ports:
      - "9998:9000"
    environment:
      MINIO_ACCESS_KEY: ${MINIO_ACCESS_KEY}
      MINIO_SECRET_KEY: ${MINIO_SECRET_KEY}
    command: server /data
  mlflow:
    build: mlflow-server
    container_name: mlflow
    environment:
      DEFAULT_ARTIFACT_ROOT: s3://mlflow/artefacts
      BACKEND_STORE_URI: sqlite:////data/mlflow.db
      MLFLOW_S3_ENDPOINT_URL: http://minio:9000
      AWS_ACCESS_KEY_ID: ${MINIO_ACCESS_KEY}
      AWS_SECRET_ACCESS_KEY: ${MINIO_SECRET_KEY}
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "1000m"
        max-file: "10"
    ports:
      - 5000:5000
    links: 
      - minio
    volumes:
      - "./data/mlflow:/data/"
```

How to run? 

```
docker-compose up -d
```

How to configure client? 

Make sure you have configured environment variables: 

* `MLFLOW_HOST` should point to MLFLOW ui 
* `MLFLOW_S3_ENDPOINT_URL` should point to S3 endpoint (if it is not s3)

Create experiment
```
import mlflow, os
import mlflow.tensorflow

EXPERIMENT_NAME = '/prjx/imdb'
mlflow.set_tracking_uri(os.environ["MLFLOW_HOST"])
experiment = mlflow.get_experiment_by_name(EXPERIMENT_NAME)
if experiment:
    experiment_id = experiment.experiment_id
else:
    # Possible to set up own s3 bucket artifact_location
    experiment_id = mlflow.create_experiment(name=EXPERIMENT_NAME)
print(f'Active experiment_id: {experiment_id}')

```

Use 

```
with mlflow.start_run(experiment_id=experiment_id):
	mlflow.log_param("any_param", 'your value')

```