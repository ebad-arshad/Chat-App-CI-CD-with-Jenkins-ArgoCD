FROM node:24-alpine

WORKDIR /app

COPY package*.json /app

RUN npm ci

COPY . .

EXPOSE 3000

CMD ["node", "src/server.js"]