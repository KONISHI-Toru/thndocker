FROM ruby:3.2.2

# railsコンソール中で日本語入力するための設定
ENV LANG C.UTF-8
ENV TZ Asia/Tokyo

# /var/lib/apt/lists配下のキャッシュを削除し容量を小さくする
RUN apt-get update -qq \
    && apt-get -y clean \
    && rm -rf /var/lib/apt/lists/*

# 作業ディレクトリの設定
RUN mkdir /app
ENV APP_ROOT /app
WORKDIR $APP_ROOT

RUN gem install rails -v 7.1.3

CMD ["bash"]
