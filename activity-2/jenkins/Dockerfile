FROM jenkins/jenkins:lts-jdk17

USER root

# System dependencies
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    python3-venv \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

# Node.js 20 (required by cdxgen)
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

# Semgrep (isolated in a venv to avoid system pip conflicts)
RUN python3 -m venv /opt/semgrep \
    && /opt/semgrep/bin/pip install semgrep \
    && ln -s /opt/semgrep/bin/semgrep /usr/local/bin/semgrep

# cdxgen
RUN npm install -g @cyclonedx/cdxgen

USER jenkins
