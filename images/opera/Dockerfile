FROM jupyter/base-notebook:lab-4.0.7 as release

# Base Stage ****************************************************************
USER root
WORKDIR /

RUN set -ve

COPY pkgs/vnc_patches/vnc.patch /tmp/vnc.patch

# ignore DL3009 since deleting apt-get lists makes later installs fail
# hadolint ignore=DL3009
RUN apt-get update 

# ignore DL3008 because pinning is not desired
# ignore DL3009 since deleting apt-get lists makes later installs fail
# hadolint ignore=DL3008,DL3009
RUN apt-get install --no-install-recommends -y \
        software-properties-common \
        git && \
    apt-get install --no-install-recommends -y gpg-agent && \
    add-apt-repository -y ppa:ubuntugis/ubuntugis-unstable && \
    apt-get update && \
    apt-get upgrade -y 

# ignore DL3008 because pinning is not desired
# hadolint ignore=DL3008
RUN apt-get install --no-install-recommends --fix-missing -y \
    ### GENERAL
    zip \
    unzip \
    wget \
    vim \
    rsync \
    less \
    snaphu \
    curl \
    openssh-client \
    libgl1-mesa-glx \
    emacs \
    gnupg2 \
    jq \
    gfortran \
    make \
    proj-bin \
    geotiff-bin \
    libshp-dev \
    libshp2 \
    libhdf5-dev \
    libnetcdf-dev \
    libgdal-dev \
    libgsl-dev \
    gdal-bin \
    ### SNAP
    default-jdk-headless \
    ### Install texlive for PDF exporting of notebooks containing LaTex
    texlive-xetex \
    texlive-fonts-recommended \
    texlive-plain-generic \
    ### PyGMTSAR
    csh \
    autoconf \
    make \ 
    libtiff5-dev \ 
    liblapack-dev \
    libgmt-dev \
    gmt-dcw \
    gmt-gshhg \
    gmt \
    ### Jupyter Desktop
##    patch \
##    dbus-x11 \
##    xfce4 \
##    xfce4-panel \
##    xfce4-session \
##    xfce4-settings \
##    xorg \
##    xubuntu-icon-theme \
##    tigervnc-standalone-server \
##    tigervnc-xorg-extension \
    --

# Update conda and mamba: this breaks things in the hook, need to play with it more
# Updating just conda and mamba will auto-remove pip, so force pip to update to remain
#RUN mamba update -c conda-forge -y conda mamba pip

# ignore DL3059 to make the two codeblocks distinct
# hadolint ignore=DL3059
RUN mamba install -c conda-forge -y \
    ### Install plotting and general
    awscli \
    boto3 \
    pyyaml \
    bokeh \
    plotly \
    'pyopenssl>=23.0.0' \
    zstd==1.5.5 \
    zstandard==0.21.0 \
    ### Install jupyter libaries
    kernda \
    jupyter-resource-usage \
    nb_conda_kernels \
    jupyterlab-spellchecker \
    ipympl \
    jupyterlab_widgets \
    ipywidgets \
    #jupyter-ai \
    jupyterlab-git \
    panel \
    ### Dask
    dask-gateway \
    dask \
    distributed \
    ### Jupyter Desktop
##    websockify \
    -- 

# Install python packages
# ignore DL3059 to make the two codeblocks distinct
# ignore DL3013 since pinning packages is not desired
# hadolint ignore=DL3013,DL3059
RUN python3 -m pip install --no-cache-dir \
        ### For ASF
        url-widget \
        opensarlab-frontend==1.5.1 \
        jupyterlab-jupyterbook-navigation==0.1.4 \
        ### For pyGMTSAR
        pygmtsar
WORKDIR /
RUN mkdir -p /tmp/build/GMTSAR /usr/local/GMTSAR &&\
        git clone --branch master https://github.com/gmtsar/gmtsar /tmp/build/GMTSAR/
WORKDIR /tmp/build/GMTSAR
RUN autoconf &&\
        ./configure --with-orbits-dir=/tmp CFLAGS='-z muldefs' LDFLAGS='-z muldefs' &&\
        make &&\
        make install &&\
        mv -v /tmp/build/GMTSAR/bin /usr/local/GMTSAR/bin &&\
        rm -rf /tmp/build
WORKDIR /
RUN \
    ### Jupyter Desktop
    # When running a local version of this dockerfile, consult https://github.com/jupyterhub/jupyter-remote-desktop-proxy/issues/47#issuecomment-1687484891
##    cd / &&\
##        git clone https://github.com/jupyterhub/jupyter-remote-desktop-proxy.git /opt/install &&\
##        mv /tmp/vnc.patch /opt/install/vnc.patch &&\
##        cd /opt/install &&\
##        # Due to breaking updates, for now try to find an older working commit
##        git checkout 8d1342d9c93803c5f9c3b820dbbcaa55d3b91e01 &&\
##        mamba env update -n base --file environment.yml &&\
##        patch -p1 < vnc.patch &&\
##    cd / &&\
    ### Extra stuff
    # Make sure that any files in the home directory are jovyan permission
    chown -R jovyan:users "$HOME/" &&\
    # Make sure mamba (within conda) has write access
    chmod -R 777 /opt/conda/pkgs/ &&\
    # Make sure JupyterLab settings is writable
    mkdir -p /opt/conda/share/jupyter/lab/settings/ &&\
    chown jovyan:users /opt/conda/share/jupyter/lab/settings/ &&\
    chmod -R 775 /opt/conda/share/jupyter/lab/settings/ &&\
    # Add sudo group user 599 elevation
    addgroup -gid 599 elevation &&\
    echo '%elevation ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers &&\
    # Use the kernel display name `base` for the base conda environment
    mamba run -n base kernda --display-name base -o /opt/conda/share/jupyter/kernels/python3/kernel.json &&\
    mamba clean -y --all &&\
    mamba init &&\
    rm -rf /home/jovyan/..?* /home/jovyan/.[!.]* /home/jovyan/*

# Create opera environment and register kernel
COPY environment/environment.yaml /tmp/environment.yaml

RUN mamba env create -f /tmp/environment.yaml &&\
    /opt/conda/envs/opera/bin/python -m ipykernel install --prefix /opt/conda/envs/opera --name opera --display-name "opera" &&\
    rm -rf /opt/conda/pkgs/cache &&\
    conda env config vars set -n opera GDAL_HTTP_COOKIEFILE=/tmp/cookies.txt &&\
    conda env config vars set -n opera GDAL_HTTP_COOKIEJAR=/tmp/cookies.txt &&\
    conda env config vars set -n opera VSI_CACHE=YES

### GMTSAR
ENV PATH=/usr/local/GMTSAR/bin:$PATH

WORKDIR /home/jovyan
USER jovyan

##########################################

FROM release as testing 

COPY tests/* /tests/

RUN bash /tests/*.sh
