# FROM node:24.11.1-trixie-slim
FROM node@sha256:fcdfd7bcd8f641c8c76a8950343c73912d68ba341e8dd1074e663b784d3e76f4


# Prepare the user
ARG DOCKER_UID=1000
RUN mkdir -p /app /home/app \
    && userdel node && rm -rf /home/node \
    && addgroup app \
    && useradd -g app --home /home/app --uid $DOCKER_UID app \
    && chown app:app /home/app \
    && chown app:app /app

WORKDIR /app
COPY . .
RUN npm install
RUN npm run build

USER app
CMD ["npm", "run", "start"]

