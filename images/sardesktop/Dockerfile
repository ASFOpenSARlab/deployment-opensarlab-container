FROM quay.io/jupyter/base-notebook:lab-4.3.4 as release

# Base Stage ****************************************************************
USER root
WORKDIR /

RUN set -ve

# ignore DL3008 because pinning is not desired
# ignore DL3059 because combining the two RUN statements fail
# ignore DL3009 since deleting apt-get lists makes later installs fail
# hadolint ignore=DL3008,DL3059,DL3009
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
    --

# ignore DL3059 to make the two codeblocks distinct
# ignore DL3013 since pinning packages is not desired
# hadolint ignore=DL3059,DL3013
RUN python3 -m pip install --no-cache-dir \
        ### For ASF
        url-widget \
        opensarlab-frontend==1.5.1 \
        jupyterlab-jupyterbook-navigation==0.2.0 \
        jupyterlab_myst \
        ### For pyGMTSAR
        pygmtsar
WORKDIR /
RUN     mkdir -p /tmp/build/GMTSAR /usr/local/GMTSAR &&\
        git clone --branch master https://github.com/gmtsar/gmtsar /tmp/build/GMTSAR/
WORKDIR /tmp/build/GMTSAR &&\
        autoconf &&\
        ./configure --with-orbits-dir=/tmp CFLAGS='-z muldefs' LDFLAGS='-z muldefs' &&\
        make &&\
        make install &&\
        mv -v /tmp/build/GMTSAR/bin /usr/local/GMTSAR/bin &&\
        rm -rf /tmp/build
WORKDIR /
    ### Extra stuff
    # Make sure that any files in the home directory are jovyan permission
RUN chown -R jovyan:users "$HOME/" &&\
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

### GMTSAR
ENV PATH=/usr/local/GMTSAR/bin:$PATH

######
# Virtual Desktop
# ignore DL3013 since pinning packages is not desired
# hadolint ignore=DL3013
RUN python3 -m pip install --no-cache-dir jupyter-remote-desktop-proxy &&\
    mamba install -c conda-forge -y websockify

# ignore DL3008 because pinning is not desired
# hadolint ignore=DL3008
RUN apt-get install --no-install-recommends --fix-missing -y \
    dbus-x11 \
    xfce4 \
    xfce4-panel \
    xfce4-session \
    xfce4-settings \
    xorg \
    xubuntu-icon-theme \
    tigervnc-standalone-server \
    tigervnc-xorg-extension \
    --

# Replace Hub Control button with something more suited for an OSL lab
# ignore DL3059 to make the two codeblocks distinct
# hadolint ignore=DL3059
RUN sed -i \
    -e 's|if hub_control_panel_url|if LAB_SHORT_NAME|' \
    -e 's|{{ hub_control_panel_url }}|/lab/{{ LAB_SHORT_NAME }}/hub/home|' \
    -e 's|Hub Control Panel|Shutdown and Logout Page|' \
    /opt/conda/lib/python3.11/site-packages/jupyter_remote_desktop_proxy/templates/index.html

# ignore DL3059 to make the two codeblocks distinct
# hadolint ignore=DL3059
RUN sed -i \
    -e 's|cd |sleep 1 \&\& cd |' \
    /opt/conda/lib/python3.11/site-packages/jupyter_remote_desktop_proxy/share/xstartup

# Install Browser
COPY configs/mozilla-firefox.pref /etc/apt/preferences.d/mozilla-firefox
# ignore DL3008 because pinning is not desired
# hadolint ignore=DL3008
RUN add-apt-repository ppa:mozillateam/ppa &&\
    apt-get install -y --no-install-recommends firefox
#
# END Virtual Desktop
####

WORKDIR /home/jovyan
USER jovyan

##########################################

FROM release as testing 

COPY tests/* /tests/

RUN bash /tests/*.sh
