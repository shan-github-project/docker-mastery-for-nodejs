# fill in the blanks to dockerize this node app
FROM node:10.16.0-alpine

EXPOSE 3000

#The no cache part makes sure that it's nice and small.
RUN  apk add --no-cache tini

#workdir here can be whatever you're comfortable with.make it consistent across your projects
#and your company so that each one doesn't have a different
#location. That way you can just get used to where the files
#need to be. The sort of, some of the official standards
#you might see, some standards under home/node or source/node
WORKDIR /app

# wildcard here if the lock doesn't exist, it won't error out.That's why package*.json a shorter way.
# copy in multiple files so that's why dot and forward slash
COPY package*.json ./    

#clean force no leftover files that are downloaded from the npm package repositories.
#double ampersand means that the first command has to successfully install before it will do a clean. 
RUN npm install && npm cache clean --force

COPY . . 

ENTRYPOINT [ "tini", "--"]

CMD ["node", "app.js"]
