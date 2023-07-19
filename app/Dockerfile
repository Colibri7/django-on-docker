# app/Dockerfile

FROM python:3.8.3-alpine as builder

WORKDIR /usr/src/app

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

RUN apk update \
    && apk add postgresql-dev gcc python3-dev musl-dev

RUN pip install --upgrade pip
COPY . .
COPY ./requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /usr/src/app/wheels -r requirements.txt

FROM python:3.8.3-alpine
RUN mkdir -p /home/app/web
WORKDIR /home/app/web

RUN addgroup -S app && adduser -S app -G app

# install dependencies
RUN pip install --upgrade pip
RUN apk update && apk add libpq
COPY --from=builder /usr/src/app/wheels /wheels
COPY --from=builder /usr/src/app/requirements.txt .
RUN pip install --no-cache /wheels/*
RUN pip install gunicorn

# create the appropriate directories
ENV HOME=/home/app
ENV APP_HOME=/home/app/web

WORKDIR $APP_HOME

# copy project
COPY . .

# change to the app user
USER app
RUN mkdir -p /home/app/web/staticfiles
RUN mkdir -p /home/app/web/mediafiles
ENTRYPOINT ["/home/app/web/entrypoint.sh"]

CMD ["gunicorn", "hello_django.wsgi:application", "--bind", "0.0.0.0:8000"]
