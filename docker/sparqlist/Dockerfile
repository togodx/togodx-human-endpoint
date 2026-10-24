# Customized Dockerfile for https://github.com/dbcls/sparqlist
# Using rootless docker frees from switching to a non-root user in the Dockerfile.

FROM node:24.22.1-bookworm

ARG SPARQLIST_REF=main

ENV ROOT_PATH=/human/sparqlist/
ENV REPOSITORY_PATH=/app/sparqlist/repository

WORKDIR /app
RUN git clone --depth 1 --branch ${SPARQLIST_REF} https://github.com/dbcls/sparqlist
COPY . .

WORKDIR /app/sparqlist
RUN npm -g install npm
#RUN npm audit fix
RUN npx browserslist@latest --update-db
RUN npm ci && npm audit fix --omit=dev && npm run build && npm prune --omit=dev && rm -rf frontend/node_modules
CMD npm start
