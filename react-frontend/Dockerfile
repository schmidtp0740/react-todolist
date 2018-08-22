FROM node:6.11.1
WORKDIR /usr/src/frontend/app

COPY package.json .
RUN npm install
COPY . .

EXPOSE 3000
CMD [ "npm", "start" ]

