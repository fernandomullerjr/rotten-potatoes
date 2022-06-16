FROM python:3.10.1-slim

LABEL MAINTAINER="Fernando Müller <fernandomj90@gmail.com>"

RUN pip install --upgrade pip

WORKDIR /app

COPY ./requirements.txt ./requirements.txt
RUN pip3 install -r requirements.txt
RUN pip install gunicorn

COPY . .

EXPOSE 5000

ENV FLASK_APP=./app.py
ENV FLASK_ENV=development

CMD ["gunicorn", "-c", "config.py", "--bind", "0.0.0.0:5000", "app:app"]