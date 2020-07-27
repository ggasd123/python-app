#!groovy
pipeline {
  agent any

  environment {
    GIT_COMMIT_SHORT = sh(  script: "printf \$(git rev-parse --short ${GIT_COMMIT})", returnStdout: true)
  }

  stages {

    stage('Test') {
      steps {
        sh script: '''
        echo $GIT_COMMIT
        echo "Testing python-app before building"
        '''
      }
    }

    stage('Build') {
      steps {
        sh script: '''
        git checkout master
        echo $GIT_COMMIT
        echo $GIT_COMMIT_SHORT
        pwd
        echo "I am in the root folder"
        docker build --tag cloudgenius/python-app:$GIT_COMMIT .
        ''', label: "Building docker image for python-app"
      }
    }

    stage('Push to container registry') {
      steps {
        sh script: '''
        echo $D_PASS | docker login --username $D_USER --password-stdin
        docker push  cloudgenius/python-app:$GIT_COMMIT
        ''', label: "Pushing images container registry"
      }
    }

    stage('Production deploy') {
      steps {
        sh script: '''
          # grant k8s permissions
          echo $GKE_SERVICE_ACCOUNT | base64 --decode > /tmp/gke_key.json
          gcloud auth activate-service-account --key-file /tmp/gke_key.json
          gcloud config set project $GKE_PROJECT_ID
          gcloud config set compute/zone $GKE_COMPUTE_ZONE
          gcloud --quiet container clusters get-credentials $GKE_CLUSTER_NAME
          # edit k8s deployment
          kubectl get deploy
          kubectl set image deployment/python-app-deploy python-app=cloudgenius/python-app:$GIT_COMMIT
          kubectl annotate deployment.v1.apps/python-app-deploy kubernetes.io/change-cause=$GIT_COMMIT
          kubectl get deploy
        ''', label: "Deploying the python-app to production kubernetes cluster"
      }
    }
  }

  post {
        always {
            echo 'This will always run'
        }
        success {
            echo 'This will run only if successful'
        }
        failure {
            echo 'This will run only if failed'
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
        }
        changed {
            echo 'This will run only if the state of the Pipeline has changed'
            echo 'For example, if the Pipeline was previously failing but is now successful'
        }
    }
}
