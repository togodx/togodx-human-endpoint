# Customized Dockerfile for https://github.com/dbcls/sparqlist
# Using rootless docker frees from switching to a non-root user in the Dockerfile.

FROM node:18

ENV ROOT_PATH=/human/sparqlist/
ENV REPOSITORY_PATH=/app/sparqlist/repository

WORKDIR /app
RUN git clone https://github.com/dbcls/sparqlist
COPY . .

WORKDIR /app/sparqlist
RUN npm -g install npm
RUN npm audit fix
RUN npx browserslist@latest --update-db
RUN npm ci --production && npm run build
RUN npm run build
CMD npm start
