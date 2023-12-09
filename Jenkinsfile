def userId = slackUserIdFromEmail("${BUILD_USER_EMAIL}")
pipeline {
  agent any
  stages {
    stage('version') {
      steps {
        slackSend color: 'good', message: "Hi <@$userId> your build has started and url is ${env.BUILD_URL}"
        sh 'python3 --version'
      }
    }
    stage('Running requirements.txt') {
      steps {
       slackSend color: 'good', message: "Hi <@$userId> Running requirements.txt "
       sh 'pip3 install -r /bitnami/jenkins/home/workspace/dataflow-etl/dataflow/requirements.txt'
      }
    }
    stage('Running setup.py') {
      steps {
        dir('/bitnami/jenkins/home/workspace/dataflow-etl/dataflow/'){
          slackSend color: 'good', message: "Hi <@$userId> Running setup.py"
          sh 'python3 /bitnami/jenkins/home/workspace/dataflow-etl/dataflow/setup.py install --user'
        }
      }
    }
    stage('build the template and deploy on gcs') {
      steps {
       dir('/bitnami/jenkins/home/workspace/dataflow-etl/dataflow/'){
         slackSend color: 'good', message: "Hi <@$userId> build the template and deploy on gcs"
         sh 'python3 -m gcs_to_bq --runner DataflowRunner  --project playground-375318  --staging_location gs://bronze-poc-group/gcp-batch-raw-ingestion/dataflow/staging  --temp_location gs://bronze-poc-group/gcp-batch-raw-ingestion/dataflow/temp --template_location gs://bronze-poc-group/gcp-batch-raw-ingestion/dataflow/templates/gcs_to_bq --region US --config_file config/gcs_to_bq_config.json --setup_file ./setup.py --save_main_session'
         slackSend color: 'good', message: "Hi <@$userId> templated is deployed "
       }
      }
    }
    stage('Running Dag with airflow') {
       steps {
              withEnv(['GCLOUD_PATH=/usr/lib/google-cloud-sdk/bin']) {
              slackSend color: 'good', message: "Running Dag dataflow-python-etl with airflow"
                sh '$GCLOUD_PATH/gcloud composer environments  run  data-generator-demo --location us-central1  dags trigger -- dataflow-python-etl'

             }
         }
    }
  }
  post {
        success {
           slackSend color: 'good', message: "Hi <@$userId> Airflow dag is triggered please check the ui"
        }
        failure {
            slackSend color: 'danger', message: "Hi <@$userId> your build has failed pleas check ${env.BUILD_URL}"
                }
        }

}