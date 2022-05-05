FROM python:3.7-alpine

WORKDIR /usr/src/app

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install dependencies and netcat for entrypoint.sh
RUN apk update && \
    apk add --virtual build-deps gcc python3-dev musl-dev \
    postgresql-dev postgresql-libs jpeg-dev zlib-dev \
    freetype-dev lcms2-dev \ 
    openjpeg-dev tiff-dev tk-dev \
    tcl-dev harfbuzz-dev fribidi-dev \
    graphviz g++ make swig linux-headers graphviz-dev \
    postgresql-libs libstdc++
    
COPY ./requirements.txt .

RUN pip install -r requirements.txt

COPY ./entrypoint.sh /usr/src/app/entrypoint.sh

COPY ./django_app . 

RUN apk del build-deps && \
    apk add netcat-openbsd postgresql-libs && \
    rm requirements.txt && \
    chmod +x /usr/src/app/entrypoint.sh
