FROM apache/airflow:2.9.0 AS base

COPY ./requirements.txt /requirements.txt
RUN pip install --no-cache-dir "apache-airflow==2.9.0" -r /requirements.txt

FROM base AS final

COPY ./dags /opt/airflow/dags
COPY ./plugins /opt/airflow/plugins
COPY ./config /opt/airflow/config
