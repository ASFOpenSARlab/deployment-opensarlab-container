# https://github.com/dask/dask-docker/tree/main
# Make sure that the python lib verions here match the minor versions of dask-gateway found in the notebook image
FROM ghcr.io/dask/dask:2024.1.0 as release

# ignore DL3008 because pinning is not desired
# hadolint ignore=DL3008
RUN apt-get update &&\
    apt-get upgrade -y &&\
    apt-get install --no-install-recommends --fix-missing -y \
        curl &&\
    mamba install -c conda-forge -y \
        python=3.11.6 \
        pandas=2.2 \
        tornado=6.3 \
        boto3 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

##########################################

FROM release as testing
