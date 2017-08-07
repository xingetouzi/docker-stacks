# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
FROM jupyterhub/systemuser

USER root

# update apt-get source with 163
RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak && \
    echo "deb http://mirrors.163.com/debian/ jessie main non-free contrib" >/etc/apt/sources.list && \
    echo "deb http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list && \
    echo "deb-src http://mirrors.163.com/debian/ jessie main non-free contrib" >>/etc/apt/sources.list && \
    echo "deb-src http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list

# update pip source
RUN mkdir -p ~/.pip && \
echo "[global]" > ~/.pip/pip.conf && \
echo "index-url = http://mirrors.aliyun.com/pypi/simple/" >> ~/.pip/pip.conf && \
echo "" >> ~/.pip/pip.conf && \
echo "[install]" >> ~/.pip/pip.conf && \
echo "trusted-host=mirrors.aliyun.com" >> ~/.pip/pip.conf

# activate ipyparallel-mesos
RUN pip install git+git://github.com/xingetouzi/ipyparallel-mesos.git && \
pip install jupyter_contrib_nbextensions && \
pip install jupyter_nbextensions_configurator

# update conda settings
RUN conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/ --system && \
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/ --system && \
conda config --set show_channel_urls yes --system && \
conda config --set allow_softlinks False --system

# Install Python 3 packages
RUN conda install --quiet --yes \
    'ipyparallel=6.0.*' \
    'nb_conda' \
    && conda clean -tipsy

# Install Python 2 packages
RUN $CONDA_DIR/envs/python2/bin/conda install --quiet --yes \
    'ipyparallel=6.0.*' \
    && conda clean -tipsy

RUN jupyter nbextension install --sys-prefix --py ipyparallel  &&\
    jupyter nbextension enable --sys-prefix --py ipyparallel  &&\
    jupyter serverextension enable --sys-prefix --py ipyparallel &&\
    jupyter contrib nbextension install --sys-prefix && \
    jupyter nbextensions_configurator enable --sys-prefix &&\
    jupyter nbextensions_configurator enable --system

COPY systemuser.sh /usr/local/bin/start-systemuser.sh

ENTRYPOINT ["tini", "--"]

CMD ["sh", "/usr/local/bin/start-systemuser.sh"]

# install all extra packages at last to ensure cache works well
COPY ./requirements.txt /opt/requirements.txt

RUN pip install && ${CONDA_DIR}/envs/python2/bin/pip install -r /opt/requirements.txt