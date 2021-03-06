FROM heroku/heroku:18-build

ENV DEBIAN_FRONTEND noninteractive
RUN mkdir /app
RUN addgroup --quiet --gid 2000 slug && \
    useradd slug --uid=2000 --gid=2000 --home-dir /app --no-create-home

# disable source repos (speeds up apt-get update)
RUN sed -i -e 's/^deb-src/#deb-src/' /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y md5deep && \
    # cleanup
    apt-get autoremove -y && \
    apt-get clean -y && \
    # package up license files if any by appending to existing tar
    rm -rf \
        /usr/share/doc \
        /usr/share/man \
        /usr/share/info \
        /usr/share/locale \
        /var/lib/apt/lists/* \
        /var/log/* \
        /var/cache/debconf/* \
        /etc/systemd \
        /lib/lsb \
        /lib/udev \
        /usr/lib/x86_64-linux-gnu/gconv/IBM* \
        /usr/lib/x86_64-linux-gnu/gconv/EBC* && \
    bash -c "mkdir -p /usr/share/man/man{1..8}"

ADD . /
ENV PYTHONPATH $PYTHONPATH:/usr/local/lib/python3/site-packages
COPY --from=quay.io/pngmbh/object-storage-cli:latest /objstorage-latest-linux-amd64 /bin/objstorage
RUN chmod +x /bin/objstorage && \
    chown -R slug:slug /app && \
    chown slug:slug /bin/get_object \
                    /bin/normalize_storage \
                    /bin/put_object \
                    /bin/objstorage \
                    /bin/read_procfile_keys \
                    /bin/restore_cache \
                    /bin/store_cache

USER slug
ENV HOME /app
RUN /builder/install-buildpacks

ENTRYPOINT ["/builder/build.sh"]
