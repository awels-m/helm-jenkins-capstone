pipeline {
  agent any

  options {
    // Removed ansiColor here to avoid plugin dependency
    timestamps()
  }

  environment {
    // Ensure Homebrew-installed tools are visible to Jenkins on macOS
    PATH = "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:${PATH}"

    // ----- Adjust these if you like -----
    NAMESPACE = 'capstone'
    RELEASE   = 'demo'
    CHART_DIR = 'webapp'
  }

  stages {
    stage('Checkout') {
      steps {
        sh '''
          set -euxo pipefail
          pwd
          ls -la
        '''
      }
    }

    stage('Kubernetes Context') {
      steps {
        sh '''
          set -euxo pipefail
          echo "kubectl version:"; kubectl version --client
          echo "helm version:"; helm version
          echo "current kubectl context:"; kubectl config current-context || true

          # Ensure target namespace exists
          kubectl get ns "$NAMESPACE" >/dev/null 2>&1 || kubectl create ns "$NAMESPACE"

          echo "Cluster nodes:"
          kubectl get nodes -o wide
        '''
      }
    }

    stage('Helm Lint & Template') {
      steps {
        sh '''
          set -euxo pipefail
          helm lint "./$CHART_DIR"
          echo "Rendering first lines of the manifest:"
          helm template "$RELEASE" "./$CHART_DIR" | head -n 80
        '''
      }
    }

    stage('Deploy/Upgrade with Helm') {
      steps {
        sh '''
          set -euxo pipefail
          helm upgrade --install "$RELEASE" "./$CHART_DIR" -n "$NAMESPACE"
          kubectl rollout status deploy/"$RELEASE"-webapp -n "$NAMESPACE" --timeout=180s || true
          kubectl get pods,svc -n "$NAMESPACE" -o wide
        '''
      }
    }

    stage('How to access the app') {
      steps {
        sh '''
          set -euxo pipefail
          echo "If using minikube: run 'minikube tunnel' in another terminal for EXTERNAL-IP."
          echo "Otherwise, port-forward with:"
          echo "kubectl -n $NAMESPACE port-forward svc/$RELEASE-webapp 8080:80"
        '''
      }
    }
  }

  post {
    success { echo '✅ Deployment succeeded.' }
    failure { echo '❌ Deployment failed. Open Console Output to see the failing command.' }
  }
}

