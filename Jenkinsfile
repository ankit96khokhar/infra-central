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

          /* ---------- Repo discovery ---------- */
          def repoList = listGithubRepos("ankit96khokhar")

          echo "===== Discovered GitHub Repositories ====="
          repoList.each { repo ->
            echo " - ${repo}"
          }
          echo "========================================="

          def selectedRepo = input(
            message: "Select GitHub repository",
            parameters: [
              choice(
                name: 'REPO',
                choices: repoList.join('\n')
              )
            ]
          )

          env.REPO = selectedRepo
          echo "Selected repo: ${env.REPO}"


          /* ---------- Branch discovery ---------- */
          def branchList = listGithubBranches("ankit96khokhar", env.REPO)

          echo "===== Discovered Branches for ${env.REPO} ====="
          branchList.each { br ->
            echo " - ${br}"
          }
          echo "============================================="

          def selectedBranch = input(
            message: "Select branch for ${env.REPO}",
            parameters: [
              choice(
                name: 'BRANCH',
                choices: branchList.join('\n')
              )
            ]
          )

          env.BRANCH = selectedBranch
          echo "Selected branch: ${env.BRANCH}"
        }
      }
    }

    stage('Checkout Service Repo') {
      steps {
        script {
          echo "Checking out repo: ${env.REPO}"
          echo "Branch: ${env.BRANCH}"

          checkout([
            $class: 'GitSCM',
            branches: [[name: "*/${env.BRANCH}"]],
            userRemoteConfigs: [[
              url: "https://github.com/ankit96khokhar/${env.REPO}.git",
              credentialsId: 'github-api-token'
            ]],
            extensions: [
              [$class: 'CleanBeforeCheckout'],
              [$class: 'CloneOption', shallow: true, depth: 1]
            ]
          ])
        }
      }
    }

    stage('Terraform Init') {
      steps {
        sh '''
          terraform version
          terraform init -backend=false -input=false
        '''
      }
    }

    stage('Terraform Fmt Check') {
      steps {
        sh '''
          terraform fmt -check -recursive
        '''
      }
    }

    stage('Terraform Validate') {
      steps {
        sh '''
          terraform validate
        '''
      }
    }

    stage('Terraform Plan') {
      steps {
        sh '''
          terraform plan -no-color
        '''
      }
    }
  }
}

