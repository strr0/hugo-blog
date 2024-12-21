---
title: "GitLab CI/CD 自动化部署"
date: 2023-10-25T10:00:00+08:00
categories: ["operations"]
tags: ["linux"]
draft: false
---

## 介绍

GitLab CI/CD 是 GitLab 提供持续集成服务，主要功能通过一个执行器和作业配置实现，执行器一般不部署在 Gitlab 服务器下，推荐部署到开发环境服务器。将 .gitlab-ci.yml 文件添加到存储库的根目录，并 GitLab 项目配置为使用执行器，然后每次提交或推送，触发 CI 管道，执行自动化部署。

## 1 Gitlab Runner 部署

### 1.1 环境需要

- Docker 环境（docker 类型 runner 需要）

### 1.2 安装 Runner

#### 1.2.1 添加官方源

```bash
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | sudo bash
```

#### 1.2.2 安装最新版本

```bash
yum install gitlab-runner
```

#### 1.2.3 安装指定版本

通过 /api/v4/version 查看 Gitlab 版本

查看可用 runner 版本
```bash
yum list gitlab-runner --showduplicates | sort -r
```
安装
```bash
yum install gitlab-runner-15.11.0-1
```

### 1.3 注册 Runner

#### 1.3.1 交互模式

运行命令注册
```bash
gitlab-runner register
```
如需代理，运行如下命令注册
```bash
export HTTP_PROXY=http://yourproxyurl:3128
export HTTPS_PROXY=http://yourproxyurl:3128

sudo -E gitlab-runner register
```
输入 Gitlab URL，如 https://gitlab.com

输入验证 token

输入 Runner 名称

输入执行器类型，如 docker

#### 1.3.2 非交互模式

Shell 类型执行器（推荐）
```bash
gitlab-runner register \
    --non-interactive \
    --url "https://gitlab.com/" \
    --registration-token "$PROJECT_REGISTRATION_TOKEN" \
    --executor "shell" \
    --shell "bash" \
    --description "shell-executor-runner" \
    --tag-list "shell" \
    --run-untagged="true" \
    --locked="false"
```
Docker 类型执行器（GitLab 15.10 及以上）
```bash
gitlab-runner register \
    --non-interactive \
    --url "https://gitlab.com/" \
    --token "$RUNNER_TOKEN" \
    --executor "docker" \
    --docker-image alpine:latest \
    --description "docker-runner"
```
Docker 类型执行器（GitLab 15.10 以下）
```bash
gitlab-runner register \
    --non-interactive \
    --url "https://gitlab.com/" \
    --registration-token "$PROJECT_REGISTRATION_TOKEN" \
    --executor "docker" \
    --docker-image alpine:latest \
    --description "docker-runner" \
    --tag-list "docker,aws" \
    --run-untagged="true" \
    --locked="false"
```
注册成功后可在 Gitlab 项目的 Runner 页面看到可用的 runner

> 使用 docker 类型执行器，执行结果保存在 docker 容器内部，不可直接部署，需要上传到容器外部部署，一般只需要结果的情况下可以使用 docker 类型执行器。token 可以在 Project -> Settings -> CI / CD -> Runner 中找到

### 1.4 移除 Runner

```bash
gitlab-runner unregister --name shell-executor-runner
```

## 2 GitLab CI/CD 配置

### 2.1 环境变量设置

在 Settings -> CI /CD 页设置所需要的环境变量，如 APP_NAME（项目名称），APP_PORT（端口号），IMAGE_NAME（镜像名称），IMAGE_VERSION（镜像版本）

### 2.2 作业配置

在项目根目录创建 .gitlab-ci.yml
```yml
stages:
  - build
  - deploy

variables:
  MAVEN_CLI_OPTS: >-
    -DskipTests

build-job:
  stage: build
  script:
    - mvn $MAVEN_CLI_OPTS package
    - |
      if docker ps -a | grep -q "$APP_NAME"; then
        docker stop "$APP_NAME"
        docker rm "$APP_NAME"
      fi
    - |
      if docker images | grep -q "$IMAGE_NAME"; then
        docker rmi "$IMAGE_NAME:$IMAGE_VERSION"
      fi
    - docker build -t "$IMAGE_NAME:$IMAGE_VERSION" ruoyi-admin
  only:
    refs:
      - master
  except:
    changes:
      - xxx/*

deploy-job:
  stage: deploy
  script:
    - docker run -d --name "$APP_NAME" -p "$APP_PORT:8080" "$IMAGE_NAME:$IMAGE_VERSION"
  only:
    refs:
      - master
  except:
    changes:
      - xxx/*
```

### 2.3 自动化部署

当代码被 push 到 gitlab 时触发部署，可在 CI / CD 页看到执行结果

## 3 问题与修复

### 3.1 mvn: 未找到命令

mvn 路径权限问题，可以修改 mvn 权限为所有人可读
```bash
chmod -R 755 $M2_HOME
```

### 3.2 docker 权限问题

```bash
sudo gpasswd -a gitlab-runner docker
newgrp docker
```

### 3.3 413 Request Entity Too Large

当构建作业失败并出现以下错误时，增加[最大产物大小](https://docs.gitlab.com/ee/administration/settings/continuous_integration.html#maximum-artifacts-size)
```
Uploading artifacts as "archive" to coordinator... too large archive ...
```

### 3.4 artifacts 问题

在流水线执行每一个作业的时候会默认将上一个作业的产物清空，配置 artifacts 可以临时保存作业产物，以供下一个作业使用

> artifacts 问题的另一个解决思路是尽量不跨作业使用产物，后续作业如需使用上一个作业的产物时可以选择将两个作业合并，如 mvn 打包项目和 docker 打包镜像整合为一个作业