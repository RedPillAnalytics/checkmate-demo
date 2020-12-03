def options = '-S'
def gradle = "./gradlew ${options}"

pipeline {
  agent {
    kubernetes {
      defaultContainer 'oas'
      yamlFile 'pod-template.yaml'
      slaveConnectTimeout 1000
    }
  }

  environment {
    ORG_GRADLE_PROJECT_githubToken = credentials('github-redpillanalyticsbot-secret')
    AWS = credentials("rpa-development-build-server-svc")
    AWS_ACCESS_KEY_ID = "${env.AWS_USR}"
    AWS_SECRET_ACCESS_KEY = "${env.AWS_PSW}"
    ADMIN = credentials('obiee-admin-user')
    REPOSITORY_PSW = credentials('obiee-repository-password')
  }

  stages {

    stage('Build') {
      when { changeRequest() }
      steps {
        sh "$gradle featureCompare buildZip deployZip -Pobi.repositoryPassword=${env.REPOSITORY_PSW}"
      }
      post {
        always {
           junit testResults: 'obi/build/test-groups/**/*.xml', allowEmptyResults: true
        }
      }
    }

    stage('Baseline Test') {
       when { changeRequest() }
       steps {
          sh "$gradle featureBaselineWorkflow -Pobi.adminUser=${env.ADMIN_USR} -Pobi.adminPassword=${env.ADMIN_PSW} -Pobi.repositoryPassword=${env.REPOSITORY_PSW}"
       }
       post {
          always {
             junit testResults: 'obi/build/test-groups/**/*.xml', allowEmptyResults: true
          }
       }
    }

    stage('Revision Test') {
       when { changeRequest() }
       steps {
          sh "$gradle revisionWorkflow -Pobi.adminUser=${env.ADMIN_USR} -Pobi.adminPassword=${env.ADMIN_PSW} -Pobi.repositoryPassword=${env.REPOSITORY_PSW}"
       }
       post {
          always {
             junit testResults: 'obi/build/test-groups/**/*.xml', allowEmptyResults: true
          }
       }
    }

    stage('Publish') {
       when { branch "master" }
       steps {
          sh "$gradle featureCompare publish -Pobi.repositoryPassword=${env.REPOSITORY_PSW}"
       }
    }

    stage('Deploy to QA') {
       when { branch "master" }
       steps {
          sh "$gradle importWorkflow -Pobi.adminUser=${env.ADMIN_USR} -Pobi.adminPassword=${env.ADMIN_PSW} -Pobi.repositoryPassword=${env.REPOSITORY_PSW}"
       }
    }
}
  post {
    always {
       archiveArtifacts artifacts: 'obi/build/distributions/*.zip', fingerprint: true, allowEmptyArchive: true
       sh "$gradle producer"
    }
  }
}
