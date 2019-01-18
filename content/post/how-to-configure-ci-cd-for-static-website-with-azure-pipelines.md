---
title: "Azure Pipelinesで静的サイトのCI/CDを実現する手順"
date: 2019-01-15T19:30:32+09:00
draft: true
categories:
  - Azure
tags:
  - Azure DevOps
  - Azure Blob Storage
---

[Azure Pipelines](https://azure.microsoft.com/ja-jp/services/devops/pipelines/)を使って、GitHubリポジトリで管理している静的サイトのCI/CDを実現する手順をまとめます。

<!--more-->

[前回の手順]({{< ref "how-to-host-hugo-on-azure-blob-storage.md" >}})で、静的サイトのホスティング用のアカウント、コードが作成済みであること前提とします。

