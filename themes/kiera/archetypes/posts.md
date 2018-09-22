+++
title = "{{ replace .TranslationBaseName "-" " " | title }}"
date = {{ .Date }}
draft = true
tags = []
categories = []
+++

![{{ replace .TranslationBaseName "-" " " | title }}](/images/titelbild.jpg)
{{< figure src="/images/titelbild.jpg" caption="{{ replace .TranslationBaseName "-" " " | title }}" >}}

<!--more-->
