---
layout: post
title:  "Создание zip архива с зависимостями при помощи maven"
date:   2015-08-19 00:56:00
categories: posts
description: "Создание zip архива с зависимостями при помощи maven"
tags:
- maven
- java
- maven assembly
- log4j
---

## Предыстория
Несколько лет я живу в мире JavaEE - основная работа + хобби. Опыт с языками кроме Java, есть, но не значительный. Однажды знакомый предложил поучаствовать в его проекте. Ему была нужно десктопная софтинка, которая интегрировала бы сервисы гугла с их CMS. Ограничений на технологии не было и я, не сильно заморачиваясь, выбрал Java (на то время была 7 версия). Но какой проект без экспериментов? В качестве эксперимента было решено попробовать использовать JavaFX в качестве UI. При разработке использовались стороние библиотеки, которые регулярно обновлялись и добавлялись, поэтому подготовка релизного zip архива была взвалена на плечи maven'a. Задача  подготовки архива решается не сложно. Единственным местом преткновения стал JavaFX который в версии Java не входил в состав JDK.

С тех пор утекло много времени. Проект готов и больше не дорабатывается, седьмой Java пришла на смену восьмая и уже доступен тестовый релиз девятой. Но вопрос "как подготовить zip архив со всеми зависимостями с использованием maven?" вполне актуален. Далее следует небольшая инструкция по подготовке проекта.

<!--more-->

## Подготавливаем проект
В качестве экспериментального проекта предлагаю написать консольную утилиту которая будет писать в лог файл время запуска. В качестве логгера будем использовать Apache log4j

Создадим maven проект. Для этого в консоли перейдём в папку, где будет находиться проект и пишем
<pre>
<code class="bash">
mvn archetype:generate -DgroupId=io.github.itnotes -DartifactId=zip-with-dependencies -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
</code>
</pre>. В результате получаем пустой проект со следующей структурой:

<pre>
  <code>
zip-with-dependencies
|-- pom.xml
`-- src
    |-- main
    |   `-- java
    |       `-- com
    |           `-- mycompany
    |               `-- app
    |                   `-- App.java
    `-- test
        `-- java
            `-- com
                `-- mycompany
                    `-- app
                        `-- AppTest.java
  </code>
</pre>

И pom.xml:
<pre>
  <code class="xml">
    &lt;project xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot; xmlns=&quot;http://maven.apache.org/POM/4.0.0&quot; xsi:schemaLocation=&quot;http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd&quot;&gt;&#10;      &lt;modelVersion&gt;4.0.0&lt;/modelVersion&gt;&#10;      &lt;groupId&gt;io.github.itnotes&lt;/groupId&gt;&#10;      &lt;artifactId&gt;zip-with-dependencies&lt;/artifactId&gt;&#10;      &lt;packaging&gt;jar&lt;/packaging&gt;&#10;      &lt;version&gt;1.0-SNAPSHOT&lt;/version&gt;&#10;      &lt;name&gt;zip-with-dependencies&lt;/name&gt;&#10;      &lt;url&gt;http://maven.apache.org&lt;/url&gt;&#10;      &lt;dependencies&gt;&#10;        &lt;dependency&gt;&#10;          &lt;groupId&gt;junit&lt;/groupId&gt;&#10;          &lt;artifactId&gt;junit&lt;/artifactId&gt;&#10;          &lt;version&gt;3.8.1&lt;/version&gt;&#10;          &lt;scope&gt;test&lt;/scope&gt;&#10;        &lt;/dependency&gt;&#10;      &lt;/dependencies&gt;&#10;    &lt;/project&gt;
  </code>
</pre>

Добавим зависимость на логгер
<pre>
  <code class="xml">
    &lt;dependency&gt;&#10;      &lt;groupId&gt;log4j&lt;/groupId&gt;&#10;      &lt;artifactId&gt;log4j&lt;/artifactId&gt;&#10;      &lt;version&gt;1.2.17&lt;/version&gt;&#10;    &lt;/dependency&gt;
  </code>
</pre>

и элементарная реализация основного класса
<pre>
  <code class="java">
package io.github.itnotes;
import org.apache.log4j.Logger;
import java.util.Date;
public class App {
    public static void main(String[] args) {
        try {
            Logger log = Logger.getLogger(App.class);
            Date date = new Date();
            System.out.println(date);
            log.debug(date);
        } catch (RuntimeException e) {
            e.printStackTrace();
        }
    }
}
  </code>
</pre>

Создадим папку `src/main/resources` и туда положим файл конфигурации для логгера log4j.properties:
<pre>
  <code class="text">
    log4j.rootLogger=DEBUG, file<p/>
    log4j.appender.file=org.apache.log4j.RollingFileAppender
    log4j.appender.file.File=application.log
    log4j.appender.file.MaxFileSize=5MB
    log4j.appender.file.MaxBackupIndex=10
    log4j.appender.file.layout=org.apache.log4j.PatternLayout
    log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
  </code>
</pre>

пробуем запустить код в IDE. В консоли видим следующее сообщение
<pre>
  <code class="text">
    Wed Aug 19 00:11:53 GST 2015
  </code>
</pre>

Так же появился файл application.log со следующим содержанием:
<pre>
  <code class="text">
    2015-08-18 23:47:53 DEBUG App:18 - Tue Aug 18 23:47:53 GST 2015
  </code>
</pre>

Теперь поправим pom.xml таким образом, чтобы он стал собирать наш проект в jar файл:
<pre>
  <code class="xml">
  &lt;build&gt;&#10;      &lt;plugins&gt;&#10;          &lt;plugin&gt;&#10;              &lt;groupId&gt;org.apache.maven.plugins&lt;/groupId&gt;&#10;              &lt;artifactId&gt;maven-compiler-plugin&lt;/artifactId&gt;&#10;              &lt;version&gt;2.3.2&lt;/version&gt;&#10;              &lt;configuration&gt;&#10;                  &lt;source&gt;1.7&lt;/source&gt;&#10;                  &lt;target&gt;1.7&lt;/target&gt;&#10;              &lt;/configuration&gt;&#10;          &lt;/plugin&gt;&#10;          &lt;plugin&gt;&#10;              &lt;groupId&gt;org.apache.maven.plugins&lt;/groupId&gt;&#10;              &lt;artifactId&gt;maven-jar-plugin&lt;/artifactId&gt;&#10;              &lt;configuration&gt;&#10;                  &lt;archive&gt;&#10;                      &lt;manifest&gt;&#10;                          &lt;addClasspath&gt;true&lt;/addClasspath&gt;&#10;                          &lt;classpathPrefix&gt;lib/&lt;/classpathPrefix&gt;&#10;                          &lt;mainClass&gt;io.github.itnotes.App&lt;/mainClass&gt;&#10;                      &lt;/manifest&gt;&#10;                  &lt;/archive&gt;&#10;              &lt;/configuration&gt;&#10;          &lt;/plugin&gt;&#10;      &lt;/plugins&gt;&#10;  &lt;/build&gt;
  </code>
</pre>

запускам `mvn clean compile package` и получаем jar файл "zip-with-dependencies-1.0-SNAPSHOT.jar" в папке target. Перейдём в папку target и попробуем запустить его командой `java -jar zip-with-dependencies-1.0-SNAPSHOT.jar`. Получаем следующую ошибку:
<pre>
  <code class="text">
  Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/log4j/Logger
          at io.github.itnotes.App.main(App.java:12)
  Caused by: java.lang.ClassNotFoundException: org.apache.log4j.Logger
          at java.net.URLClassLoader.findClass(Unknown Source)
          at java.lang.ClassLoader.loadClass(Unknown Source)
          at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
          at java.lang.ClassLoader.loadClass(Unknown Source)
          ... 1 more
  </code>
</pre>

