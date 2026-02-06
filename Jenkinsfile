@Library('jenkins-shared-library@main') _

pipeline {
  agent {
    kubernetes {
      defaultContainer 'terraform'
      idleMinutes 1
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: terraform
    image: terraform-python-venv:local
    imagePullPolicy: IfNotPresent
    command: ["cat"]
    tty: true
"""
    }
  }

  environment {
    TF_IN_AUTOMATION = "true"
    TF_INPUT = "false"
    BLUEPRINT_REPO = 'git@github.com:ankit96khokhar/blueprint-config.git'
  }

  stages {
    stage('Discovery') {
      steps {
        script {
          def repoList = listGithubRepos("ankit96khokhar")
          echo "===== Discovered GitHub Repositories ====="
          repoList.each { repo ->
            echo " - ${repo}"
          }
          echo "========================================="          
        }
      }
    }

  }

}

