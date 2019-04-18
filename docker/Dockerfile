FROM continuumio/anaconda3:5.2.0

RUN apt-get update && apt-get upgrade -y \
    && apt-get install -y \
        libpq-dev \
        build-essential \
        git \
        sudo \
        cmake zlib1g-dev libjpeg-dev xvfb libav-tools xorg-dev libboost-all-dev libsdl2-dev swig \
    && rm -rf /var/lib/apt/lists/*


RUN conda config --append channels conda-forge
RUN conda update -n base conda
#RUN conda install -y  conda-forge 
RUN conda install -y         tensorflow 
RUN conda install -y         jupyter_contrib_nbextensions 
RUN conda install -y         pyopengl
#RUN conda install -y -c conda-forge \
#        tensorflow \
#        jupyter_contrib_nbextensions \
#        pyopengl

RUN pip install "gym[atari,box2d,classic_control]"
RUN pip install bottle

ENV username devel
ENV userid 1000

ARG home=/home/${username}
ARG workdir=${home}/handson-ml

RUN adduser devel --uid 1000 --gecos '' --disabled-password \
    && echo "${username} ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/${username} \
    && chmod 0440 /etc/sudoers.d/${username}

WORKDIR ${workdir}
RUN chown ${username}:${username} ${workdir}

USER ${username}

RUN mkdir -p  ~/.vim/pack/foo/start && cd  ~/.vim/pack/foo/start \
	&& git clone https://github.com/python-mode/python-mode.git \
	&& cd python-mode \
	&& git submodule update --init --recursive
 
RUN jupyter contrib nbextension install --user
RUN jupyter nbextension enable toc2/main


# INFO: Jupyter and nbdime extension are not totally integrated (anaconda image is py36,
#       nbdime checks for py35 at the moment, still the config below enables diffing
#       notebooks with nbdiff (and nbdiff support in git diff command) after connecting
#       to the container by "make exec" (or "docker-compose exec handson-ml bash")
#       You may also try running:
#         nbd NOTEBOOK_NAME.ipynb
#       to get nbdiff between checkpointed version and current version of the given notebook
USER root
WORKDIR /
#RUN conda install -y -c conda-forge 
RUN conda install -y  nbdime
USER ${username}
WORKDIR ${workdir}

RUN git-nbdiffdriver config --enable --global

# INFO: Optionally uncomment any (one) of the following RUN commands below to ignore either
#       metadata or details in nbdiff within git diff
#RUN git config --global diff.jupyternotebook.command 'git-nbdiffdriver diff --ignore-metadata'
RUN git config --global diff.jupyternotebook.command 'git-nbdiffdriver diff --ignore-details'


# INFO: Dirty nbdime patching (ignored if not matching)
#COPY docker/nbdime-*.patch /tmp/
USER root
WORKDIR /
RUN patch -d /opt/conda/lib/python3.6/site-packages -p1 --forward --reject-file=- < \
        /tmp/nbdime-1-details.patch || true \
    && patch -d /opt/conda/lib/python3.6/site-packages -p1 --forward --reject-file=- < \
        /tmp/nbdime-2-toc.patch || true
#RUN rm /tmp/nbdime-*.patch
RUN apt-get update && apt-get install -y vim exuberant-ctags  ack-grep vim-nox
USER ${username}
WORKDIR ${workdir}


#COPY docker/bashrc.bash /tmp/
#RUN cat /tmp/bashrc.bash >> ${home}/.bashrc
RUN echo "export PATH=\"${workdir}/docker/bin:$PATH\"" >> ${home}/.bashrc
#RUN sudo rm /tmp/bashrc.bash

RUN export DISPLAY="docker.for.mac.localhost:0"

# INFO: Uncomment lines below to enable automatic save of python-only and html-only
#       exports alongside the notebook
#COPY docker/jupyter_notebook_config.py /tmp/
#RUN cat /tmp/jupyter_notebook_config.py >> ${home}/.jupyter/jupyter_notebook_config.py
#RUN sudo rm /tmp/jupyter_notebook_config.py

# INFO: Uncomment the RUN command below to disable git diff paging
#RUN git config --global core.pager ''

# INFO: Uncomment the RUN command below for easy and constant notebook URL (just localhost:8888)
#       That will switch jupyter to using empty password instead of a token.
#       To avoid making a security hole you SHOULD in fact not only uncomment but
#       regenerate the hash for your own non-empty password and replace the hash below.
#       You can compute a password hash in any notebook, just run the code:
#          from notebook.auth import passwd
#          passwd()
#       and take the hash from the output
RUN mkdir -p ${home}/.jupyter && \
    echo 'c.NotebookApp.password = u"sha1:c6bbcba2d04b:f969e403db876dcfbe26f47affe41909bd53392e"' \
    >> ${home}/.jupyter/jupyter_notebook_config.py
