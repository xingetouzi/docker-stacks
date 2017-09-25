# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.

FROM jupyter/scipy-notebook

USER root

RUN userdel $NB_USER
# && rm -rf /home/$NB_USER

WORKDIR /opt

ENV CONDA_DIR /opt/conda

# TA-lib dependencies
RUN wget -O ta-lib.tar.gz http://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-src.tar.gz && \
tar -xzf ta-lib.tar.gz &&\
cd ta-lib && ./configure --prefix=/usr && make && make install && cd ..

# update pip source
RUN mkdir -p ~/.pip && \
echo "[global]" > ~/.pip/pip.conf && \
echo "index-url = http://mirrors.aliyun.com/pypi/simple/" >> ~/.pip/pip.conf && \
echo "" >> ~/.pip/pip.conf && \
echo "[install]" >> ~/.pip/pip.conf && \
echo "trusted-host=mirrors.aliyun.com" >> ~/.pip/pip.conf

# activate jupyter_contrib
 RUN pip install jupyter_contrib_nbextensions && \
 pip install jupyter_nbextensions_configurator

# update conda settings
RUN rm /opt/conda/.condarc && \
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/ --system && \
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/ --system && \
conda config --set show_channel_urls yes --system && \
conda config --set allow_softlinks False --system

# Install Python 3 packages
RUN conda install --quiet --yes \
    'ipyparallel=6.0.*' \
    'nb_conda'

RUN jupyter nbextension install --sys-prefix --py ipyparallel  &&\
    jupyter nbextension enable --sys-prefix --py ipyparallel  &&\
    jupyter serverextension enable --sys-prefix --py ipyparallel

RUN jupyter contrib nbextension install --sys-prefix && \
    jupyter nbextensions_configurator enable --sys-prefix &&\
    jupyter nbextensions_configurator enable --system

ENTRYPOINT ["tini", "--"]

CMD ["sh", "/usr/local/bin/start-systemuser.sh"]

COPY systemuser.sh /usr/local/bin/start-systemuser.sh

# install all extra packages at last to ensure cache works well
COPY ./requirements.txt /opt/requirements.txt

ARG PIP_EXTRA

RUN pip install --no-cache ${PIP_EXTRA} TA-lib

RUN conda update numba

RUN pip install --no-cache ${PIP_EXTRA} -r /opt/requirements.txt

RUN ${CONDA_DIR}/envs/python2/bin/pip install --no-cache ${PIP_EXTRA} -r /opt/requirements.txt

#COPY ./submodule /opt/submodule
#
#RUN cd /opt/submodule && \
#ls -F1 | sed 's/\///g' | xargs -I {} bash -c "echo {} && cd {} && pip --no-cache install -q --force-reinstall ." && \
#cd ..