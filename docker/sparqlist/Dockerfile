# Customized Dockerfile for https://github.com/dbcls/sparqlist

FROM node:18

ENV ROOT_PATH=/human/sparqlist/
ENV REPOSITORY_PATH=/app/sparqlist/repository

RUN npm -g install npm
RUN useradd --create-home --home-dir /app app

USER app
WORKDIR /app

RUN git clone https://github.com/dbcls/sparqlist
COPY --chown=app:app . .

WORKDIR /app/sparqlist

RUN npm ci --production && npm run build
RUN npm run build
CMD npm start
