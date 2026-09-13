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

RUN ln -sf /usr/bin/python3.12 /usr/bin/python3 && \
    ln -sf /usr/bin/python3.12 /usr/bin/python

# 強制 Rust build 腳本使用 Python 3.12
ENV PYTHON=/usr/bin/python3.12
ENV PYO3_PYTHON=/usr/bin/python3.12


ENV PYTHONNOUSERSITE=1
ENV PYTHONPATH=""

RUN git clone -b v0.5.19 https://github.com/sgl-project/sglang .
# 補複製 README 與 LICENSE 至 python 目錄，避免 setuptools 找不到文件的警告/錯誤
RUN cp README.md LICENSE python/ 2>/dev/null || true

RUN python3.12 -m pip install --upgrade pip setuptools wheel && \
    python3.12 -m pip install -e "python"


WORKDIR /
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
          "name": "Qwen3 Coder Next (64K Direct)",
          "options": {
            "temperature": 0
          }
        }
      }
    }
  }
}
EOF

RUN mkdir -p /root/.config/opencode/agents
RUN mkdir -p /root/.claude/skills
RUN git clone https://github.com/Weizhena/Deep-Research-skills
RUN cd /Deep-Research-skills && cp agents/web-search-opencode.md /root/.config/opencode/agents/web-search.md
RUN cd /Deep-Research-skills && cp -r agents/web-search-modules /root/.config/opencode/agents/
RUN cd /Deep-Research-skills && cp -r skills/research-en/* /root/.claude/skills/

# vscode server
RUN mkdir -p /root/.vscode-server/bin/${VSCODE_COMMIT_ID}
RUN curl -L https://update.code.visualstudio.com/commit:${VSCODE_COMMIT_ID}/server-linux-x64/stable \
    | tar --strip-components=1 -xz -C /root/.vscode-server/bin/${VSCODE_COMMIT_ID}

CMD ["/usr/sbin/sshd", "-D"]
