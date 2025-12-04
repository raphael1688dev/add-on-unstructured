ARG BUILD_FROM
FROM downloads.unstructured.io/unstructured-io/unstructured-api:latest

LABEL io.hass.version="1.0.2" \
      io.hass.type="addon" \
      io.hass.arch="amd64|aarch64"

USER root

ENV HF_HOME="/data/huggingface"
ENV NLTK_DATA="/data/nltk_data"
ENV TORCH_HOME="/data/torch"

RUN mkdir -p /data/huggingface /data/nltk_data /data/torch

EXPOSE 8000
