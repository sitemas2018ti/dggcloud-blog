---
title: "{{ replace .File.ContentBaseName "-" " " | title }}"
slug: "{{ .File.ContentBaseName }}"
date: {{ .Date }}
draft: true
description: ""
clouds: []   # aws, azure, gcp
tags: []
---
