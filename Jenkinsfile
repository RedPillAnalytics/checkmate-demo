def options = '-S'
def properties = "-Panalytics.buildTag=${env.BUILD_TAG}"
def gradle = "./gradlew ${options} ${properties}"
def domain = '/home/oracle/fmw/config/domains/bi'

pipeline {
   agent { label 'gce-obi-12.2.1.4' }

   environment {
      ORG_GRADLE_PROJECT_githubToken = credentials('github-redpillanalyticsbot-secret')
		AWS = credentials("rpa-development-build-server-svc")
		AWS_ACCESS_KEY_ID = "${env.AWS_USR}"
		AWS_SECRET_ACCESS_KEY = "${env.AWS_PSW}"
      ADMIN = credentials('obiee-admin-user')
      REPOSITORY_PSW = credentials('obiee-repository-password')
      JENKINS_NODE_COOKIE = 'dontKillMe'
   }

   stages {
      stage('Assemble') {
         when { changeRequest() }
         parallel {
            stage('Startup') {
               steps {
                  sh "$gradle startServices"
               }
            }
            stage('Build') {
               steps {
                  sh "$gradle featureCompare buildZip deployZip -Pobi.repositoryPassword=${env.REPOSITORY_PSW}"
               }
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
            sh "$gradle importWorkflow"
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
