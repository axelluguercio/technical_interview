FROM node:latest

WORKDIR /usr/src/app 

# Copia las dependencias
COPY ./package*.json ./

RUN yarn install

COPY . .

ARG API_SERVER

# Buildea el entorno y agrega serve 
ENV REACT_APP_API_SERVER=${API_SERVER}
RUN REACT_APP_API_SERVER=${API_SERVER} \
  yarn run build

RUN yarn global add serve