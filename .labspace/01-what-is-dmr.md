# Introduction

## What is Docker Model Runner?

- Docker Model Runner (DMR) is a tool that manages, runs, and deploys AI models using Docker
- **Purpose**: Brings AI models into your familiar DevOps workflow - treat models like containers
- **Key Benefit**: No need for specialized ML infrastructure or complex Python environments

## Why Use Docker Model Runner?

- **Familiar tooling**: Use `docker` CLI commands you already know
- **Reproducible**: Package models as OCI artifacts for consistent deployment
- **Local-first**: Run models on your laptop without cloud dependencies
- **OpenAI-compatible**: Drop-in replacement for OpenAI API calls
- **Resource-efficient**: Models load on-demand and unload when idle

## Platform Support

- **macOS**: Apple Silicon
- **Windows**: NVIDIA GPUs (drivers 576.57+), Qualcomm Adreno GPUs, Vulkan enabled (AMD / Intel) GPUs (Since 2025 Oct)
- **Linux**: CPU and NVIDIA GPU support (drivers 575.57.08+)
