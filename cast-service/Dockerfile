FROM python:3.8-slim

ENV DATABASE_URI=postgresql://cast_db_username:cast_db_password@castdb/cast_db_dev

WORKDIR /app

COPY ./requirements.txt /app/requirements.txt

RUN apt-get update \
    && apt-get install gcc -y \
    && apt-get clean

RUN pip install -r /app/requirements.txt \
    && rm -rf /root/.cache/pip

COPY . /app/
