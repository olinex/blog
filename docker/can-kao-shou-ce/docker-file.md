---
description: '本文章由olinex翻译, 转载请在页面开头标明出处'
---

# Docker File

Docker 可以通过读取 `Dockerfile` 的结构来自动构建镜像. 它是一个文本文件, 包含了用户创建一个镜像时所需要调用的所有命令. 通过 `docker build` 命令, 用户可以创建一个能自动运行一系列命令的任务.

这个教程描述了你可以在 Dockerfile 内使用的命令. 当你完成了这个教程的阅读之后, 请参考 [Dockerfile 最佳实践](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/) .



