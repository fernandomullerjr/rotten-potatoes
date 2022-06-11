FROM python:3.6.8-alpine3.9

LABEL MAINTAINER="Fernando Muller <fernandomj90@gmail.com>"

ENV GROUP_ID=1000 \
    USER_ID=1000

WORKDIR /app
ADD . /app

RUN pip install --upgrade pip
RUN pip install -U pip setuptools wheel
RUN pip install markupsafe
RUN pip install flask

RUN apk add --no-cache jpeg-dev zlib-dev
RUN apk add --no-cache --virtual .build-deps build-base linux-headers \
    && pip install Pillow

# RUN export PYTHONPATH="$PYTHONPATH:/app"

COPY app.py /app/app.py

RUN pip install --no-cache-dir -r /app/requirements.txt
RUN pip install gunicorn

RUN addgroup -g $GROUP_ID www
RUN adduser -D -u $USER_ID -G www www -s /bin/sh

# Change current user to www
USER www

EXPOSE 5000

#CMD ["python","app.py"]
#CMD [ "gunicorn", "-w", "4", "--bind", "0.0.0.0:5000", "app.app:app"]
#CMD [ "gunicorn", "-w", "4", "--bind", "0.0.0.0:5000", "wsgi"]
CMD [ "gunicorn", "-w", "4", "--bind", "0.0.0.0:5000", "app:app"]