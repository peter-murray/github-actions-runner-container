ARG VERSION=latest
ARG BASE=ubi8/ubi
FROM registry.access.redhat.com/$BASE:$VERSION

# You would typically obtain this latest version from an API endpoint and use that for the runner version
# as the runner will self update to the latest version when it gets its first job.
# The reason this is specified is so that we can test the upgrade scenarios using this Dockerfile.
ARG GH_RUNNER_VERSION=2.286.1

ARG COMMIT_SHA=main
ARG CREATED=""

ENV RUNNER_NAME=""
ENV GITHUB_SERVER=""
ENV GITHUB_TOKEN=""
ENV RUNNER_LABELS=""
ENV RUNNER_OPTIONS=""
ENV RUNNER_WORK_DIRECTORY="_work"
ENV RUNNER_ALLOW_RUNASROOT=false
ENV AGENT_TOOLS_DIRECTORY=/opt/hostedtoolcache

# Fill in your labels as appropriate here
LABEL \
    org.opencontainers.image.created="$CREATED" \
    org.opencontainers.image.authors="Peter Murray" \
    org.opencontainers.image.url=https://github.com/peter-murray/github-actions-runner-container \
    org.opencontainers.image.documentation=https://github.com/peter-murray/github-actions-runner-container/README.md \
    org.opencontainers.image.source=https://github.com/peter-murray/github-actions-runner-container \
    org.opencontainers.image.version=$GH_RUNNER_VERSION \
    org.opencontainers.image.revision=$COMMIT_SHA \
    org.opencontainers.image.vendor="Peter Murray" \
    org.opencontainers.image.licenses=MIT \
    org.opencontainers.image.ref.name=centos-actions-runner \
    org.opencontainers.image.title="GitHub Actions Runner Container - CentOS" \
    org.opencontainers.image.description="GitHub Actions Runner built into a Container to provide self-hosted runners for Enterprise, Organization or Repositories" \
    github_actions_version=$GH_RUNNER_VERSION

# Create a user for running actions
RUN useradd -m actions
RUN mkdir -p /home/actions ${AGENT_TOOLS_DIRECTORY}
WORKDIR /home/actions

# jq is used by the runner to extract the token when registering the runner
RUN curl -L -O https://github.com/actions/runner/releases/download/v${GH_RUNNER_VERSION}/actions-runner-linux-x64-${GH_RUNNER_VERSION}.tar.gz \
    && tar -zxf actions-runner-linux-x64-${GH_RUNNER_VERSION}.tar.gz \
    && rm -f actions-runner-linux-x64-${GH_RUNNER_VERSION}.tar.gz \
    && ./bin/installdependencies.sh \
    && yum install jq git -y \
    && yum clean all

# Copy out the runsvc.sh script to the root directory for running the service
RUN cp bin/runsvc.sh . && chmod +x ./runsvc.sh

COPY common/entrypoint.sh .
RUN chmod +x ./entrypoint.sh

# Now that the OS has been updated to include required packages, update ownership and then switch to actions user
RUN chown -R actions:actions /home/actions ${AGENT_TOOLS_DIRECTORY}

USER actions
CMD [ "./entrypoint.sh" ]
