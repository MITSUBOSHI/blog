---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
categories: ["ja"]
tags: []
slug: "{{ .Name | slug }}"
draft: true
---
