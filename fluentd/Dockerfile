FROM fluent/fluentd:v1.13-debian-1

USER root
RUN echo "root:Docker!" | chpasswd

RUN buildDeps="sudo make gcc g++ libc-dev curl" \
 && apt-get update \
 && apt-get install -y --no-install-recommends $buildDeps \
 && sudo gem install fluent-plugin-grafana-loki \
 && sudo gem install fluent-plugin-netflow \
 && sudo gem sources --clear-all \
 && SUDO_FORCE_REMOVE=yes \
    apt-get purge -y --auto-remove \
                  -o APT::AutoRemove::RecommendsImportant=false \
                  $buildDeps \
 && rm -rf /var/lib/apt/lists/* \
 && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem

USER fluent
