FROM python:3.10-slim-bookworm AS base

LABEL maintainer="mapproxy.org"

# The MAPPROXY_VERSION argument can be used like this to overwrite the default:
# docker build --build-arg MAPPROXY_VERSION=1.15.1 [--target base|development|nginx] -t mapproxy:1.15.1 .
ARG MAPPROXY_VERSION=1.16.0

RUN apt update && apt -y install --no-install-recommends \
  python3-pil \
  python3-yaml \
  python3-pyproj \
  libgeos-dev \
  python3-lxml \
  libgdal-dev \
  python3-shapely \
  libxml2-dev libxslt-dev && \
  apt-get -y --purge autoremove && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/*

RUN mkdir /mapproxy

WORKDIR /mapproxy

# fix potential issue finding correct shared library libproj (fixed in newer releases)
RUN ln -s /usr/lib/`uname -m`-linux-gnu/libproj.so /usr/lib/`uname -m`-linux-gnu/liblibproj.so

RUN pip install MapProxy==$MAPPROXY_VERSION && \
    # temporary fix for v1.15.1
    if [ "$MAPPROXY_VERSION" = '1.15.1' ]; then pip install six; fi && \
    pip cache purge

COPY app.py .

COPY start.sh .

ENTRYPOINT ["bash", "-c", "./start.sh base"]

###### development image ######

FROM base AS development

EXPOSE 8080

ENTRYPOINT ["bash", "-c", "./start.sh development"]

##### nginx image ######

FROM base AS nginx

RUN apt update && apt -y install --no-install-recommends nginx gcc

# cleanup
RUN apt-get -y --purge autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN pip install uwsgi pyproj && \
    pip cache purge

COPY uwsgi.conf .

COPY nginx-default.conf /etc/nginx/sites-enabled/default
COPY origin-cert.pem /etc/ssl/certs/
COPY origin-key.key /etc/ssl/private/

EXPOSE 80

ENTRYPOINT ["bash", "-c", "./start.sh nginx"]
