FROM quay.io/jupyter/base-notebook:lab-4.3.4 AS release

USER root
WORKDIR /

RUN set -ve

# ignore DL3059 to make the two codeblocks distinct
# hadolint ignore=DL3059
RUN mamba install -c conda-forge -y \
    dask-gateway \
    dask=='2024.1.0' \
    distributed=='2024.1.0' \
    ipywidgets \
    --

COPY ./etc/gateway_access_example.ipynb /etc/singleuser/notebooks/gateway_access_example.ipynb

WORKDIR /home/jovyan
USER jovyan


##########################################

FROM release AS testing
