# checkov:skip=CKV_DOCKER_2: This is not a running container. Its purpose is to be used in CI pipeline jobs
# checkov:skip=CKV_DOCKER_3: Kaniko must be root to work
# checkov:skip=CKV_DOCKER_8: Kaniko must be root to work 

FROM alpine:3.16 as builder

ARG RELEASE_CLI=v0.11.0
ARG VAULT_VERSION=1.11.0
ARG ENVCONSUL_VERSION=0.12.1

SHELL ["/bin/ash", "-eo", "pipefail", "-c"]
RUN apk update --no-cache \
    && apk add curl npm unzip bash --no-cache --update \ 
    && curl -o /usr/local/bin/gitlab-helper -s "https://s3.eu-central-1.amazonaws.com/valmano-gitlab-helper/gitlab-helper" \
    && chmod +x /usr/local/bin/gitlab-helper \
    && curl -sSfLO "https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64" \
    && chmod +x ./argocd-linux-amd64 \
    && mv ./argocd-linux-amd64 /usr/local/bin/argocd \
    && curl -sSfLO "https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64" \
    && chmod +x ./kubectl-argo-rollouts-linux-amd64 \
    && mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts \
    && curl -sSfLO "https://github.com/argoproj/argo-workflows/releases/latest/download/argo-linux-amd64.gz" \
    && gunzip argo-linux-amd64.gz \
    && chmod +x argo-linux-amd64 \
    && mv ./argo-linux-amd64 /usr/local/bin/argo \
    && HELM_VERSION=$(curl -sSfL "https://github.com/kubernetes/helm/releases/latest" | grep -oE 'v[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq | tail -1) \
    && curl -sSfL "https://get.helm.sh/helm-${HELM_VERSION}-linux-amd64.tar.gz" | tar xz \
    && mv linux-amd64/helm /usr/local/bin/helm \
    && curl -sSfL "https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/linux/oc.tar.gz" -o /tmp/oc.tar.gz \
    && tar -xzf /tmp/oc.tar.gz -C /usr/local/bin \
    && rm /tmp/oc.tar.gz \
    && KUBECTL_VERSION=$(curl -sSfL "https://storage.googleapis.com/kubernetes-release/release/stable.txt") \
    && curl -sSfLO "https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" \
    && chmod +x ./kubectl \
    && mv ./kubectl /usr/local/bin/kubectl \
    && curl -sSfLO "https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_amd64.zip" \
    && unzip -qqo "./vault_${VAULT_VERSION}_linux_amd64.zip" -d /usr/local/bin \
    && curl -sSLO "https://releases.hashicorp.com/envconsul/${ENVCONSUL_VERSION}/envconsul_${ENVCONSUL_VERSION}_linux_amd64.zip" \
    && unzip -qqo "./envconsul_${ENVCONSUL_VERSION}_linux_amd64.zip" -d /usr/local/bin \
    && curl -sSfL "https://install-cli.jfrog.io" | sh \
    && curl -sSfL "https://raw.githubusercontent.com/anchore/grype/main/install.sh" | sh -s -- -b /usr/local/bin \
    && curl -sSfL "https://raw.githubusercontent.com/anchore/syft/main/install.sh" | sh -s -- -b /usr/local/bin \
    && curl -sLO "https://github.com/hadolint/hadolint/releases/latest/download/hadolint-Linux-x86_64" \
    && chmod +x hadolint-Linux-x86_64 \
    && mv ./hadolint-Linux-x86_64 /usr/local/bin/hadolint \
    && curl -sL "https://github.com/stackrox/kube-linter/releases/latest/download/kube-linter-linux.tar.gz" | tar -xz \
    && chmod +x kube-linter \
    && mv ./kube-linter /usr/local/bin/kube-linter \
    && curl -sL "https://github.com/controlplaneio/kubesec/releases/latest/download/kubesec_linux_amd64.tar.gz" | tar -xz \
    && chmod +x kubesec \
    && mv ./kubesec /usr/local/bin/kubesec \
    && KUBE_SCORE_VERSION=$(curl -sL "https://github.com/zegl/kube-score/tags" | grep -Eo "^\\s+v[0-9]{1,2}\.[0-9]{1,2}\.[0-9]{1,2}$" | sed 's/ //g' | sort -rV | uniq | head -1) \
    && KUBE_SCORE_VER_LIGHT=$(echo "${KUBE_SCORE_VERSION}" | sed 's/v//g') \
    && curl -sL "https://github.com/zegl/kube-score/releases/download/${KUBE_SCORE_VERSION}/kube-score_${KUBE_SCORE_VER_LIGHT}_linux_amd64.tar.gz" | tar -xz \
    && chmod +x kube-score \
    && mv ./kube-score /usr/local/bin/kube-score \
    && npm install -g github-files-fetcher \
    && fetcher --url="https://github.com/aquasecurity/trivy/tree/main/contrib" --out=/mnt \
    && curl -sSfLO "https://storage.googleapis.com/container-structure-test/latest/container-structure-test-linux-amd64" \
    && chmod +x container-structure-test-linux-amd64 \
    && mv container-structure-test-linux-amd64 /usr/local/bin/container-structure-test \
    && curl -sSfL "https://github.com/aelsabbahy/goss/releases/latest/download/goss-linux-amd64" -o /usr/local/bin/goss \
    && chmod +rx /usr/local/bin/goss \
    && curl -sSfL "https://raw.githubusercontent.com/aelsabbahy/goss/master/extras/dgoss/dgoss" -o /usr/local/bin/dgoss \
    && chmod +rx /usr/local/bin/dgoss \
    && DIVE_VERSION=$(curl -sSL "https://github.com/wagoodman/dive/releases/latest" | grep -oE 'v[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq | tail -1) \
    && DIVE_VERSION_LIGHT=$(echo "${DIVE_VERSION}" | sed 's/v//g') \
    && curl -sSfL "https://github.com/wagoodman/dive/releases/download/${DIVE_VERSION}/dive_${DIVE_VERSION_LIGHT}_linux_amd64.tar.gz" | tar -xz \
    && chmod +x dive \
    && mv ./dive /usr/local/bin/dive \
    && curl -sSfL "https://get.datree.io" | /bin/bash \
    && curl -sSfLO "https://gitlab.com/gitlab-org/release-cli/-/releases/${RELEASE_CLI}/downloads/bin/release-cli-linux-amd64" \
    && chmod +x ./release-cli-linux-amd64 \
    && mv ./release-cli-linux-amd64 /usr/local/bin/release-cli \
    && curl -sSfL -o /usr/local/bin/semver "https://raw.githubusercontent.com/fsaintjacques/semver-tool/master/src/semver" \
    && chmod +x /usr/local/bin/semver

FROM alpine:3.16

LABEL maintainer="Frederic Spiers <fredspiers@gmail.com>" \
      component="CI/CD tools"

ARG BUILD_VERSION
    
ENV TZ="Europe/Paris" \
    PATH="/kaniko:$PATH" \
    IMG_VERSION="$BUILD_VERSION"

WORKDIR /usr/app
    
COPY --from=ixxel/musl-dns-hack-alpine /lib/ld-musl-x86_64.so.1 /lib/ld-musl-x86_64.so.1

COPY --from=gcr.io/kaniko-project/executor:latest /kaniko/executor /usr/local/bin/executor

COPY --from=builder /usr/local/bin/helm \
                    /usr/local/bin/kube-score \
                    /usr/local/bin/kubesec \
                    /usr/local/bin/hadolint \
                    /usr/local/bin/kube-linter \
                    /usr/local/bin/semver \
                    /usr/local/bin/container-structure-test \
                    /usr/local/bin/goss \
                    /usr/local/bin/dgoss \
                    /usr/local/bin/dive \
                    /usr/local/bin/datree \
                    /usr/local/bin/gitlab-helper \
                    /usr/local/bin/release-cli \
                    /usr/local/bin/oc \
                    /usr/local/bin/argo \
                    /usr/local/bin/kubectl-argo-rollouts \
                    /usr/local/bin/grype \
                    /usr/local/bin/syft \
                    /usr/local/bin/jf \
                    /usr/local/bin/envconsul \
                    /usr/local/bin/vault \
                    /usr/local/bin/argocd \
                    /usr/local/bin/kubectl \
                    /usr/local/bin/
                    
COPY --from=builder /mnt/contrib ./contrib

RUN apk update --no-cache \
    && apk upgrade --no-cache \
    && apk add --no-cache --update \
    libc6-compat \
    podman \
    skopeo \
    buildah \
    img \
    git \
    wget \
    curl \
    gettext \
    bash \
    jq \
    python3 \
    py3-pip \
    bind-tools \
    util-linux \
    shellcheck \
    black \
    yamllint \
    nodejs \
    npm \
    fish \
    fzf \
    openssl \
    tzdata \
    && cp /usr/share/zoneinfo/${TZ} /etc/localtime \
    && rm -f /var/lib/containers/storage/libpod/bolt_state.db \
    && sed -i "s/driver = \"overlay\"/driver = \"vfs\"/" /etc/containers/storage.conf \
    && apk add trivy --no-cache --update --repository="http://dl-cdn.alpinelinux.org/alpine/edge/testing" \
    && apk add github-cli terraform yq --no-cache --update --repository="http://dl-cdn.alpinelinux.org/alpine/edge/community" \
    && ln -s /usr/bin/python3 /usr/bin/python \
    && helm plugin install "https://github.com/chartmuseum/helm-push" \
    && helm plugin install "https://github.com/databus23/helm-diff" \
    && helm plugin install "https://github.com/datreeio/helm-datree" \
    && curl -sSfLO "https://raw.githubusercontent.com/oh-my-fish/oh-my-fish/master/bin/install" \
    && fish install --noninteractive \
    && rm install \
    && fish -c "omf install fisk" \
    && fish -c "curl -sL https://git.io/fisher | source && fisher install jorgebucaran/fisher" \
    && fish -c "fisher install jethrokuan/fzf" \
    && fish -c "fisher install evanlucas/fish-kubectl-completions" \
    && { \
    echo "#!/usr/bin/fish"; \
    echo "set -x; set temp_dir (mktemp -d); cd $temp_dir"; \
    echo "curl -fsSLO https://github.com/kubernetes-sigs/krew/releases/latest/download/krew-linux_amd64.tar.gz"; \
    echo "tar zxvf krew-linux_amd64.tar.gz"; \
    echo "./krew-linux_amd64 install krew"; \
    echo "set -e temp_dir"; \
    } > install-krew.sh \
    && fish -c "chmod +x ./install-krew.sh" \
    && fish -c "./install-krew.sh" \
    && fish -c "echo 'set -gx PATH $PATH $HOME/.krew/bin' >> ~/.config/fish/config.fish" \
    && fish -c "kubectl krew install ctx" \
    && fish -c "kubectl krew install ns" \
    && fish -c "kubectl krew install neat" \
    && fish -c "kubectl krew install datree" \
    && fish -c "kubectl krew install whoami" \
    && { \
    echo "function k --wraps kubectl"; \
    echo '  kubectl $argv'; \
    echo "end"; \
    } > /root/.config/fish/functions/k.fish

RUN pip3 install --no-cache-dir flake8 \
                                flake8-json \
                                pytest \
                                pytest-cov \
                                coverage \
                                bandit \
                                yamlfix \
                                awxkit \
                                ansible-tower-cli \
                                python-gitlab
    
RUN apk add --virtual=build --no-cache --update gcc musl-dev python3-dev libffi-dev \
    && pip3 install --no-cache-dir cpython wheel \
    && pip3 install --no-cache-dir checkov \
    && pip3 uninstall --yes cpython wheel \
    && apk del --purge build

RUN npm install -g install-peerdeps \
                   @prantlf/jsonlint \
                   jslint \
                   eslint \
                   jshint \
                   snyk@latest

ENTRYPOINT ["tail", "-f", "/dev/null"]