Всё логично, рядом с нашим jar файлом нету необходимых ему зависимостей. Добавим maven-dependency-plugin, который выкачает необходимые зависимости и положит их в папку lib (ранее мы указали папку lib как префикс classpath'a - `<classpathPrefix>lib/</classpathPrefix>`):
<pre>
  <code class="xml">
  &lt;plugin&gt;&#10;      &lt;groupId&gt;org.apache.maven.plugins&lt;/groupId&gt;&#10;      &lt;artifactId&gt;maven-dependency-plugin&lt;/artifactId&gt;&#10;      &lt;version&gt;2.5&lt;/version&gt;&#10;      &lt;executions&gt;&#10;          &lt;execution&gt;&#10;              &lt;id&gt;copy-dependencies&lt;/id&gt;&#10;              &lt;phase&gt;prepare-package&lt;/phase&gt;&#10;              &lt;goals&gt;&#10;                  &lt;goal&gt;copy-dependencies&lt;/goal&gt;&#10;              &lt;/goals&gt;&#10;              &lt;configuration&gt;&#10;                  &lt;outputDirectory&gt;${project.build.directory}/lib&lt;/outputDirectory&gt;&#10;                  &lt;excludeArtifactIds&gt;junit,hamcrest-core&lt;/excludeArtifactIds&gt;&#10;              &lt;/configuration&gt;&#10;          &lt;/execution&gt;&#10;      &lt;/executions&gt;&#10;  &lt;/plugin&gt;
  </code>
</pre>

Возвращаемся в корень проекта и еще раз выполняем `mvn clean compile package`. Отлично, теперь в папке target лежит готовый jar файл, а в target/lib лежат библиотеки к нему. Если запустить его командой `java -jar zip-with-dependencies-1.0-SNAPSHOT.jar`, то код выполнится и рядом с jar файлом появится файл `application.log` с информацией о времени запуска программы.

Осталось запаковать всё необходимое в zip архив.

Для создания архивов есть замечательный assembly плагин. Добавим его:
<pre>
  <code class="xml">
&lt;plugin&gt;&#10;    &lt;artifactId&gt;maven-assembly-plugin&lt;/artifactId&gt;&#10;    &lt;configuration&gt;&#10;        &lt;archive&gt;&#10;            &lt;manifest&gt;&#10;                &lt;mainClass&gt;io.github.itnotes.App&lt;/mainClass&gt;&#10;            &lt;/manifest&gt;&#10;        &lt;/archive&gt;&#10;        &lt;descriptors&gt;&#10;            &lt;descriptor&gt;src/main/assembly/zip.xml&lt;/descriptor&gt;&#10;        &lt;/descriptors&gt;&#10;    &lt;/configuration&gt;&#10;    &lt;executions&gt;&#10;        &lt;execution&gt;&#10;            &lt;id&gt;make-assembly&lt;/id&gt;&#10;            &lt;phase&gt;package&lt;/phase&gt;&#10;            &lt;goals&gt;&#10;                &lt;goal&gt;single&lt;/goal&gt;&#10;            &lt;/goals&gt;&#10;        &lt;/execution&gt;&#10;    &lt;/executions&gt;&#10;&lt;/plugin&gt;
  </code>
</pre>

Assembly плагин требует дескриптор для конфигурации (в нём описывается какие файлы включать/не включать в архив и т.д). Добавим дескриптор src/main/assembly/zip.xml:
<pre>
  <code class="xml">
  &lt;assembly xmlns=&quot;http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0&quot;&#10;            xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;&#10;            xsi:schemaLocation=&quot;http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0 http://maven.apache.org/xsd/assembly-1.1.0.xsd&quot;&gt;&#10;      &lt;formats&gt;&#10;          &lt;format&gt;zip&lt;/format&gt;&#10;      &lt;/formats&gt;&#10;      &lt;includeBaseDirectory&gt;false&lt;/includeBaseDirectory&gt;&#10;      &lt;fileSets&gt;&#10;          &lt;fileSet&gt;&#10;              &lt;directory&gt;${project.basedir}/target&lt;/directory&gt;&#10;              &lt;outputDirectory/&gt;&#10;              &lt;includes&gt;&#10;                  &lt;include&gt;*.jar&lt;/include&gt;&#10;              &lt;/includes&gt;&#10;          &lt;/fileSet&gt;&#10;          &lt;fileSet&gt;&#10;              &lt;directory&gt;${project.basedir}/target/lib&lt;/directory&gt;&#10;              &lt;outputDirectory&gt;lib&lt;/outputDirectory&gt;&#10;              &lt;includes&gt;&#10;                  &lt;include&gt;*.*&lt;/include&gt;&#10;              &lt;/includes&gt;&#10;          &lt;/fileSet&gt;&#10;      &lt;/fileSets&gt;&#10;  &lt;/assembly&gt;
  </code>
</pre>

Теперь после запуска `mvn clean compile package` получаем zip архив со всеми зависимостями. После его распаковки всё работает без каких либо танцев с бубном.

Ссылка на готовый проект на [GutHub](https://github.com/itnotes/maven-zip-with-dependencies).

## Что дальше?
А дальше "раз-раз и в продакшен!".

На самом деле область применения данного решения довольно большая: можно подготавливать релизы для публикации на github/bitbucket, подготавливать архивы для передачи заказчику и т.д.
