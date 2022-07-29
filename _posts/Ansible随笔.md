---
title: 'Ansible随笔'
date: 2019-11-09
permalink: /posts/2019/09/Ansible随笔/
tags:
  - DevOps
---

## 1.Ansible提示failed to execute setup

我遇到这一问题的原因是使用的用户不具有权限.下面用具体的例子来说说Ansible里的用户和权限.
目标: 以spark用户的身份在/usr/local/lib/jvm/下面装个JDK.

我的Ansible是只有gpadmin用户有权限使用的.spark用户不能使用ansible.gpadmin用户配置了免密登录,Spark用户没有配置免密登录,此时playbook如下:
    ---
    -
    hosts: sparkservers
    user: gpadmin
    become: yes
    become_user: root
    tasks:
      - name : push java tarball
        copy : src=/usr/local/lib/jvm/ dest=/usr/local/lib/jvm/

      - name : config java_home
        blockinfile : 
          dest=/etc/profile
          content="export JAVA_HOME=/.....
                    CLASSPATH=...
                    PATH=..."

上面的代码有很多细节,下面一一道来:
1. user使用了gpadmin,因为gpadmin是有免密登录的,如果使用spark用户会无法登录到远程机器,任务失败
2. 由于/usr/local/lib/jvm/目录需要root权限才能写,所以gpadmin还要有sudo权限,一开始我没有配置sudo权限,会报错.总结1和2也即是,用户需要能免密登录,要进行sudo操作的话还要有sudo权限.
3. ansible的详细错误信息可以使用ansible-playbook -vvv myplaybook.yml 来查看,其中的-vvv参数是是详细信息的开关
4. 使用sudo权限需要输入sudo密码,用法是加一个-K 参数ansible-playbook -vvv myplaybook.yml -K
5. become要和become_user一起使用,参考这篇博文[Ansible-become(权限提升)](https://blog.csdn.net/kozazyh/article/details/88082965)
6. 使用copy模块,路径最后加/和不加/是有区别的,加/会直接拷贝目录下的所有内容到目标目录,不加/则会先在src路径下创建一个文件夹,并且把要复制的内容都复制进去,然后再发送到各个节点. [参考](https://www.mydailytutorials.com/how-to-copy-files-and-directories-in-ansible-using-copy-and-fetch-modules/)
7. blockinline可以修改文件的多行,是lineinfile的扩展[参考](https://stackoverflow.com/questions/24334115/ansible-lineinfile-for-several-lines)