# Stage 1: builder
FROM python:3.12-slim AS builder
WORKDIR /src
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .

# Stage 2: runtime
FROM python:3.12-slim
RUN useradd -m appuser
WORKDIR /app
COPY --from=builder /src /app
USER appuser
EXPOSE 8080
CMD ["python", "app.py"]