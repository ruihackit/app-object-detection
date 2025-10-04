# Intel(x86_64)版のイメージを強制的に使用
FROM --platform=linux/amd64 ubuntu:22.04

# --- ↓↓↓ ここから修正：会社の証明書を信頼させる設定 ↓↓↓ ---
# 依存関係のインストール
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y \
    ca-certificates \
    curl \
    git \
    unzip \
    xz-utils \
    zip \
    jq \
    libglu1-mesa \
    clang \
    cmake \
    ninja-build \
    pkg-config \
    libgtk-3-dev \
    liblzma-dev \
    && rm -rf /var/lib/apt/lists/*

# asdfをインストールするユーザーを作成
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m -s /bin/bash $USERNAME \
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# asdfのインストールと設定
USER $USERNAME
WORKDIR /home/$USERNAME

RUN git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch v0.14.0

# asdfのパスとASDF_DIR環境変数を.bashrcに追加
RUN /bin/bash -c "echo -e '\nexport ASDF_DIR=\"\$HOME/.asdf\"' >> ~/.bashrc && \
    echo '. \"\$ASDF_DIR/asdf.sh\"' >> ~/.bashrc && \
    echo '. \"\$ASDF_DIR/completions/asdf.bash\"' >> ~/.bashrc"

# asdfプラグインの追加とツールのインストール
COPY .tool-versions .
RUN export ASDF_DIR="$HOME/.asdf" && \
    . "$ASDF_DIR/asdf.sh" && \
    asdf plugin add flutter && \
    asdf plugin add java && \
    asdf install

# Android SDKのインストール
ENV ANDROID_SDK_ROOT="/home/$USERNAME/android_sdk"
ENV PATH="$PATH:${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin:${ANDROID_SDK_ROOT}/platform-tools"

USER root
RUN mkdir -p ${ANDROID_SDK_ROOT}/cmdline-tools && \
    curl -L -o /tmp/sdk.zip "https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip" && \
    unzip /tmp/sdk.zip -d ${ANDROID_SDK_ROOT}/cmdline-tools && \
    mv ${ANDROID_SDK_ROOT}/cmdline-tools/cmdline-tools ${ANDROID_SDK_ROOT}/cmdline-tools/latest && \
    rm /tmp/sdk.zip && \
    chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/.android 2>/dev/null || true && \
    chown -R ${USERNAME}:${USERNAME} ${ANDROID_SDK_ROOT}
USER $USERNAME

# SDK PlatformとPlatform Toolsをインストール
RUN export ASDF_DIR="$HOME/.asdf" && \
    . "$ASDF_DIR/asdf.sh" && \
    yes | sdkmanager --licenses && \
    sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"

# Flutter doctorの事前実行（キャッシュ作成など）
RUN export ASDF_DIR="$HOME/.asdf" && \
    . "$ASDF_DIR/asdf.sh" && \
    flutter --version && \
    flutter config --no-analytics && \
    flutter precache && \
    flutter doctor
