---
title: Claude code MCP - VoiceMode
abbrlink: 541f03d0
date: 2025-12-02 17:41:53
updated: 2025-12-02 17:41:53
tags:
  - AI
  - claude code
categories:
  - AI
copyright:
---

# VoiceMode Claude Code 快速入门指南

## 📖 项目简介

VoiceMode 是一个通过 Model Context Protocol (MCP) 为 Claude Code 提供自然语音对话功能的工具。它让你可以用语音与 Claude Code 进行交互，实现真正的对话式编程体验。

### ✨ 核心特性

- 🎙️ **自然语音对话** - 用语音提问并听取 Claude Code 的回答
- 🗣️ **支持本地语音模型** - 兼容任何 OpenAI API 格式的 STT/TTS 服务
- ⚡ **实时交互** - 低延迟语音交互，自动选择最佳传输方式
- 🔧 **MCP 集成** - 与 Claude Code 无缝集成
- 🎯 **自动静音检测** - 停止说话时自动停止录音
- 🔄 **多种传输方式** - 支持本地麦克风或 LiveKit 房间通信

---

## 🚀 快速开始

### 系统要求

- **Python**: 3.10 或更高版本
- **操作系统**: Linux / macOS / Windows (WSL) / NixOS
- **硬件**: 带麦克风和扬声器的计算机
- **API密钥**: OpenAI API Key（推荐，可作为本地服务的备选）

---

## 📦 安装步骤

### 1. 安装 UV 包管理器

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### 2. 安装系统依赖

根据你的操作系统选择对应的安装命令：

#### Ubuntu/Debian (包括 WSL2)
```bash
sudo apt update
sudo apt install -y ffmpeg gcc libasound2-dev libasound2-plugins \
  libportaudio2 portaudio19-dev pulseaudio pulseaudio-utils python3-dev
```

#### Fedora/RHEL
```bash
sudo dnf install alsa-lib-devel ffmpeg gcc portaudio portaudio-devel python3-devel
```

#### macOS
```bash
# 安装 Homebrew（如果尚未安装）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 安装依赖
brew install ffmpeg node portaudio
```

#### NixOS
```bash
# 使用开发环境（临时）
nix develop github:mbailey/voicemode

# 或安装到系统
nix profile install github:mbailey/voicemode
```

### 3. 安装 VoiceMode

```bash
uvx voice-mode-install
```

### 4. 配置 OpenAI API Key（推荐）

```bash
export OPENAI_API_KEY=your-openai-api-key
```

> 💡 **提示**: 虽然可以使用本地语音服务，但建议配置 OpenAI API Key 作为备选方案。

---

## 🔧 集成到 Claude Code

### 基础集成

```bash
claude mcp add --scope user voicemode -- uvx --refresh voice-mode
```

### 带环境变量的集成

```bash
claude mcp add --scope user \
  --env OPENAI_API_KEY=your-openai-key \
  voicemode -- uvx --refresh voice-mode
```

---

## 🎤 开始使用

### 启动语音对话

```bash
claude converse
```

执行后，VoiceMode 会：
1. 自动启动录音
2. 检测你何时停止说话
3. 将语音转换为文本
4. 发送给 Claude Code 处理
5. 将 Claude 的回复转换为语音播放

### `converse` 函数特点

- **自动等待**: 默认会等待你的响应，创建自然的对话流
- **无需手动控制**: 不需要按键启动/停止录音
- **持续对话**: 支持多轮对话交互

---

## 🔐 本地语音服务（隐私优先）

如果你注重隐私或需要离线使用，可以配置本地语音服务：

### Whisper.cpp (本地语音识别)
```bash
# 详见文档
# docs/guides/whisper-setup.md
```

### Kokoro (本地语音合成)
```bash
# 详见文档
# docs/guides/kokoro-setup.md
```

这些本地服务提供与 OpenAI 相同的 API 接口，可以无缝切换。

---

## 🛠️ 常见配置

### 保存所有音频文件

```bash
export VOICEMODE_SAVE_AUDIO=true
```

音频文件会保存到: `~/.voicemode/audio/YYYY/MM/`

### 从源代码安装（用于开发）

```bash
git clone https://github.com/mbailey/voicemode.git
cd voicemode
uv tool install -e .
```

---

## ❓ 常见问题

### 无法访问麦克风
- 检查终端/应用程序的系统权限
- WSL2 用户需要安装额外的音频包（pulseaudio, libasound2-plugins）

### UV 未找到
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# 重新加载 shell 配置
source ~/.bashrc  # 或 ~/.zshrc
```

### OpenAI API 错误
- 验证 `OPENAI_API_KEY` 是否正确设置
- 检查 API key 是否有效且有足够余额

### 没有音频输出
- 检查系统音频设置
- 确认输出设备正常工作
- 验证音量未静音

---

## 📚 进阶文档

- **[完整配置指南](https://github.com/mbailey/voicemode/blob/master/docs/guides/configuration.md)** - 所有环境变量参考
- **[Whisper.cpp 配置](https://github.com/mbailey/voicemode/blob/master/docs/guides/whisper-setup.md)** - 本地语音识别
- **[Kokoro 配置](https://github.com/mbailey/voicemode/blob/master/docs/guides/kokoro-setup.md)** - 本地语音合成
- **[LiveKit 集成](https://github.com/mbailey/voicemode/blob/master/docs/guides/livekit-setup.md)** - 实时语音通信
- **[开发环境配置](https://github.com/mbailey/voicemode/blob/master/docs/tutorials/development-setup.md)** - 本地开发指南

---

## 🔗 相关资源

- **官网**: [getvoicemode.com](https://getvoicemode.com)
- **完整文档**: [voice-mode.readthedocs.io](https://voice-mode.readthedocs.io)
- **GitHub**: [github.com/mbailey/voicemode](https://github.com/mbailey/voicemode)
- **PyPI**: [pypi.org/project/voice-mode](https://pypi.org/project/voice-mode/)
- **Twitter/X**: [@getvoicemode](https://twitter.com/getvoicemode)
- **YouTube**: [@getvoicemode](https://youtube.com/@getvoicemode)

---

## 📝 许可证

MIT License - A [Failmode](https://failmode.com) Project

---

## 🎯 下一步

1. ✅ 完成基础安装
2. ✅ 配置 API Key
3. ✅ 启动第一次语音对话
4. 🔧 根据需要配置本地语音服务
5. 📖 探索进阶功能和配置选项

**祝你使用愉快！开始用语音与 Claude Code 对话吧！** 🎉