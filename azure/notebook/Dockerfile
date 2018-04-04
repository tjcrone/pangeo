FROM jupyter/base-notebook

USER root
RUN apt-get update \
  && apt-get install -yq --no-install-recommends libfuse-dev nano fuse vim git jq less pkgconf ffmpeg libavfilter-dev libavdevice-dev build-essential \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*
USER $NB_USER

RUN conda update -y conda
RUN pip install --upgrade pip

RUN conda install --yes -c ioam -c bokeh/channel/dev \
    bokeh=0.12.15dev1 \
    cython \
    cytoolz \
    datashader \
    dask=0.17.2 \
    distributed=1.21.4 \
    fastparquet \
    ipywidgets \
    jupyterlab \
    holoviews \
    lz4 \
    matplotlib \
    nb_conda_kernels \
    netcdf4 \
    nomkl \
    numba \
    numcodecs \
    numpy \
    pandas \
    python-blosc \
    scipy \
    scikit-image \
    tornado \
    xarray \
    zict \
    && conda clean -tipsy

RUN pip install fusepy click jedi kubernetes --upgrade --no-cache-dir

RUN pip install daskernetes==0.1.3 \
                git+https://github.com/dask/dask-kubernetes@5ba08f714ef38e585e9f2038b6be530c578b96dd \
                git+https://github.com/zarr-developers/zarr \
                git+https://github.com/pydata/xarray \
                git+https://github.com/ioam/holoviews@3f015c0a531f54518abbfecffaac72a7b3554ed3 \
                git+https://github.com/dask/gcsfs@2fbdc27e838a531ada080886ae778cb370ae48b8 \
                git+https://github.com/jupyterhub/nbserverproxy \
                --no-cache-dir \
                --upgrade

RUN pip install av==0.4.0 \
                numpngw \
                git+https://github.com/tjcrone/pycamhd

RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager \
                                 @jupyterlab/hub-extension \
                                 @pyviz/jupyterlab_holoviews

RUN jupyter serverextension enable --py nbserverproxy --sys-prefix

USER root
COPY prepare.sh /usr/bin/prepare.sh
RUN chmod +x /usr/bin/prepare.sh
RUN mkdir /home/$NB_USER/examples && chown -R $NB_USER /home/$NB_USER/examples
RUN mkdir /pre-home && mkdir /pre-home/examples && chown -R $NB_USER /pre-home
COPY worker-template.yaml /pre-home
COPY examples/ /pre-home/examples/

ENV DASK_TICK_MAXIMUM_DELAY=5s
ENV DASKERNETES_DIAGNOSTICS_LINK {JUPYTERHUB_SERVICE_PREFIX}proxy/{port}/status
ENV DASKERNETES_WORKER_TEMPLATE_PATH /home/$NB_USER/worker-template.yaml
ENV DASKERNETES_WORKER_NAME dask-{JUPYTERHUB_USER}-{uuid}
ENV DASK_DIAGNOSTICS_LINK {JUPYTERHUB_SERVICE_PREFIX}proxy/{port}/status
ENV DASK_KUBERNETES_WORKER_TEMPLATE_PATH /home/$NB_USER/worker-template.yaml
ENV DASK_KUBERNETES_WORKER_NAME dask-{JUPYTERHUB_USER}-{uuid}

RUN mkdir /gcs && chown -R $NB_USER /gcs
RUN mkdir /opt/app

# Add NB_USER to sudo
RUN echo "$NB_USER ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/notebook
RUN sed -ri "s#Defaults\s+secure_path=\"([^\"]+)\"#Defaults secure_path=\"\1:$CONDA_DIR/bin\"#" /etc/sudoers
USER $NB_USER

ENTRYPOINT ["tini", "--", "/usr/bin/prepare.sh"]
CMD ["start.sh jupyter lab"]
