FROM alpine:latest
ENV MADDY_VERSION=0.7.0
ENV MADDY_TARGET=maddy-$MADDY_VERSION-x86_64-linux-musl
WORKDIR /srv/maddy
RUN apk update && apk upgrade && apk add \
  tar \
  curl \
  zstd
RUN curl -LO https://github.com/foxcpp/maddy/releases/download/v$MADDY_VERSION/$MADDY_TARGET.tar.zst && \
  tar --zstd -xvf $MADDY_TARGET.tar.zst && \
  cp $MADDY_TARGET/maddy .

FROM alpine:latest
WORKDIR /srv
COPY --from=0 /srv/maddy/maddy ./maddy
ENTRYPOINT ["./maddy"]
CMD ["run"]
