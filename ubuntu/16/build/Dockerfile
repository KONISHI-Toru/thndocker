FROM thndocker/ubuntu16 AS builder

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       build-essential \
       software-properties-common \
    && rm -rf /var/lib/apt/lists/*


CMD ["mysqld"]
