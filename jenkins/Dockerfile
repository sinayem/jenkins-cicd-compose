# Custom Jenkins controller image:
#   - Docker CLI only (the daemon runs in the separate DinD container)
#   - Plugins pre-installed from plugins.txt
#   - Configuration-as-Code file baked in
FROM jenkins/jenkins:lts-jdk17

# Temporarily switch to root to install OS packages.
USER root

# Install the Docker CLI from Docker's official apt repository.
RUN apt-get update && \
    apt-get install -y --no-install-recommends ca-certificates curl && \
    install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc && \
    chmod a+r /etc/apt/keyrings/docker.asc && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
      > /etc/apt/sources.list.d/docker.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends docker-ce-cli && \
    rm -rf /var/lib/apt/lists/*

# JCasC file lives outside JENKINS_HOME so an image rebuild always applies
# the latest version (JENKINS_HOME is a persistent volume).
RUN mkdir -p /var/jenkins_conf && chown jenkins:jenkins /var/jenkins_conf
COPY --chown=jenkins:jenkins casc.yaml /var/jenkins_conf/casc.yaml

# Drop back to the unprivileged jenkins user (UID 1000) for runtime.
USER jenkins

# Pre-install plugins so the controller is reproducible.
COPY --chown=jenkins:jenkins plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt
