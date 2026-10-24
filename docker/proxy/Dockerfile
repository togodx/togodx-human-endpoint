FROM ghcr.io/dbcls/sparql-proxy@sha256:018af42fd87a5619fc5a084b073e47325fc0ec92224e75e18b686957f56a4fc2

ENV ROOT_PATH /human/proxy/

RUN npm install && npm audit fix && npm run build && npm prune --omit=dev
