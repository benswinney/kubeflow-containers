# https://github.com/jupyter/docker-stacks/blob/master/datascience-notebook/Dockerfile
FROM jupyter/datascience-notebook:04f7f60d34a6
USER root
ENV PATH="/home/jovyan/.local/bin:${PATH}"

RUN pip --no-cache-dir install --quiet \
      'kfp==1.0.0' \
      'kfp-server-api==1.0.0' \
      'kfp-tekton==0.2.0' \
      'kubeflow-fairing==1.0.1' \
      'kubeflow-metadata==0.3.1' \
      'kubeflow-pytorchjob==0.1.3' \
      'kubeflow-tfjob==0.1.3' \
      'minio==5.0.10' \
      'git+https://github.com/zachomedia/s3fs@8aa929f78666ff9e323cde7d9be9262db5a17985'

# kfp-azure-databricks needs to be run after kfp
RUN pip --no-cache-dir install --quiet \
      'fire==0.3.1' \
      'git+https://github.com/kubeflow/pipelines@1d86111d8f152d3ed7506ea59cee1bfbc28abbf9#egg=kfp-azure-databricks&subdirectory=samples/contrib/azure-samples/kfp-azure-databricks'

ARG DREMIO_VERSION=1.4.2.1003
ARG DREMIO_FULL_VERSION=1.4.2.1003-1
ARG DREMIO_RPM=https://download.dremio.com/odbc-driver/${DREMIO_VERSION}/dremio-odbc-${DREMIO_FULL_VERSION}.x86_64.rpm
ARG DREMIO_SHA=d2f5ff5d498d97f29a0eedc485dbcc85faf0e82eed2c1f7eb0ef3fee393ad450

# Added ODBC Support for Dremio
RUN conda install --quiet --yes pyodbc && \
    apt-get update && \
    apt-get install -y alien unixodbc unixodbc-dev && \
    wget --quiet "$DREMIO_RPM" && \
    echo "${DREMIO_SHA} dremio-odbc-${DREMIO_FULL_VERSION}.x86_64.rpm" | sha256sum -c - && \
    alien -i --scripts dremio-odbc-*x86_64.rpm && \
    rm -f dremio-odbc-*x86_64.rpm && \
    rm -rf /var/lib/apt/lists/* && \
    npm cache clean --force && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    rm -rf /home/$NB_USER/.node-gyp && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Need this to connect
ENV DREMIO_DRIVER=/opt/dremio-odbc/lib64/libdrillodbc_sb64.so

# Configure container startup
EXPOSE 8888
USER jovyan
COPY start-custom.sh /usr/local/bin/
ENTRYPOINT ["tini", "--"]
CMD ["start-custom.sh"]
