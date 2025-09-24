pipeline {
  agent any

  environment {
    // Use full paths for Apple Silicon Homebrew tools
    KUBECTL = '/opt/homebrew/bin/kubectl'
    HELM    = '/opt/homebrew/bin/helm'
    NAMESPACE = 'capstone'
    RELEASE   = 'demo'
    CHART_DIR = 'webapp'
  }

  options {
    timestamps()
    ansiColor('xterm')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'echo "Repo checked out at: $(pwd)" && ls -la'
      }
    }

    stage('Kube Context Check') {
      steps {
        sh """
          ${KUBECTL} config current-context
          ${KUBECTL} get nodes
          ${KUBECTL} get ns || true
          ${KUBECTL} create namespace ${NAMESPACE} || true
        """
      }
    }

    stage('Helm Lint & Template') {
      steps {
        sh """
          ${HELM} lint ${CHART_DIR}
          ${HELM} template ${RELEASE} ${CHART_DIR} --namespace ${NAMESPACE} | head -n 60
        """
      }
    }

    stage('Deploy/Upgrade') {
      steps {
        sh """
          ${HELM} upgrade --install ${RELEASE} ${CHART_DIR} -n ${NAMESPACE}
          ${KUBECTL} rollout status deploy/${RELEASE}-webapp -n ${NAMESPACE} --timeout=120s
        """
      }
    }

    stage('Smoke Test') {
      steps {
        sh """
          # Try to reach the service inside the cluster
          ${KUBECTL} get svc -n ${NAMESPACE}
          echo "If using port-forward, uncomment the lines below to curl:"
          # ${KUBECTL} port-forward -n ${NAMESPACE} svc/${RELEASE}-webapp 8080:80 &
          # PF_PID=$!
          # sleep 3
          # curl -s http://localhost:8080 | head -n 5
          # kill $PF_PID
        """
      }
    }
  }

  post {
    success {
      echo 'Deployment successful ✅'
    }
    failure {
      echo 'Deployment failed ❌'
    }
  }
}

