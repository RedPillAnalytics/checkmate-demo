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
		GRADLE_COMBINED = credentials("gradle-publish-key")
		GRADLE_KEY = "${env.GRADLE_COMBINED_USR}"
		GRADLE_SECRET = "${env.GRADLE_COMBINED_PSW}"
      ADMIN = credentials('obiee-admin-user')
      repositoryPassword = credentials('obiee-repository-password')
   }

   stages {
      stage('Initial') {
         parallel {
            stage('Startup') {
               steps {
                  sh "$gradle startServices"
               }
            }
            stage('Build') {
               steps {
                  sh "$gradle build"
               }
            }
         }
         post {
            always {
               archiveArtifacts artifacts: 'build/distributions/*.zip', fingerprint: true, allowEmptyArchive: true
               sh "$gradle producer"
            }
         }
      }

      // stage('Test') {
      //    when { changeRequest() }
      //    environment {
      //       JENKINS_NODE_COOKIE = 'dontKillMe'
      //    }
      //    steps {
      //       sh "$gradle cleanJunit startOBI runAllTests"
      //    }
      //    post {
      //       always {
      //          junit testResults: 'build/test-results/**/*.xml', allowEmptyResults: true
      //          archiveArtifacts artifacts: 'build/tmp/onlineTest/**/build/distributions/*.zip', fingerprint: true, allowEmptyArchive: true
      //          sh "$gradle producer"
      //       }
      //    }
      // }

      // stage('Publish') {
      //    when { branch "master" }
      //    steps {
      //       sh "$gradle publish"
      //    }
      //    post {
      //       always {
      //          sh "$gradle producer"
      //       }
      //    }
      // }
   }
}
