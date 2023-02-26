FROM node:18-slim as base
# labels from https://github.com/opencontainers/image-spec/blob/master/annotations.md
LABEL org.opencontainers.image.authors=shan-github@gmail.com
#LABEL org.opencontainers.image.created=$CREATED_DATE
#LABEL org.opencontainers.image.revision=$SOURCE_COMMIT
LABEL org.opencontainers.image.title="Node.js Ultimate Dockerfile"
#LABEL org.opencontainers.image.url=https://hub.docker.com/r/bretfisher/jekyll
#LABEL org.opencontainers.image.source=https://github.com/BretFisher/udemy-docker-mastery-for-nodejs
LABEL org.opencontainers.image.licenses=MIT
LABEL com.bretfisher.nodeversion=$NODE_VERSION
ENV NODE_ENV=production
EXPOSE 8080
ENV PORT 80
# You can install tini multiple ways. This way works in modern apt
# and works across platforms (amd, arm, etc)
RUN apt-get update -qq && apt-get install -qy \ 
    tini \
    --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*
WORKDIR /app
COPY package*.json ./
RUN npm config list
RUN npm ci \
    && npm cache clean --force
ENV PATH /app/node_modules/.bin:$PATH
ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["node", "server.js"]

FROM base as dev
# NOTE: these apt dependencies are only needed
# for testing. they shouldn't be in production
ENV NODE_ENV=development
RUN apt-get update -qq && apt-get install -qy \
    bzip2 \
    ca-certificates \
    curl \
    libfontconfig \
    --no-install-recommends
RUN npm config list
RUN npm install \
    && npm cache clean --force
USER node
CMD ["node","server.js"]

FROM dev as test
RUN npm audit
# --audit-level critical
ENV TRIVY_VERSION=0.35.0
# Use BuildKit to help translate architecture names
ARG TARGETPLATFORM
USER root
RUN case ${TARGETPLATFORM} in \
         "linux/amd64")    ARCH=64bit  ;; \
         "linux/arm64")    ARCH=ARM64  ;; \
         "linux/arm64/v8") ARCH=ARM64  ;; \
         "linux/arm/v7")   ARCH=ARM    ;; \
    esac \
    && apt-get -qq update \
    && apt-get -qq install -y ca-certificates wget --no-install-recommends \
    && wget --progress=dot:giga https://github.com/aquasecurity/trivy/releases/download/v${TRIVY_VERSION}/trivy_${TRIVY_VERSION}_Linux-${ARCH}.deb \
    && dpkg -i trivy_${TRIVY_VERSION}_Linux-${ARCH}.deb
COPY . .
RUN trivy fs --severity "HIGH,CRITICAL" --no-progress --security-checks vuln .
#IF above SECURITY CONFIG DIDN'T RUN THIS 2 COMMENTED COMMAND. LEAVE COPY COMMAND UNCOMMNETED     
# RUN curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin \ 
#   && trivy filesystem --exit-code 0 --no-progress /

FROM test as pre-prod
RUN rm -rf ./tests && rm -rf ./node_modules

FROM base as prod
COPY --from=pre-prod /app /app
HEALTHCHECK CMD curl http://127.0.0.1/ || exit 1
USER node