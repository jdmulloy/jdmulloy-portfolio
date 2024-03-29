# Table of Contents

* [Celery Redis Monitoring Scripts](#celery-redis-monitoring-scripts)
* [Tetris](#tetris)

# Celery Redis Monitoring Scripts

I wrote these scripts while working at edX.org to monitor celery queues stored in redis. The motivation for writing these scripts was to be able to monitor the celery queues after migrating from RabbitMQ to Redis. These scripts were moved into a private repository, so the versions that I can show publically are missing some more recent updates. Please see a description of each script below.

Originally these ran once a minute in jenkins jobs. Then I moved them into once per minute Kubernetes cronjobs. More recently I modified them to sleep until the beginning of the next minute so that they could run indefinately in a Kubernetes pod. The motivation for not running them as Kubernetes crons is that Kuberntes nodes have resource leaks when pods are created and destroyed too often, which in this case was 6 pods once a minute.

## Celery Queue Size Monitoring (update_celery_monitoring.py)

[Source](https://github.com/jdmulloy/configuration/blob/jdmulloy/undelete_celery_scripts/util/jenkins/update_celery_monitoring/update_celery_monitoring.py)

[Git History](https://github.com/jdmulloy/configuration/commits/jdmulloy/undelete_celery_scripts/util/jenkins/update_celery_monitoring/update_celery_monitoring.py)

This is the first monitoring script I wrote for monitoring celery queues in redis after we migrated off of RabbitMQ. This script publishes the size of each celery queue in Redis so that AWS CloudWatch can trigger an alert if any of the queues becomes too full. This script also counts the number of worker EC2 instances and publishes that count to CloudWatch so that it can be graphed in a CloudWatch dashboard, to show correlation between the queue metrics and how many workers were running.

We had some processes that would periodically add a large batch of work to a queue that would then slowly be worked on over the course of hours. Over time the size of these batches grew and we kept increasing the alert threshold which resulted in nuisance alerts and made our alerts less sensitive to potential issues. The solution to this was to add the check_celery_progress.py to better track if the queue was being processed, rather than just how large it was. However we kept the monitoring for queue size with high thresholds so that we could still catch a large backup.

## Celery Task Age Monitoring (check_celery_progress.py)

[Source](https://github.com/jdmulloy/configuration/blob/jdmulloy/undelete_celery_scripts/util/jenkins/check_celery_progress/check_celery_progress.py)

[Git History](https://github.com/jdmulloy/configuration/commits/jdmulloy/undelete_celery_scripts/util/jenkins/check_celery_progress/check_celery_progress.py)

This script tracks how long the item at the front of the queue has been at the front of the queue. The idea is to detect if the workers have stopped processing a queue so that the appropriate owner can be alerted. The age of the next task in seconds is publshed to AWS CloudWatch so that operators can view trends over time in a dashboard created by create_celery_dashboard.py

## Automated CloudWatch Dashboard Updates (create_celery_dashboard.py)

[Source](https://github.com/jdmulloy/configuration/blob/jdmulloy/undelete_celery_scripts/util/jenkins/update_celery_monitoring/create_celery_dashboard.py)

[Git History](https://github.com/jdmulloy/configuration/commits/jdmulloy/undelete_celery_scripts/util/jenkins/update_celery_monitoring/create_celery_dashboard.py)

I wrote this script to keep the Celery CloudWatch dashboards up to date. The main motivation for this is that our deployments create new ASGs(Autoscaling Groups) so the metrics from the autoscaling group, such as CPU, are constantly changing. Before this script, every time I needed to use the dashboard I had to manually add the CPU metrics for the most recent ASGs. This script also automatically adds new queues to the charts when they are created. Without this script new queues had to be manually added to the dashboards.

# Tetris
[Source](https://github.com/jdmulloy/python-tetris/blob/master/tetris.py)

[Git History](https://github.com/jdmulloy/python-tetris/commits/master/)

This is a curses console based Tertis clone I wrote in Python while job searching in 2017. I wrote it as a way to practice Python and demostrate my ability to write Python. I've tested it on my personal Macbook as of March 2024 and it still works.
