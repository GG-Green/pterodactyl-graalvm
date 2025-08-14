# ----------------------------------
# Pterodactyl Core Dockerfile
# Environment: Java
# Minimum Panel Version: 1.7.0
# ----------------------------------
FROM ubuntu:24.04

ARG TARGETPLATFORM
ARG GRAAL_VERSION=21.0.2
ARG JAVA_VERSION=21
# Define the Graal.Python version to download
ARG GRAALPYTHON_VERSION=24.2.2

MAINTAINER RikoDEV, <kontakt@riko.dev>

ENV DEBIAN_FRONTEND=noninteractive

# Default to UTF-8 file.encoding
ENV LANG='en_US.UTF-8' LANGUAGE='en_US:en' LC_ALL='en_US.UTF-8'

RUN apt-get update -y \
  && apt-get install -y --no-install-recommends curl ca-certificates openssl git tar sqlite3 fontconfig tzdata locales iproute2 \
  && echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
  && locale-gen en_US.UTF-8 \
  && case ${TARGETPLATFORM} in \
          "linux/amd64")   ARCH=x64  ;; \
          "linux/arm64")   ARCH=aarch64  ;; \
     esac \
  && curl --retry 3 -Lfso /tmp/graalvm.tar.gz https://github.com/graalvm/graalvm-ce-builds/releases/download/jdk-${GRAAL_VERSION}/graalvm-community-jdk-${GRAAL_VERSION}_linux-${ARCH}_bin.tar.gz \
  && mkdir -p /opt/java/graalvm \
  && cd /opt/java/graalvm \
  && tar -xf /tmp/graalvm.tar.gz --strip-components=1 \
  && export PATH="/opt/java/graalvm/bin:$PATH" \
  && rm -rf /var/lib/apt/lists/* \
  && rm -rf /tmp/graalvm.tar.gz

# Add Graal.Python installation and configuration
RUN case ${TARGETPLATFORM} in \
          "linux/amd64")   ARCH=x64  ;; \
          "linux/arm64")   ARCH=aarch64  ;; \
     esac \
  # Download the Graal.Python component from its official release page
  && curl --retry 3 -Lfso /tmp/graalpython.tar.gz https://github.com/oracle/graalpython/releases/download/graal-${GRAALPYTHON_VERSION}/graalpy-community-jvm-${GRAALPYTHON_VERSION}-linux-${ARCH}.tar.gz \
  # Extract the Graal.Python files into the GraalVM directory
  && tar -xf /tmp/graalpython.tar.gz --strip-components=1 -C /opt/java/graalvm/languages \
  && rm -rf /tmp/graalpython.tar.gz

# Set the environment variables for Graal.Python so it can find its components
ENV JAVA_HOME=/opt/java/graalvm \
    PATH="/opt/java/graalvm/bin:$PATH" \
    python.CoreHome=/opt/java/graalvm/languages/graalpython \
    python.SysPrefix=/opt/java/graalvm/languages/graalpython \
    python.StdLibHome=/opt/java/graalvm/languages/graalpython/lib-graalpython \
    python.CAPI=/opt/java/graalvm/languages/graalpython/lib/native/libgraalpython_capi.so \
    python.JNILibrary=/opt/java/graalvm/languages/graalpython/lib/native/libgraalpython_jni.so

# Step 2 - add pterodactyl stuff
RUN useradd -d /home/container -m container

USER        container
ENV         USER=container HOME=/home/container

WORKDIR     /home/container

COPY        ./../entrypoint.sh /entrypoint.sh

CMD         ["/bin/bash", "/entrypoint.sh"]
