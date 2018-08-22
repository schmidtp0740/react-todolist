FROM node:6.11.1
WORKDIR /usr/src/backend/app

COPY package.json .
RUN npm install
RUN npm install -g nodemon
COPY . .

EXPOSE 6200
CMD [ "npm", "start" ]