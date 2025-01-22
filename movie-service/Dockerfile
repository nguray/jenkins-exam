FROM python:3.8-slim

ENV DATABASE_URI=postgresql://movie_db_username:movie_db_password@moviedb/movie_db_dev
ENV CAST_SERVICE_HOST_URL=http://cast_service:8000/api/v1/casts/

WORKDIR /app

COPY ./requirements.txt /app/requirements.txt

RUN apt-get update \
    && apt-get install gcc -y \
    && apt-get clean

RUN pip install -r /app/requirements.txt \
    && rm -rf /root/.cache/pip

COPY . /app/
