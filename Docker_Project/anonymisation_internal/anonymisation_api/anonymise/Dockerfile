FROM python:3.7-slim


RUN apt-get -y update && apt-get install -y unixodbc-dev gcc g++ &&\
   apt-get install -y --no-install-recommends build-essential  \
   curl wget nginx ca-certificates npm git git-lfs \
   && npm install pm2 -g \
   && pip install --upgrade pip setuptools \
   && rm -rf /var/lib/apt/lists/*
RUN pip install dataclasses
RUN pip install datasets
# RUN pip install transformers>=4.1
# RUN pip install torch
RUN pip install seqeval
RUN pip install nltk
RUN pip install Flask gunicorn celery Redis
RUN pip install importlib-metadata==4.13.0
COPY requirements.txt .
RUN pip install -r requirements.txt


COPY . ./

RUN mkdir /models && \
   mkdir -p /app/logs && \
   mkdir -p /app/training/trained_model

RUN <<EOF cat > /entrypoint.sh
#!/bin/sh
set -e

echo "Downloading models..."
git clone https://huggingface.co/buerokratt/bert-truecaser /models/bert-truecaser
git clone https://huggingface.co/buerokratt/ner_old /models/bert_old
git clone https://huggingface.co/buerokratt/ner_new /models/bert_new
git clone https://huggingface.co/buerokratt/ner_gdpr /models/gdpr_model

echo "Running git lfs pull..."
cd /models/bert-truecaser && git lfs install && git lfs pull
cd /models/bert_old && git lfs install && git lfs pull
cd /models/bert_new && git lfs install && git lfs pull
cd /models/gdpr_model && git lfs install && git lfs pull

cd /

echo "Starting gunicorn..."

exec "\$@"
EOF

RUN <<EOF cat > /startup-probe.sh
#!/bin/sh
curl -f http://localhost:8000/healthz || exit 1
curl -f http://localhost/ || exit 1
EOF

RUN chmod +x /entrypoint.sh

EXPOSE 5001

#
CMD ["gunicorn", "accept_request:app", "--bind", ":5001", "--timeout", "900", "--log-file", "/app/logs/gunicorn.log" , "--log-level" , "DEBUG" ]
#CMD ["python3", "accept_request.py"]

ENTRYPOINT ["/entrypoint.sh"]