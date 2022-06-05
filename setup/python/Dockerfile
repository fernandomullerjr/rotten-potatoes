FROM python:3.6.1-alpine

LABEL MAINTAINER="Fernando Muller <fernandomj90@gmail.com>"

ENV GROUP_ID=1000 \
    USER_ID=1000

RUN pip install --upgrade pip
RUN pip install -U pip setuptools wheel
RUN pip install markupsafe
RUN pip install flask
RUN pip install gunicorn
RUN apk add --no-cache jpeg-dev zlib-dev
RUN apk add --no-cache --virtual .build-deps build-base linux-headers \
    && pip install Pillow
COPY ./src/app.py /app/app.py
COPY ./src/requirements.txt /app/requirements.txt
WORKDIR /app
RUN pip install --no-cache-dir -r /app/requirements.txt
COPY ./src .
#CMD ["python","app.py"]

RUN addgroup -g $GROUP_ID www
RUN adduser -D -u $USER_ID -G www www -s /bin/sh

USER www

EXPOSE 5000

CMD [ "gunicorn", "-w", "4", "--bind", "0.0.0.0:5000", "app.wsgi_app"]