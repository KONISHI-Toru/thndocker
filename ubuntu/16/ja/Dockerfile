FROM ubuntu:16.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install -y gnupg locales tzdata \
    && rm -rf /var/lib/apt/lists/* \
    && localedef -i ja_JP -c -f UTF-8 -A /usr/share/locale/locale.alias ja_JP.UTF-8

ENV LANG ja_JP.utf8

RUN rm -f /etc/localtime && \
    ln -s /usr/share/zoneinfo/Asia/Tokyo /etc/localtime


# Start the main process.
CMD ["bash"]


