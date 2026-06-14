FROM nvidia/cuda:13.0.1-cudnn-devel-rockylinux9

ARG HF_TOKEN
ARG VSCODE_COMMIT_ID

#vscode server
RUN yum install -y procps-ng tar gzip curl --allowerasing
RUN yum install -y gcc gcc-c++ make cmake
RUN yum install -y ncurses git
RUN yum install -y python3.12 python3.12-pip python3.12-devel
RUN yum install -y libnccl libnccl-devel libnccl-static
RUN yum install -y epel-release
RUN yum install -y ccache
#sglang
RUN dnf config-manager --enable crb
RUN yum install -y protobuf-compiler which curl numactl-libs
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
ENV PATH="/root/.cargo/bin:${PATH}"
RUN rustc --version && cargo --version

#vscode server
RUN yum install -y libnccl libnccl-devel libnccl-static


RUN dnf install -y openssh-server passwd && dnf clean all

RUN echo "root:root123" | chpasswd
RUN mkdir /var/run/sshd && ssh-keygen -A
RUN sed -i 's/^#PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config \
    && sed -i 's/^#PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config \
    && sed -i 's/^#PermitUserEnvironment no/PermitUserEnvironment yes/' /etc/ssh/sshd_config

RUN echo 'NVIDIA_VISIBLE_DEVICES=all' >> /etc/environment



# Claude
# 安裝 Claude CLI
RUN curl -fsSL https://claude.ai/install.sh | bash

# 建立設定目錄與設定檔
RUN mkdir -p ~/.claude && \
    echo '{"env": {"CLAUDE_CODE_ATTRIBUTION_HEADER": "0", "ANTHROPIC_BASE_URL": "http://10.1.1.60:11434", "ANTHROPIC_AUTH_TOKEN": "dummy-token", "ANTHROPIC_MODEL": "my-local-model"}, "skipAutoPermissionPrompt": true}' > ~/.claude/settings.json

CMD ["/usr/sbin/sshd", "-D"]
