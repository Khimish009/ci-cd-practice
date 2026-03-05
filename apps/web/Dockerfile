FROM node:20-alpine

WORKDIR /app

COPY package.json .
COPY packages/ui/package.json .

RUN corepack enable pnpm
RUN pnpm install

COPY . .

RUN pnpm build

CMD [ "pnpm", "server.js" ]
