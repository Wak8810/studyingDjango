# Dockerfile (最終修正版)
FROM python:3.9.0-slim
ENV PYTHONUNBUFFERED=1

ARG UID=1000
ARG GID=1000
RUN groupadd --gid $GID django && \
    useradd --uid $UID --gid $GID -m django

# 作業ディレクトリを /app に変更
WORKDIR /app

USER django

COPY --chown=django:django requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY --chown=django:django . .

# 最後にCMDを追加して、このDockerfile自体も単体で機能するようにしておく
CMD ["python3", "manage.py", "runserver", "0.0.0.0:8000"]