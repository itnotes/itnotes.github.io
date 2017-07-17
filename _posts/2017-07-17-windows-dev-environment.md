---
layout: post
title:  Настройка dev окружения под Windows
date:   2017-07-17 23:50:00
categories: posts
description:
tags:
- soft
- environment
keywords:
- windows
- soft
- quick start
---

Сегодня в очередной раз пришлось настраивать рабочее окружение с нуля, в связи с чем появилось большое желание набросать список телодвижений чтобы в следующий раз (а я боюсь, что следующий раз может наступить в любую секунду) было не так больно вспоминать что откуда берётся и куда ставится.

<!--more-->

## Чек-лист софта

- Google Chrome
- IntellIJ IDEA - ставится через [Toolbox](https://www.jetbrains.com/toolbox/app/)
- Putty
- [Git](https://git-scm.com/)
- WinMerge
- Notepad++/Atom/VSCode
- Docker (toolbox)
- PGAdmin 3/4
- Skype
- Slack


## Докручиваем конфиги

### Git + Pageant
Чтобы можно было использовать ключи из Pageant'a в GitBash нужно добавить в environments переменную 'GIT_SSH' с путём до файла plink.exe (он находится в той же папке что и PuTTY):
```
GIT_SSH=C:\Program Files (x86)\PuTTY\plink.exe
```

### IntellIJ IDEA + Pageant
Чтобы IDEA так же могла использовать ключи из Pageant'a нужно выбрать 'Native' в меню Version Control -> Git -> SSH executable.
