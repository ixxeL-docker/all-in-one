FROM gcr.io/kaniko-project/executor:latest as kaniko

FROM alpine:latest AS builder

RUN apk update --no-cache \
    && apk add curl --no-cache --update \
    && curl -sLO "https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64" \
    && chmod +x ./argocd-linux-amd64 \
    && mv ./argocd-linux-amd64 /usr/local/bin/argocd \
    && curl -sLO "https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64" \
    && chmod +x ./kubectl-argo-rollouts-linux-amd64 \
    && mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts \
    && curl -sLO "https://github.com/argoproj/argo-workflows/releases/latest/download/argo-linux-amd64.gz" \
    && gunzip argo-linux-amd64.gz \
    && chmod +x argo-linux-amd64 \
    && mv ./argo-linux-amd64 /usr/local/bin/argo \
    && HELM_VERSION=$(curl -sSL "https://github.com/kubernetes/helm/releases/latest" | grep -oE 'v[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq | tail -1) \
    && curl -sSL "https://get.helm.sh/helm-${HELM_VERSION}-linux-amd64.tar.gz" | tar xzv \
    && mv linux-amd64/helm /usr/local/bin/helm \
    && curl -sSL "https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/linux/oc.tar.gz" -o /tmp/oc.tar.gz \
    && tar -xzvf /tmp/oc.tar.gz -C /usr/local/bin \
    && rm /tmp/oc.tar.gz \
    && KUBECTL_VERSION=$(curl -s "https://storage.googleapis.com/kubernetes-release/release/stable.txt") \
    && curl -sSLO "https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" \
    && chmod +x ./kubectl \
    && mv ./kubectl /usr/local/bin/kubectl \
    && VAULT_VERSION=$(curl -sSL "https://releases.hashicorp.com/vault" | grep -oE '/[0-9]+\.[0-9]+\.[0-9]+/' | head -1 | sed 's/\///g') \
    && curl -sLO "https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_amd64.zip" \
    && unzip "./vault_${VAULT_VERSION}_linux_amd64.zip" -d /usr/local/bin \
    && ENVCONSUL_VERSION=$(curl -sSL "https://releases.hashicorp.com/envconsul" | grep -oE '/[0-9]+\.[0-9]+\.[0-9]+/' | head -1 | sed 's/\///g') \
    && curl -sLO "https://releases.hashicorp.com/envconsul/${ENVCONSUL_VERSION}/envconsul_${ENVCONSUL_VERSION}_linux_amd64.zip" \
    && unzip "./envconsul_${ENVCONSUL_VERSION}_linux_amd64.zip" -d /usr/local/bin \
    && curl -fL "https://getcli.jfrog.io" | sh \
    && mv jfrog /usr/local/bin \
    && curl -sSfL "https://raw.githubusercontent.com/anchore/grype/main/install.sh" | sh -s -- -b /usr/local/bin \
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
    && mv ./kube-score /usr/local/bin/kube-score

FROM alpine:latest

LABEL maintainer="Frederic Spiers <fredspiers@gmail.com>" \
      component="CI/CD tools"

ENV TZ="Europe/Paris" \
    IMG_VERSION="2.0.0"

COPY --from=kaniko /kaniko/executor /usr/local/bin/executor
COPY --from=builder /usr/local/bin/oc \
                    /usr/local/bin/kube-score \
                    /usr/local/bin/kubesec \
                    /usr/local/bin/hadolint \
                    /usr/local/bin/kube-linter \
                    /usr/local/bin/argo \
                    /usr/local/bin/kubectl-argo-rollouts \
                    /usr/local/bin/grype \
                    /usr/local/bin/jfrog \
                    /usr/local/bin/envconsul \
                    /usr/local/bin/vault \
                    /usr/local/bin/argocd \
                    /usr/local/bin/helm \
                    /usr/local/bin/kubectl \
                    /usr/local/bin/

RUN apk update --no-cache \
    && apk upgrade --no-cache \
    && apk add --no-cache --update \
    libc6-compat \
    podman \
    skopeo \
    buildah \
    git \
    wget \
    curl \
    gettext \
    bash \
    jq \
    yq \
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
    && apk add trivy --no-cache --update --repository=http://dl-cdn.alpinelinux.org/alpine/edge/testing \
    && ln -s /usr/bin/python3 /usr/bin/python \
    && curl -sLO "https://raw.githubusercontent.com/oh-my-fish/oh-my-fish/master/bin/install" \
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
    && { \
    echo "function k --wraps kubectl"; \
    echo '  kubectl $argv'; \
    echo "end"; \
    } > /root/.config/fish/functions/k.fish

#RUN pip3 install --upgrade pip \
RUN pip3 install --no-cache-dir flake8 \
                                   flake8-json \
                                   pytest \
                                   pytest-cov \
                                   coverage \
                                   bandit \
                                   yamlfix \
                                   awxkit \
                                   ansible-tower-cli
    
RUN apk add --no-cache --update gcc musl-dev python3-dev libffi-dev \
    && pip3 install --no-cache-dir cpython wheel \
    && pip3 install --no-cache-dir checkov \
    && pip3 uninstall --yes cpython wheel \
    && apk del gcc musl-dev python3-dev libffi-dev

RUN npm install -g install-peerdeps \
                   @prantlf/jsonlint \
                   jslint \
                   eslint \
                   jshint \
                   snyk@latest

ENTRYPOINT ["tail", "-f", "/dev/null"]
