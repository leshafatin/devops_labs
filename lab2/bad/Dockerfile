FROM node:latest

WORKDIR /app

COPY . /app
COPY package.json package-lock.json /app/

RUN npm install --package-lock-only \
 && npm ci \ 
 && npm install -g serve \
 && npm run build

CMD serve -s build