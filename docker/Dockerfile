# Container for packages that need to be built from source but have massive dev dependencies.
FROM python:3.7.9-slim-buster as builder
RUN set -e && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        gcc \
        python3.7-dev
RUN pip wheel --wheel-dir=/root/wheels uwsgi==2.0.19.1 && \
    pip wheel --wheel-dir=/root/wheels regex==2020.11.13

# The actual container.
FROM python:3.7.9-slim-buster
RUN set -e && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        nginx \
        memcached \
        python3-rasterio \
        python3-pylibmc \
        supervisor && \
    rm -rf /var/lib/apt/lists/*

ENV PYTHONPATH="/usr/lib/python3/dist-packages:${PYTHONPATH}"

COPY --from=builder /root/wheels /root/wheels
COPY requirements.in /app/requirements.in
RUN pip install \
        --no-index \
        --no-cache-dir \
        --disable-pip-version-check \
        --find-links=/root/wheels \
        uwsgi regex && \
    pip install --no-cache-dir --disable-pip-version-check -r /app/requirements.in && \
        rm -rf /root/.cache/pip/* && \
        rm root/wheels/* && \
        rm /app/requirements.in

WORKDIR /app
COPY . /app/

RUN echo > /etc/nginx/sites-available/default && \
    cp /app/docker/nginx.conf /etc/nginx/conf.d/nginx.conf && \
    cp /app/docker/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

CMD ["sh", "/app/docker/run.sh"]
EXPOSE 5000
ENV CURL_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt