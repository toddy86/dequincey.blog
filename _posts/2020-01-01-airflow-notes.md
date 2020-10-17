---
layout: post
title: Airflow Notes and Resources
categories: resources
---

A living and ever changing library of Airflow notes, code snippets and resources.

<!-- more -->

## Documentation Notes
### Concepts
Note that SubDAG operators should contain a factory method that returns a DAG object. This will prevent the SubDAG from being treated like a separate DAG in the main UI. For example:

As another example, consider the following DAG: We can combine all of the parallel task-* operators into a single SubDAG, so that the resulting DAG resembles the following:

Operators are only loaded by Airflow if they are assigned to a DAG.

Connections¶ The connection information to external systems is stored in the Airflow metadata database and managed in the UI (Menu -> Admin -> Connections). A conn_id is defined there and hostname / login / password / schema information attached to it. Airflow pipelines can simply refer to the centrally managed conn_id without having to hard code any of this information anywhere.

dags/subdag.py from airflow.models import DAG from airflow.operators.dummy_operator import DummyOperator # Dag is returned by a factory method def sub_dag(parent_dag_name, child_dag_name, start_date, schedule_interval): dag = DAG( '%s.%s' % (parent_dag_name, child_dag_name), schedule_interval=schedule_interval, start_date=start_date, ) dummy_operator = DummyOperator( task_id='dummy_task', dag=dag, ) return dag

Once an operator is instantiated, it is referred to as a “task”.

Airflow, a DAG – or a Directed Acyclic Graph – is a collection of all the tasks you want to run, organized in a way that reflects their relationships and dependencies.

In general, each one should correspond to a single logical workflow.
DAG describes how you want to carry out your workflow

main_dag.py from datetime import datetime, timedelta from airflow.models import DAG from airflow.operators.subdag_operator import SubDagOperator from dags.subdag import sub_dag PARENT_DAG_NAME = 'parent_dag' CHILD_DAG_NAME = 'child_dag' main_dag = DAG( dag_id=PARENT_DAG_NAME, schedule_interval=timedelta(hours=1), start_date=datetime(2016, 1, 1) ) sub_dag = SubDagOperator( subdag=sub_dag(PARENT_DAG_NAME, CHILD_DAG_NAME, main_dag.start_date, main_dag.schedule_interval), task_id=CHILD_DAG_NAME, dag=main_dag, )

DAGs are defined in standard Python files that are placed in Airflow’s DAG_FOLDER

When setting relationships between two list of operators and wish all up list operators as upstream to all down list operators, we have to split one list manually using bitshift composition. [op1, op2, op3] >> op4 [op1, op2, op3] >> op5 [op1, op2, op3] >> op6 cross_downstream could handle list relationships easier. cross_downstream([op1, op2, op3], [op4, op5, op6]) When setting single direction relationships to many operators, we could concat them with bitshift composition. op1 >> op2 >> op3 >> op4 >> op5 use chain could do that chain(op1, op2, op3, op4, op5)
Bitshift can also be used with lists. For example: op1 >> [op2, op3] >> op4 is equivalent to: op1 >> op2 >> op4 op1 >> op3 >> op4

The BranchPythonOperator is much like the PythonOperator except that it expects a python_callable that returns a task_id (or list of task_ids).

Some other tips when using SubDAGs: by convention, a SubDAG’s dag_id should be prefixed by its parent and a dot. As in parent.child share arguments between the main DAG and the SubDAG by passing arguments to the SubDAG operator (as demonstrated above) SubDAGs must have a schedule and be enabled. If the SubDAG’s schedule is set to None or @once, the SubDAG will succeed without having done anything clearing a SubDagOperator also clears the state of the tasks within marking success on a SubDagOperator does not affect the state of the tasks within refrain from using depends_on_past=True in tasks within the SubDAG as this can be confusing it is possible to specify an executor for the SubDAG. It is common to use the SequentialExecutor if you want to run the SubDAG in-process and effectively limit its parallelism to one. Using LocalExecutor can be problematic as it may over-subscribe your worker, running multiple tasks in a single slot

An operator describes a single task in a workflow. Operators are usually (but not always) atomic, meaning they can stand on their own and don’t need to share resources with any other operators.

Critically, that means the DAG must appear in globals(). Consider the following two DAGs. Only dag_1 will be loaded; the other one only appears in a local scope. dag_1 = DAG('this_dag_will_be_discovered') def my_function(): dag_2 = DAG('but_this_dag_will_not') my_function()

DAG isn’t concerned with what its constituent tasks do; its job is to make sure that whatever they do happens at the right time, or in the right order, or with the right handling of any unexpected issues.

Airbnb uses the stage-check-exchange pattern when loading data. Data is staged in a temporary table, after which data quality checks are performed against that table. Once the checks all pass the partition is moved into the production table.

We recommend you setting operator relationships with bitshift operators rather than set_upstream() and set_downstream().

Variables are a generic way to store and retrieve arbitrary content or settings as a simple key value store within Airflow. Variables can be listed, created, updated and deleted from the UI (Admin -> Variables), code or CLI

DAG: a description of the order in which work should take place Operator: a class that acts as a template for carrying out some work Task: a parameterized instance of an operator Task Instance: a task that 1) has been assigned to a DAG and 2) has a state associated with a specific run of the DAG

When searching for DAGs, Airflow only considers python files that contain the strings “airflow” and “DAG” by default. To consider all python files instead, disable the DAG_DISCOVERY_SAFE_MODE configuration flag.

### Tutorial
python /airflow/dags/tutorial.py If the script does not raise an exception it means that you haven’t done anything horribly wrong, and that your Airflow environment is somewhat sound.
Testing Let’s test by running the actual task instances on a specific date. The date specified in this context is an execution_date, which simulates the scheduler running your task or dag at a specific date + time: # command layout: command subcommand dag_id task_id date # testing print_date airflow test tutorial print_date 2015-06-01 # testing sleep airflow test tutorial sleep 2015-06-01

Files can also be passed to the bash_command argument, like bash_command='templated_command.sh', where the file location is relative to the directory containing the pipeline file (tutorial.py in this case). This may be desirable for many reasons, like separating your script’s logic and pipeline code, allowing for proper code highlighting in files composed in different languages, and general flexibility in structuring pipelines. It is also possible to define your template_searchpath as pointing to any folder locations in the DAG constructor call.

This should result in displaying a verbose log of events and ultimately running your bash command and printing the result.

Note that the airflow test command runs task instances locally, outputs their log to stdout (on screen), doesn’t bother with dependencies, and doesn’t communicate state (running, success, failed, …) to the database. It simply allows testing a single task instance.

