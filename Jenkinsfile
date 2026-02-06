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
          def repos = listGithubRepos("ankit96khokhar")

          def selectedRepo = input(
            message: "Select repository",
            parameters: [
              choice(name: 'REPO', choices: repos.join('\n'))
            ]
          )

          env.REPO = selectedRepo     
        }
      }
    }

  }

}

