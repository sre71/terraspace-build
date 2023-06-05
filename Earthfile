VERSION 0.7

FROM ruby:3.2.2-alpine3.18
WORKDIR /terraspace

build:
    BUILD --platform=linux/amd64 --platform=linux/arm64/v8 +build-platform

build-aws:
    ARG TARGETPLATFORM
    ARG TARGETARCH
    ARG TARGETVARIANT
    FROM --platform=$TARGETPLATFORM ruby:3.2.2-alpine3.18
    WORKDIR /awscliinstall
    RUN apk add --no-cache \
        curl \
        make \
        cmake \
        gcc \
        g++ \
        libc-dev \
        libffi-dev \
        openssl-dev \
        python3-dev \
        py3-pip \
        py3-setuptools \
        libc6-compat \
        file \
        bash \
        && curl https://awscli.amazonaws.com/awscli-2.10.1.tar.gz | tar -xz \
        && cd awscli-2.10.1 \
        && ./configure --prefix=/opt/aws-cli/ --with-download-deps \
        && make \
        && make install

build-platform:
    ARG TARGETPLATFORM
    ARG TARGETARCH
    ARG TARGETVARIANT
    FROM --platform=$TARGETPLATFORM +build-aws
    WORKDIR /terraspace
    COPY .terraformrc /root/
    COPY .gemrc /root/
    RUN mkdir -p /usr/share/terraform/providers \
        && mkdir -p /root/.terraform.d/plugin-cache \
        && curl -L https://raw.githubusercontent.com/warrensbox/terraform-switcher/release/install.sh | bash \
        && /usr/local/bin/tfswitch -d 1.2.4 -b /usr/local/bin/terraform
    COPY Gemfile /terraspace/
    RUN bundle update
    RUN terraspace new shim
    SAVE IMAGE --push sre71/terraspace:2.2.6
