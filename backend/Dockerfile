FROM python:3.9-slim

RUN apt-get update

RUN apt-get install -y make

WORKDIR /app

RUN pip install gunicorn==20.1.0

COPY requirements.txt ./

RUN pip install -r requirements.txt --no-cache-dir

COPY . .

CMD ["make", "start"]
