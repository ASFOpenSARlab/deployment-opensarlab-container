# deployment-opensarlab-container

This repository hosts the container image used by OpenSARLab,
(as well as several others) an
[OpenScienceLab](https://asf.alaska.edu/asf-services-open-science-lab/)
JupyterHub deployment.

This image is also runnable locally: all packages are publicly available
[here](https://github.com/orgs/ASFOpenSARlab/packages/container/package/deployment-opensarlab-container_sar).
To run the image used in the production OpenSARLab environment, run
the following from the command line:

```bash
docker run -p 8888:8888 ghcr.io/asfopensarlab/deployment-opensarlab-container_sar:main
```

Click the `127.0.0.1:8888` link that appears, and you will be able to access the image.
