# docker build --platform=linux/amd64 -t thndocker/ruby-him .
# docker push thndocker/ruby-him
FROM ruby:2.3.1

# railsコンソール中で日本語入力するための設定
ENV LANG=C.UTF-8

RUN echo "deb http://archive.debian.org/debian/ jessie main non-free contrib" > /etc/apt/sources.list && \
    echo "deb http://archive.debian.org/debian-security/ jessie/updates main non-free contrib" >> /etc/apt/sources.list

# /var/lib/apt/lists配下のキャッシュを削除し容量を小さくする
RUN apt-get update -qq && \
    apt-get install -y --force-yes --no-install-recommends \
       	    	       mariadb-client \
                       nodejs \
                       apt-transport-https lsb-release && \
    curl -sL https://deb.nodesource.com/setup_10.x | bash - && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/*

# システム時間を日本時間にするための設定
RUN rm /etc/localtime && \
    ln -s /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
