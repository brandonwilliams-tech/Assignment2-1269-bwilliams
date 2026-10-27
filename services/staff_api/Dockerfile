########################
# Builder Stage
########################
# Full Python image: has the tools to install packages
FROM python:3.14 AS builder
WORKDIR /app

# Copy requirements first so this layer is cached until requirements change
COPY requirements.txt .

# Install packages into one folder we can copy into the final stage
RUN pip install --no-cache-dir --target=/python-packages -r requirements.txt

########################
# Production Stage
########################

# Slim image: much smaller, no build tools
FROM python:3.14-slim

RUN apt-get update && \
    apt-get install -y curl wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Bring in only the installed packages from the builder stage
COPY --from=builder /python-packages /python-packages
ENV PYTHONPATH=/python-packages

# Remove pip and setuptools, the running app doesn't need them
RUN pip uninstall -y pip setuptools

# Create a non-root user (security: the app should not run as root)
RUN adduser --disabled-password --gecos '' appuser

# Copy only the files the app needs
COPY app.py ./

# Give the non-root user ownership so it can write app.log
RUN chown -R appuser:appuser /app
USER appuser

# The app listens on port 5000
EXPOSE 5000

CMD ["python", "-m", "flask", "--app", "app", "run", "--host=0.0.0.0", "--port=5000"]