FROM node:10-alpine

EXPOSE 3000

WORKDIR /usr/src/app

# here * means copy if there else file doesn't exit don't fail the run
COPY package.json package-lock.json* ./  

RUN npm install && npm cache clean --force

COPY . .

CMD [ "node", "./bin/www" ]
