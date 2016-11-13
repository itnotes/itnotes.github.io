---
layout: post
title:  Kottler 01. Построение каркаса.
date:   2016-08-24 15:00:00
categories: posts
description: Построение каркаса приложения spring + kotlin
tags:
- Spring
- Kotlin
- Tutorial
- Kottler
keywords:
- Spring
- Kotlin
- Tutorial
- Kottler
---

Первая часть цикла статей посвящена созданию каркаса будущего приложения и 
настройке логирования.

<!--more-->

# Создание скелета приложения

Для быстрого старта разработки spring приложения существует сайт специальный 
сервис - [SPRING INITIALIZR](https://start.spring.io/). Там необходимо выбрать 
сборщик (maven/gradle),  указать версию Spring Boot, информацию о проекте 
(Group и Artifact),  а так же выбрать список начальных зависимостей 
(для старта нам хватит Web зависимости). По дефолту данный INITIALIZR 
подразумевает, что мы будем разрабатывать на java. Чтобы выбрать язык нужно 
нажать на кнопку "Switch to the full version" внизу страницы.
<img style="width: 100%" src="/public/images/2016-08-24-kottler-build-skeleton/sk-01.png">

После нажатия на кнопку "Generate Project" скачивается архив со следующим содержимым:

* .mvn
* src
* mvnw
* mvnw.cmd
* pom.xml

Из всего этого нам необходимы только папка src и файл pom.xml

Импортируем проект в нашу любимую IDE (в моём случае это IntelliJ IDEA). Если 
установлен плагин [ignore](https://github.com/hsz/idea-gitignore), то идея 
любезно предложит создать файл .gitignore (или же его можно создать самостоятельно).

После импорта мы получили следующую структуру:
<img style="width: 100%" src="/public/images/2016-08-24-kottler-build-skeleton/sk-02.png">

Сразу добавим в pom.xml необходимые для логирования зависимости:

{% gist bc208cae4cf43c4c297c08cfc8936400 pom.xml %}

# Тестовый контроллер

Теперь напишем небольшой тестовый контроллер чтобы убедиться, что Spring MVC работает:

{% gist bc208cae4cf43c4c297c08cfc8936400 HelloController.kt %}

После этого можно запустить приложение и убедиться, что при переходе на URL 
['http://127.0.0.1:8080/hello'](http://127.0.0.1:8080/hello) отображается 
сообщение 'Hello'. Для запуска необходимо выполнить задачу ```mvn spring-boot:run```
(можно вызвать её прямо из IDEA):
<img style="width: 100%" src="/public/images/2016-08-24-kottler-build-skeleton/sk-03.png">

Несмотря на то что контроллер отрабатывает корректно, debug сообщение из строки 
```logger.debug("HelloController#hello executed")``` никуда не вывелось. Причина 
в том, что не добавлен файл конфигурации логирования.

# Настройка логирования

Вывод логов организуем в консоль и файл. Максимальный размер файла будет 5Мб, 
а храниться будет 20 последних файлов.

Создадим файл 'src/main/resources/log4j2.xml':

{% gist bc208cae4cf43c4c297c08cfc8936400 log4j2.xml %}

В данной XML описаны два appender'а:

* Console - отвечает за вывод логов на консоль
* Full - отвечает за вывод логов в файл
  * fileName - имя файла лога.
  * filePattern - паттерн для сохранения предыдущих файлов. Когда размер 
  текущего файла станет больше 5 мегабайт логгер автоматически переместит 
  текущий файл в путь указанный в данной переменной и продолжит писать логи в 
  новый файл.
  * PatternLayout - указываем паттерн вывода логов
  * Policies - содержит набор правил по обновлению текущего файла логирования. 
  У нас указано, что надо обнулять файл по достижении им размера 5 МБ.
  * DefaultRolloverStrategy - указывает сколько предыдущих файлов необходимо хранить

Перезапустим приложение, перейдём на URL ['http://127.0.0.1:8080/hello'](http://127.0.0.1:8080/hello)
и увидим уже знакомую надпись 'Hello', а так же увидим запись в консоли: 

```2016-08-24 07:59:47.522 [http-nio-8080-exec-1] DEBUG io.github.itnotes.controllers.HelloController[14] - HelloController#hello executed```

# Summary

В данном уроке мы создали каркас приложения и настроили логирование.

Исходный код можно посмотреть на [github](https://github.com/itnotes/spring-kottler/tree/sk-01).

Или скачать, выполнив команды:

{% highlight bash %}
git clone https://github.com/itnotes/spring-kottler.git
git checkout tags/sk-01
{% endhighlight %}