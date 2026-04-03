---
version: 1
default_provider: openai
default_quality: 2k
default_aspect_ratio: "16:9"
default_image_size: null
default_model:
  google: null
  openai: "gpt-image-1"
  azure: null
  openrouter: null
  dashscope: null
  minimax: null
  replicate: null
batch:
  max_workers: 6
  provider_limits:
    openai:
      concurrency: 2
      start_interval_ms: 1200
---
