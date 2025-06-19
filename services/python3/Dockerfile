# 使用官方 Python 镜像作为基础
FROM python:3.11

# 安装 Node.js（使用 Nodesource 官方推荐的方法）
RUN curl -fsSL https://deb.nodesource.com/setup_16.x | bash - \
    && apt-get install -y nodejs \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# 设置工作目录
WORKDIR /www

# 确保 Node.js 和 Python 版本正确
RUN node -v && npm -v && python3 --version && pip3 --version

# 容器启动时进入 /app 目录
CMD ["sleep", "infinity"]
