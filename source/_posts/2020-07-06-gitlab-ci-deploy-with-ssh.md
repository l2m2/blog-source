---
title: GitLab CI - 通过SSH部署
toc: false
date: 2020-07-06 13:30:00
description: 通过SSH自动部署你的目标文件
tags:
- CI
- GitLab
---

工作中的场景：需要将qdoc生成的html文件自动部署到nginx搭建的静态网站下。

**`.gitlab-ci.yml`**

```yaml
linux_develop:
  stage: build
  only:
    - develop
  tags:
    - centos6_cd
  script:
    - python ci/build.py -p $CI_PROJECT_DIR/src/$CI_PROJECT_NAME.pro -b $CI_PROJECT_DIR/build/$CI_COMMIT_REF_NAME/$CI_JOB_NAME -m release
    - python ci/build.py -p $CI_PROJECT_DIR/qdoc/$CI_PROJECT_NAME-qdoc.pro -b $CI_PROJECT_DIR/build/$CI_COMMIT_REF_NAME/$CI_JOB_NAME -m release
    - export SSHPASS=$TOPIKM6_DOC_SERVER_SSHPASS
    - sshpass -e scp -o stricthostkeychecking=no -prq qdoc/html/. $TOPIKM6_DOC_SERVER:/data/docker_data/topikm6doc/html/$CI_PROJECT_NAME/
    - cp -TR qdoc/html/. $TOPIKM6_DOCS/$CI_PROJECT_NAME/
```

`sshpass` 提供在命令行中直接输入密码进行SSH操作，而不需要交互。

`-e` 表示 密码使用环境变量中的`SSHPASS`。

**最佳实践**

在CI配置中增加`export SSHPASS=$TOPIKM6_DOC_SERVER_SSHPASS`，

然后在Runner中的`.bash_profile`中设定密码。

```bash
$ cat .bash_profile 
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH

### for Qt enviroment
QT_HOME=/opt/Qt5.6.3
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib64:/usr/local/lib
export PATH=$PATH:/opt/Qt5.6.3/5.6.3/gcc_64/bin:/opt/Qt5.6.3/Tools/QtCreator/bin
export QT_INSTALL_DOCS=/opt/Qt5.6.3/Docs/Qt-5.6.3
export TOPIKM6_DOC_SERVER_SSHPASS=xxx
```

**CentOS 6安装sshpass**

```bash
$ yum --enablerepo=epel -y install sshpass
```

## Reference

- https://stackoverflow.com/questions/58094583/gitlab-pipeline-fails-when-using-rsync-to-upload-to-staging-server
- https://codeburst.io/gitlab-build-and-push-to-a-server-via-ssh-6d27ca1bf7b4
- https://stackoverflow.com/questions/38129835/sshpass-command-not-found-error