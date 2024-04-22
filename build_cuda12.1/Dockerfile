ARG CUDA_VERSION=12.1.0
ARG BASE_IMAGE=nvidia/cuda:${CUDA_VERSION}-cudnn8-runtime-ubuntu22.04
FROM $BASE_IMAGE

ARG CUDA_VERSION_2=12.1
ARG PYTHON_VERSION=3.11
ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"
# Automatically set by buildx
ARG TARGETPLATFORM

RUN rm -f /etc/apt/sources.list.d/*.list && \
        apt-get update --yes && apt-get upgrade --yes && \
        DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
                build-essential \
                apt-utils \
                htop \
                libgraphviz-dev \
                fonts-liberation \
                locales \
                sudo \
                ca-certificates \
                ccache \
                cmake \
                curl \
                git \
                bzip2 \
                wget \
                vim \
                less \
                nano \
                tini \
                cm-super \
                ffmpeg \
                libncurses5-dev \
                libncursesw5-dev \
                libjpeg-dev \
                libpng-dev && \
        apt-get update --yes && apt-get upgrade --yes && \
        apt-get clean && rm -rf /var/lib/apt/lists/* && \
        echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
        locale-gen

ENV CONDA_DIR=/opt/conda \
        NB_USER=${NB_USER} \
        NB_UID=${NB_UID} \
        NB_GID=${NB_GID} \
        LC_ALL=en_US.UTF-8 \
        LANG=en_US.UTF-8 \
        LANGUAGE=en_US.UTF-8 \
        HOME="/home/${NB_USER}" \
        SHELL=/bin/bash

ENV PATH=${CONDA_DIR}/bin:${PATH} \
        LD_LIBRARY_PATH=${CONDA_DIR}/lib:$LD_LIBRARY_PATH \
        XDG_CACHE_HOME=${HOME}/.cache/

COPY fix-permissions start-notebook.sh start-notebook.py /usr/local/bin/
RUN chmod a+rx /usr/local/bin/fix-permissions && \
        sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc && \
        echo 'eval "$(command conda shell.bash hook 2> /dev/null)"' >> /etc/skel/.bashrc && \
        echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
        sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers && \
        sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers && \
        useradd -l -m -s /bin/bash -N -u ${NB_UID} ${NB_USER} && \
        mkdir -p ${CONDA_DIR} && \
        chown ${NB_USER}:${NB_GID} ${CONDA_DIR} && \
        chmod g+w /etc/passwd && \
        fix-permissions ${HOME} && \
        fix-permissions ${CONDA_DIR}

USER ${NB_UID}

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

WORKDIR /tmp
RUN curl -fsSL -v -o ~/miniconda.sh -O  "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh" && \
        chmod +x ~/miniconda.sh && \
        bash ~/miniconda.sh -b -f -p ${CONDA_DIR} && \
        rm ~/miniconda.sh && \
        source ${CONDA_DIR}/etc/profile.d/conda.sh && \
        conda activate && \
        conda install -c conda-forge python=3.11 numpy ipython pyyaml cmake conda-build gcc gxx ninja --yes && \
        conda update --all --yes && \
        conda clean --all -f -y && \
        fix-permissions ${CONDA_DIR} && \
        fix-permissions ${HOME} && \
        conda install -c conda-forge notebook jupyterhub jupyterlab jupyter_contrib_nbextensions jupyterlab-drawio --yes && \
        jupyter notebook --generate-config && \
        jupyter lab clean && \
        conda clean --all -f -y && \
        rm -rf ${HOME}/.cache/yarn && \
        npm cache clean --force && \
        fix-permissions ${CONDA_DIR} && \
        fix-permissions ${HOME}

EXPOSE 8888
ENTRYPOINT ["tini", "-g", "--"]
CMD ["start-notebook.sh"]
COPY jupyter_server_config.py /etc/jupyter/
USER root
RUN chmod 777 /usr/local/bin/start-notebook.sh && \
        chmod 777 /usr/local/bin/start-notebook.py && \
        sed -re "s/c.ServerApp/c.NotebookApp/g" \
        /etc/jupyter/jupyter_server_config.py > /etc/jupyter/jupyter_notebook_config.py && \
        fix-permissions /etc/jupyter/
HEALTHCHECK  --interval=30s --timeout=3s --start-period=5s --retries=3 \
        CMD wget -O- --no-verbose --tries=1 --no-check-certificate \
        http${GEN_CERT:+s}://localhost:8888${JUPYTERHUB_SERVICE_PREFIX:-/}api || exit 1

USER ${NB_UID}
WORKDIR ${HOME}
Run conda update -n base -c conda-forge conda --yes && \
        conda update --all --yes && \
        conda update --all --yes && \
        conda install pytorch torchvision torchaudio pytorch-cuda=${CUDA_VERSION_2} -c pytorch -c nvidia --yes && \
        conda list "^(.*torch.*)$" | tail -n+4 | awk '{ print $1 " ==" $2 }' > /opt/conda/conda-meta/pinned && \
        conda config --set channel_priority strict && \
        conda update --all --yes && \
        conda update --all --yes && \
        conda install --yes -c conda-forge cython tqdm scipy sympy pandas scikit-learn scikit-image && \
        conda install --yes -c conda-forge matplotlib-base && \
        # conda install --yes -c conda-forge matplotlib && \
        pip install matplotlib --no-cache-dir && \
        conda install --yes -c conda-forge timm wandb && \
        # conda install --yes -c conda-forge opencv && \
        pip install opencv-python --no-cache-dir && \
        conda install --yes -c conda-forge seaborn && \
        conda install --yes -c conda-forge tensorboard einops transformers numba && \
        conda install --yes -c conda-forge ipywidgets ipympl ipyleaflet && \
        pip install graphviz --no-cache-dir && \
        conda update --all --yes && \
        conda clean --all -f -y && \
        fix-permissions ${CONDA_DIR} && \
        fix-permissions ${HOME}

WORKDIR /tmp
RUN git clone https://github.com/PAIR-code/facets.git && \
        jupyter nbextension install facets/facets-dist/ --sys-prefix && \
        rm -rf /tmp/facets && \
        fix-permissions ${CONDA_DIR} && \
        MPLBACKEND=Agg python -c "import matplotlib.pyplot" && \
        fix-permissions ${HOME}

WORKDIR ${HOME}
RUN jupyter nbextension enable --py --sys-prefix ipyleaflet && \
        jupyter labextension install jupyterlab-plotly && \
        jupyter labextension install @jupyter-widgets/jupyterlab-manager plotlywidget && \
        jupyter labextension install @ijmbarr/jupyterlab_spellchecker && \
        fix-permissions $CONDA_DIR && \
        fix-permissions ${HOME}
COPY jupyter_notebook_config.json /etc/jupyter/


USER ${NB_UID}

# END
RUN conda update --all --yes && \
        conda clean --all -f -y && \
        fix-permissions ${CONDA_DIR} && \
        fix-permissions ${HOME}
USER ${NB_UID}
