ARG alpine_version=3.8
ARG alpine_glibc_version=2.28-r0

# extract libgcc_s from the same base image than the one used to build glibc
ARG alpine_pkg_glibc_image=ubuntu:16.04

FROM ${alpine_pkg_glibc_image} as libgcc
RUN chmod +x /lib/x86_64-linux-gnu/libz.so.1.2.8

FROM alpine:${alpine_version}

ARG alpine_glibc_version
ARG vcs_ref
ARG image_version
ARG build_date

LABEL org.label-schema.schema-version="1.0" \
      org.label-schema.name="JRE ready image" \
      org.label-schema.url="https://github.com/xfournet/jready" \
      org.label-schema.vcs-url="https://github.com/xfournet/jready.git" \
      org.label-schema.vcs-ref="$vcs_ref" \
      org.label-schema.version="$image_version" \
      org.label-schema.build-date="$build_date"

ENV LANG=C.UTF-8

# Add the extra libs
COPY --from=libgcc /lib/x86_64-linux-gnu/libgcc_s.so.1 /lib/x86_64-linux-gnu/libz.so.1.2.8 /usr/glibc-compat/lib/

RUN  \
# Download and install the 3 glibc apks
    ALPINE_GLIBC_BASE_URL="https://github.com/sgerrand/alpine-pkg-glibc/releases/download/${alpine_glibc_version}" && \
    ALPINE_GLIBC_BASE_PACKAGE_FILENAME="glibc-${alpine_glibc_version}.apk" && \
    ALPINE_GLIBC_BIN_PACKAGE_FILENAME="glibc-bin-${alpine_glibc_version}.apk" && \
    ALPINE_GLIBC_I18N_PACKAGE_FILENAME="glibc-i18n-${alpine_glibc_version}.apk" && \
    echo \
        "-----BEGIN PUBLIC KEY-----\
        MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEApZ2u1KJKUu/fW4A25y9m\
        y70AGEa/J3Wi5ibNVGNn1gT1r0VfgeWd0pUybS4UmcHdiNzxJPgoWQhV2SSW1JYu\
        tOqKZF5QSN6X937PTUpNBjUvLtTQ1ve1fp39uf/lEXPpFpOPL88LKnDBgbh7wkCp\
        m2KzLVGChf83MS0ShL6G9EQIAUxLm99VpgRjwqTQ/KfzGtpke1wqws4au0Ab4qPY\
        KXvMLSPLUp7cfulWvhmZSegr5AdhNw5KNizPqCJT8ZrGvgHypXyiFvvAH5YRtSsc\
        Zvo9GI2e2MaZyo9/lvb+LbLEJZKEQckqRj4P26gmASrZEPStwc+yqy1ShHLA0j6m\
        1QIDAQAB\
        -----END PUBLIC KEY-----" | sed 's/   */\n/g' > "/etc/apk/keys/sgerrand.rsa.pub" && \
    wget \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_I18N_PACKAGE_FILENAME" && \
    apk add --no-cache \
        "$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_I18N_PACKAGE_FILENAME" && \
    rm  -r \
        "/etc/apk/keys/sgerrand.rsa.pub" \
        "$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_I18N_PACKAGE_FILENAME" && \
    \
# Set the locale
    /usr/glibc-compat/bin/localedef --force --inputfile POSIX --charmap UTF-8 "$LANG" || true && \
    echo "export LANG=$LANG" > /etc/profile.d/locale.sh && \
    \
    apk del --no-cache glibc-i18n && \
    \
# Fix ldd
    sed -i -e 's#/bin/bash#/bin/sh#' /usr/glibc-compat/bin/ldd && \
    sed -i -e 's#RTLDLIST=.*#RTLDLIST="/usr/glibc-compat/lib/ld-linux-x86-64.so.2"#' /usr/glibc-compat/bin/ldd && \
    \
# Isolate the glibc lib's from musl-libc lib's
    rm \
        /lib/ld-linux-x86-64.so.2 \
        /etc/ld.so.cache && \
    echo "/usr/glibc-compat/lib" > /usr/glibc-compat/etc/ld.so.conf && \
    /usr/glibc-compat/sbin/ldconfig -i && \
    rm -r \
        /var/cache/ldconfig/aux-cache \
        /var/cache/ldconfig
