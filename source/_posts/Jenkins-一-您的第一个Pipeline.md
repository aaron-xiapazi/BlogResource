---
title: Jenkins(一) 您的第一个Pipeline
date: 2020-03-28 23:51:12
tags:
categories:
---

Jenkins Pipeline（或简称为 "Pipeline"）是一套插件，将持续交付的实现和实施集成到 Jenkins 中。

持续交付 Pipeline 自动化的表达了这样一种流程：将基于版本控制管理的软件持续的交付到您的用户和消费者手中。

Jenkins Pipeline 提供了一套可扩展的工具，用于将“简单到复杂”的交付流程实现为“持续交付即代码”。Jenkins Pipeline 的定义通常被写入到一个文本文件（称为 `Jenkinsfile` ）中，该文件可以被放入项目的源代码控制库中

- 将以下代码复制到您的仓库中并命名为JenkinsfileJava

  Jenkinsfile (Declarative Pipeline)

  ```Jenkins
  pipeline {
      agent { docker 'maven:3.3.3' }
      stages {
          stage('build') {
              steps {
                  sh 'mvn --version'
              }
          }
      }
  }
  ```

- 单机Jenkins的New Item菜单
- 为新工程起一个名字，并选择 **Multibranch Pipeline**
- 单击 **Add Source** 按钮，选择您想要使用的仓库类型并填写详细信息.
- 单击 **Save** 按钮，观察您的第一个Pipeline运行！

您可能需要修改 `Jenkinsfile` 以便应用在您自己的项目中。尝试修改 `sh` 命令，使其与您本地运行的命令相同。

在配置好 Pipeline 之后，Jenkins 会自动检测您仓库中创建的任何新的分支或合并请求， 并开始为它们运行 Pipelines。