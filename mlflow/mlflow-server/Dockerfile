FROM python:3.7.0-stretch

RUN apt-get update

ENV MLFLOW_SERVER_HOST 0.0.0.0
ENV BACKEND_STORE_URI /mlruns
ENV DEFAULT_ARTIFACT_ROOT /mlruns
ARG WORKSPACE /mlflow

RUN mkdir -p {WORKSPACE}
WORKDIR ${WORKSPACE}

COPY . ${WORKSPACE}
RUN pip install -r requirements.txt

RUN mkdir -p ${DEFAULT_ARTIFACT_ROOT}

EXPOSE 5000 

CMD mlflow server --backend-store-uri ${BACKEND_STORE_URI} \
                  --default-artifact-root ${DEFAULT_ARTIFACT_ROOT} \
                  --host ${MLFLOW_SERVER_HOST}