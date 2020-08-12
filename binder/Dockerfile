FROM jupyter/scipy-notebook:1386e2046833

ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"
ENV HOME /home/${NB_USER}

USER $NB_UID

ENV JUPYTER_ENABLE_LAB yes

# Install the packages we need
RUN conda install --yes -c conda-forge "nodejs=10.8" ipywidgets ipython-sql pymssql pyodbc \
  python-dotenv nbstripout cx_Oracle papermill pywinauto r-irkernel chardet watchdog Faker \
  pyppeteer jupyterlab=1.2.3 voila jupyterlab-git openssh \
  && conda clean -a


RUN conda init

RUN pip install nbinteract

# Lab Extensions
ENV NODE_OPTIONS="--max-old-space-size=4096"
RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager@1.1 --no-build && \
  jupyter labextension install @jupyterlab/celltags --no-build && \ 
  jupyter labextension install @jupyterlab/toc --no-build && \
  jupyter labextension install @jupyter-voila/jupyterlab-preview  --no-build
  # this is not working..? I can't seem to get it to fail when I run it manually inside the above layer.
  # build and clean assets, from: https://github.com/jupyterlab/jupyterlab/issues/4930#issuecomment-446597498
RUN jupyter lab build && jupyter lab clean && \
  jlpm cache clean && \
  npm cache clean --force && \
  rm -rf $HOME/.node-gyp && \
  rm -rf $HOME/.local 
  
# Set up git extension to work
RUN jupyter serverextension enable --py jupyterlab_git

# Set notebook widgets to work
RUN jupyter nbextension enable --py widgetsnbextension

# Generate base jupyter config
RUN jupyter notebook --generate-config -y

# set up nbstripout
RUN mkdir -p /home/$NB_USER/.config/git 
RUN git init && nbstripout --global --attributes=/home/$NB_USER/.config/git/attributes --install && rm .git -rf

# set up for binder, aka copy in repo
COPY . ${HOME}/binder

# fix permissions on folders so end user can do as they wish
RUN fix-permissions $CONDA_DIR && \
  fix-permissions /home/$NB_USER
