ARG CUDA_VERSION=12.9.0
ARG TYPE=devel
# ARG TYPE=runtime
ARG BASE_IMAGE=nvidia/cuda:${CUDA_VERSION}-cudnn-${TYPE}-ubuntu24.04
FROM $BASE_IMAGE

ARG NB_USER="jovyan"
ARG NB_UID="1010"
ARG NB_GID="110"
# Automatically set by buildx
ARG TARGETPLATFORM

RUN rm -f /etc/apt/sources.list.d/*.list && \
        apt-get update --yes && apt-get upgrade --yes && \
        DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
                build-essential \
                apt-utils \
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
                libncurses5-dev \
                libncursesw5-dev \
                && \
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

COPY fix-permissions /usr/local/bin/
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
        conda install -y python=3.14 && \
        conda update --all --yes && \
        conda install -c conda-forge numpy ipython notebook jupyterhub jupyterlab jupyter_contrib_nbextensions seaborn ipywidgets opencv --yes && \
        jupyter notebook --generate-config && \
        jupyter lab clean && \
        conda clean --all -f -y && \
        rm -rf ${HOME}/.cache/yarn && \
        npm cache clean --force && \
        pip install torch==2.8.0 torchvision --index-url https://download.pytorch.org/whl/cu129 --no-cache-dir && pip install timm && pip cache purge && \
        conda install --yes tensorboard wandb einops transformers scipy scikit-image && \
        conda update --all --yes && \
        conda clean --all -f -y && \
        pip install nnunetv2 --no-cache-dir  && pip cache purge

EXPOSE 8888
ENTRYPOINT ["tini", "-g", "--", "start.sh"]
CMD ["start-notebook.sh"]
COPY jupyter_server_config.py /etc/jupyter/
COPY start-notebook.py start-notebook.sh start-singleuser.py start-singleuser.sh start.sh /usr/local/bin/
USER root
RUN chmod 777 /usr/local/bin/start-notebook.sh && \
        chmod 777 /usr/local/bin/start-notebook.py && \
        chmod 777 /usr/local/bin/start-singleuser.sh && \
        chmod 777 /usr/local/bin/start-singleuser.py && \
        chmod 777 /usr/local/bin/start.sh && \
        sed -re "s/c.ServerApp/c.NotebookApp/g" \
        /etc/jupyter/jupyter_server_config.py > /etc/jupyter/jupyter_notebook_config.py && \
        fix-permissions /etc/jupyter/ && \
        fix-permissions ${CONDA_DIR} && \
        fix-permissions ${HOME}

HEALTHCHECK  --interval=30s --timeout=3s --start-period=5s --retries=3 \
        CMD wget -O- --no-verbose --tries=1 --no-check-certificate \
        http${GEN_CERT:+s}://localhost:8888${JUPYTERHUB_SERVICE_PREFIX:-/}api || exit 1

USER ${NB_UID}

WORKDIR /tmp
RUN git clone https://github.com/PAIR-code/facets.git && \
        jupyter nbextension install facets/facets-dist/ --sys-prefix && \
        rm -rf /tmp/facets && \
        fix-permissions ${CONDA_DIR} && \
        MPLBACKEND=Agg python -c "import matplotlib.pyplot" && \
        fix-permissions ${HOME}

ENV nnUNet_raw="${HOME}/root/data/nnUNet_raw"
ENV nnUNet_results="${HOME}/root/data/nnUNet_results"
ENV nnUNet_preprocessed="${HOME}/root/data/nnUNet_preprocessed"

WORKDIR ${HOME}
COPY jupyter_notebook_config.json /etc/jupyter/

USER ${NB_UID}
