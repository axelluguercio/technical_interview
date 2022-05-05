FROM node:latest

WORKDIR /usr/src/app 

COPY ./package*.json ./

RUN yarn install

COPY . .

ARG API_SERVER

ENV REACT_APP_API_SERVER=${API_SERVER}
RUN REACT_APP_API_SERVER=${API_SERVER} \
  yarn run build

RUN yarn global add serve