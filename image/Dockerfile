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


WORKDIR /app

RUN git clone https://github.com/sgl-project/sglang .
RUN pip3.12 install -e "python"

# install opencode
RUN curl -fsSL https://opencode.ai/install | bash
RUN mkdir -p /root/.config/opencode/
RUN cat <<'EOF' > /root/.config/opencode/opencode.json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "sglang-direct": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "SGLang Direct Engine",
      "options": {
        "baseURL": "http://10.1.1.60:11434/v1",
        "apiKey": "not-needed-for-sglang"
      },
      "models": {
        "my-local-model": {
          "name": "Qwen3 Coder Next (64K Direct)"
        }
      }
    }
  }
}
EOF


CMD ["/usr/sbin/sshd", "-D"]
