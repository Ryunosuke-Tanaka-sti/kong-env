FROM python:3.12.8-bullseye

# パッケージ更新
RUN apt-get update

# 環境変数(UID, GIDについてはidコマンドでdockerの実行ユーザーと合わせた方がよい)
ARG USERNAME=pyuser
ARG GROUPNAME=pyuser
ARG UID=1000
ARG GID=1000
ENV WORKDIR=/home/pyuser/python

# タイムゾーン設定
ENV TZ Asia/Tokyo
# import時に指定ディレクトリからにするためラク
ENV PYTHONPATH $WORKDIR

# ユーザー追加
RUN groupadd -g $GID $GROUPNAME && \
    useradd -m -s /bin/bash -u $UID -g $GID $USERNAME

# フォルダ作成＆権限者設定
RUN mkdir -p $WORKDIR
RUN chown -R $UID:$GID $WORKDIR

# ユーザーのbinaryディレクトリをパスに追加
ENV PATH /home/$USERNAME/.local/bin:$PATH

# ユーザー切り替え
USER $USERNAME

# 作業ディレクトリ設定
WORKDIR $WORKDIR

# パッケージ更新
RUN python -m pip install --upgrade --user pip
RUN python -m pip install --upgrade --user setuptools

# パッケージインストール
COPY requirements.txt $WORKDIR/
RUN python -m pip install --user -r requirements.txt