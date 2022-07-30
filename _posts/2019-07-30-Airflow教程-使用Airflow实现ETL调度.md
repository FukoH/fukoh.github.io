---
title: 'Airflow教程-使用Airflow实现ETL调度'
date: 2019-07-30
excerpt: 我使用Airflow的经验
permalink: /posts/2019/07/Airflow教程-使用Airflow实现ETL调度/
tags:
  - Data Engineering
---

# 一、Airflow是什么
airflow 是一个编排、调度和监控workflow的平台，由Airbnb开源，现在在Apache Software Foundation 孵化。airflow 将workflow编排为由tasks组成的DAGs(有向无环图)，调度器在一组workers上按照指定的依赖关系执行tasks。同时，airflow 提供了丰富的命令行工具和简单易用的用户界面以便用户查看和操作，并且airflow提供了监控和报警系统。

# 二、Airflow的核心概念

1. DAGs：即有向无环图(Directed Acyclic Graph)，将所有需要运行的tasks按照依赖关系组织起来，描述的是所有tasks执行的顺序。
2. Operators：airflow内置了很多operators，如BashOperator 执行一个bash 命令，PythonOperator 调用任意的Python 函数，EmailOperator 用于发送邮件，HTTPOperator 用于发送HTTP请求， SqlOperator 用于执行SQL命令...同时，用户可以自定义Operator，这给用户提供了极大的便利性。可以理解为用户需要的一个操作,是Airflow提供的类
3. Tasks：Task 是 Operator的一个实例
4. Task Instance：由于Task会被重复调度,每次task的运行就是不同的task instance了。Task instance 有自己的状态，包括"running", "success", "failed", "skipped", "up for retry"等。
5. Task Relationships：DAGs中的不同Tasks之间可以有依赖关系

# 三、使用AirFlow完成天级的任务调度
说了这么多抽象的概念，估计看官还是云里雾里，下面就直接举个例子来说明吧。

##1. 安装airflow
Airflow可以约等于只支持linux和mac,Windows上极其难装,笔者放弃了.
安装也很简单，以下代码来自官方文档，使用了Python的pip管理：

    # airflow needs a home, ~/airflow is the default,
    # but you can lay foundation somewhere else if you prefer
    # (optional)
    export AIRFLOW_HOME=~/airflow

    # install from pypi using pip
    pip install apache-airflow

    # initialize the database
    airflow initdb

    # start the web server, default port is 8080
    airflow webserver -p 8080

    # start the scheduler
    airflow scheduler

    # visit localhost:8080 in the browser and enable the example dag in the home page

安装好了以后访问localhost:8080即可访问ui界面

## 2. 基本配置
1. 需要创建`~/airflow/dags`目录,这个目录是默认的存放DAG的地方,想修改的话可以修改`~/airflow/airflow.cfg`文件
2. 修改airflow的数据库
airflow会使用sqlite作为默认的数据库,此情况下airflow进行调度的任务都只能单个的执行.在调度任务量不大的情况下,可以使用sqlite作为backend.如果想scale out的话,需要修改配置文件,官方推荐使用mysql或者postgresql作为backend数据库.

## 3. 使用PostgresOperator执行SQL完成ETL任务
通过搜集信息,了解到PostgresOperator能执行SQL,并且还支持传参数.能解决大多数ETL任务中的传参问题.传参使用的是Python的Jinjia模块.

1. 创建DAG
首先创建一个test_param_sql.py文件.内容如下:
```python
from datetime import datetime, timedelta
import airflow
from airflow.operators.postgres_operator import PostgresOperator
from airflow.operators.dummy_operator import DummyOperator
from airflow.models import Variable

args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2019, 7, 26), #start_date会决定这个DAG从哪天开始生效
    'email': ['airflow@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    # 'queue': 'bash_queue',
    # 'pool': 'backfill',
    # 'priority_weight': 10,
    # 'end_date': datetime(2016, 1, 1),
}
# Variable是Airflow提供的用户自定义变量的功能,在UI界面的Admin -> Variable下可以进行增删改查,此处笔者定义了sql_path作为存放sql文件的地方
tmpl_search_path = Variable.get("sql_path")  

dag = airflow.DAG(
    'test_param_sql',
    schedule_interval=timedelta(days=1), # schedule_interval是调度的频率
    template_searchpath=tmpl_search_path, 
    default_args=args,
    max_active_runs=1)

test_param_sql = PostgresOperator(
    task_id='test_param_sql',
    postgres_conn_id='postgres_default',
    sql='param_sql.sql',
    dag=dag,
    params={'period': '201905'},
    pool='pricing_pool')

match_finish = DummyOperator(
    task_id='match_finish',
    dag=dag
)

test_param_sql >> match_finish
```

2. 准备要执行的Sql文件
创建test_sql.sql文件.
SQL文件会被Jinjia解析,可以使用一些宏来实现时间的替换 例

{{ ds }}  会被转换为当天的 YYYY-MM-DD 格式的日期

{{ ds_nodash }} 会被转换为当天的  YYYYMMDD的格式的日期

在本例里则是通过{{params.period}} 取到了 DAG上传入的参数,

```sql
insert into test.param_sql_test
select * from test.dm_input_loan_info_d
where period = {{params.period}};
```

3. 整体的目录结构如下
dags/
        test_param_sql.py
sql/
        test_sql.sql

4. 测试dag是否正确
可以使用   `airflow test  dag_id task_id date` 进行测试,测试会执行Operator,Operator指定的行为会进行调度. 但是不会将执行的行为记录到Airflow的数据库里

5. 发布
把文件放到~/airflow/dags目录下,sql文件不要放在dags目录下,可以找其他地方(比如同级目录),配置好上文说到的Variable,能找到即可.笔者的理解是,airflow会扫描dags目录下的内容,并尝试解析成dag,如果有不能成功解析的内容,ui界面上会有错误提示,导致dag显示不出来等问题.

## 其他有用的信息
1. 如何在dag.py里引入其他的本地python模块
需要把本地的python模块放到一个zip文件里,例如:
my_dag1.py
my_dag2.py
package1/__init__.py
package1/functions.py
然后把这个zip文件放到dags目录下,才能被正确解析

2. pooling可以控制任务的并行度,如果给DAG指定了一个不存在的pooling,任务会一直处于scheduled的状态,不继续进行