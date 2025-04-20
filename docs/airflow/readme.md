
https://airflow.apache.org/docs/helm-chart/stable/index.html
https://www.baeldung.com/ops/kubernetes-configmaps-secrets

helm show values apache-airflow/airflow
helm show values apache-airflow/airflow | grep -v '#' | grep -v '^$'
helm uninstall -n airflow airflow || kubectl delete namespaces airflow

(venv) mickael@deborah:~/Documents/dag$ helm repo add apache-airflow https://airflow.apache.org
"apache-airflow" has been added to your repositories
(venv) mickael@deborah:~/Documents/dag$ helm upgrade --install airflow apache-airflow/airflow --namespace airflow --create-namespace
Release "airflow" does not exist. Installing it now.
NAME: airflow
LAST DEPLOYED: Sun Mar 30 19:15:45 2025
NAMESPACE: airflow
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing Apache Airflow 2.9.3!

Your release is named airflow.
You can now access your dashboard(s) by executing the following command(s) and visiting the corresponding port at localhost in your browser:

Airflow Webserver:     kubectl port-forward svc/airflow-webserver 8080:8080 --namespace airflow
Default Webserver (Airflow UI) Login credentials:
    username: admin
    password: admin
Default Postgres connection credentials:
    username: postgres
    password: postgres
    port: 5432

You can get Fernet Key value by running the following:

    echo Fernet Key: $(kubectl get secret --namespace airflow airflow-fernet-key -o jsonpath="{.data.fernet-key}" | base64 --decode)

###########################################################
#  WARNING: You should set a static webserver secret key  #
###########################################################

You are using a dynamically generated webserver secret key, which can lead to
unnecessary restarts of your Airflow components.

Information on how to set a static webserver secret key can be found here:
https://airflow.apache.org/docs/helm-chart/stable/production-guide.html#webserver-secret-key


(venv) mickael@deborah:~/Documents/dag$ cat hello.py 
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime

with DAG(
    "simple_dag", start_date=datetime(2023, 1, 1), schedule_interval=None, catchup=False
) as dag:
    task1 = BashOperator(task_id="task_1", bash_command='echo "Hello, World!"')
    task2 = BashOperator(task_id="task_2", bash_command='echo "This is task 2"')
    task1 >> task2


helm upgrade --install prometheus prometheus-community/kube-prometheus-stack --namespace prometheus --create-namespace 
helm upgrade --install airflow apache-airflow/airflow --namespace airflow --create-namespace --values values.yml
