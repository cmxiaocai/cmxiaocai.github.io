---
layout: post
title: "在jenkins中用phing插件构建php项目"
categories: jenkins
description: 
keywords: php,phing
---

> phing是PHP 项目构建工具，在jenkins配合phing使用能够优雅的实现打包部署。

## phing安装

> 装好php后下载phar包即可

```
  wget http://www.phing.info/get/phing-2.16.1.phar
  cp phing-2.16.1.phar /usr/bin/phing
  chmod +x /usr/bin/phing
  phing -h
```

## 开始

> 创建一个'build.xml'配置文件

```shell
<?xml version="1.0" encoding="UTF-8"?>
<project name="test.com" basedir="./" default="start">
    <target name="start">
        <echo msg="start..." />
    </target>
</project>
```

> phing运行构建

```shell
[root@local test_build01]# phing -f build.xml
Buildfile: /home/phing_build/test_build01/build.xml

test.com > start:

     [echo] start...

BUILD FINISHED

Total time: 0.0310 seconds
```

## 完整示例

> composer install -> clear -> tar打包

**build.xml**

```shell
<?xml version="1.0" encoding="UTF-8"?>
<project name="test.com" basedir="./" default="clear">

    <property name="build.dir" value="build" />
    <property name="project.dir" value="test.com" />
    <property name="project.name" value="test.com" />

    <target name="prepare">
        <echo msg="Making directory ./${build.dir}" />
        <delete dir="./${build.dir}" />
        <mkdir dir="./${build.dir}" />
    </target>

    <target name="build:copy" depends="prepare">
        <copy todir="${build.dir}" >
          <fileset defaultexcludes="true" expandsymboliclinks="true" dir="${project.dir}">
                <include name="**" />
          </fileset>
        </copy>
        <delete file="./${build.dir}/composer.lock" />
        <delete dir="./${build.dir}/vendor" />
        <exec checkreturn="true" command="ls -l" dir="./${build.dir}" logoutput="true" />
    </target>

    <target name="build:composer" depends="build:copy">
        <if>
            <available file="./${build.dir}/composer.json" />
            <then>
                <exec checkreturn="true" command="composer install --no-dev --prefer-dist -o --no-progress" passthru="true" logoutput="true" dir="./${build.dir}" />
            </then>
            <else>
                <echo message="ComposerInstall: composer.json does not exist!" />
            </else>
        </if>
    </target>

    <target name="deploy:create-package" depends="build:composer">
        <tstamp>
            <format property="build.timestamp" pattern="%Y%m%d%H%M%S"/>
        </tstamp>
        <property name="build.release" value="${project.name}-${build.timestamp}" />
        <property name="package.name" value="${build.release}.tar.gz" />
        <echo>Creating dist package ${package.name}</echo>
        <tar destfile="${package.name}" basedir="${build.dir}" />
    </target>

    <target name="clear" depends="deploy:create-package">
        <delete file="./${build.dir}/composer.lock" />
        <delete file="./${build.dir}/composer.json" />
        <delete file="./${build.dir}/*.md" />
    </target>
</project>
```

**result:**

```shell
[root@local test_build01]# phing -f build_01.xml
Buildfile: /home/phing_build/test_build01/build_01.xml

test.com > prepare:

     [echo] Making directory ./build
   [delete] Deleting directory /home/phing_build/test_build01/build
    [mkdir] Created dir: /home/phing_build/test_build01/build

test.com > build:copy:

     [copy] Created 82 empty directories in /home/phing_build/test_build01/build
     [copy] Copying 325 files to /home/phing_build/test_build01/build
   [delete] Deleting: /home/phing_build/test_build01/build/composer.lock
   [delete] Deleting directory /home/phing_build/test_build01/build/vendor
     [exec] total 4
     [exec] drwxr-xr-x 6 root root 132 Mar  1 17:28 app
     [exec] -rw-r--r-- 1 root root 206 Mar  1 17:28 composer.json
     [exec] drwxr-xr-x 2 root root  59 Mar  1 17:28 config
     [exec] drwxr-xr-x 6 root root  89 Mar  1 17:28 public
     [exec] drwxr-xr-x 2 root root  22 Mar  1 17:28 scripts
     [exec] drwxr-xr-x 5 root root  45 Mar  1 17:28 templates

test.com > build:composer:

Do not run Composer as root/super user! See https://getcomposer.org/root for details
Loading composer repositories with package information
Updating dependencies
Package operations: 10 installs, 0 updates, 0 removals
  - Installing psr/http-message (1.0.1): Loading from cache
  - Installing slim/php-view (2.2.0): Loading from cache
  - Installing guzzlehttp/promises (v1.3.1): Loading from cache
  - Installing guzzlehttp/psr7 (1.4.2): Loading from cache
  - Installing guzzlehttp/guzzle (6.3.0): Loading from cache
  - Installing psr/container (1.0.0): Loading from cache
  - Installing container-interop/container-interop (1.2.0): Loading from cache
  - Installing nikic/fast-route (v1.3.0): Downloading (100%)
  - Installing pimple/pimple (v3.2.3): Loading from cache
  - Installing slim/slim (3.9.2): Loading from cache
Writing lock file
Generating optimized autoload files

test.com > deploy:create-package:

     [echo] Creating dist package test.com-20180301092923.tar.gz
      [tar] Building tar: /home/phing_build/test_build01/test.com-20180301092923.tar.gz

test.com > clear:

   [delete] Deleting: /home/phing_build/test_build01/build/composer.lock
   [delete] Deleting: /home/phing_build/test_build01/build/composer.json
   [delete] Could not find file /home/phing_build/test_build01/build/*.md to delete.

BUILD FINISHED

Total time: 58.8522 seconds
```



## jenkins plguin phing插件

> 在系统设置中安装plguin phing插件

![](http://www.xiaocai.name/uploads/201803/phing.png)