pipeline {
  agent any
  environment {
    docker_username='stifstof'
  }
  stages {
    stage('clone down'){
        agent {
          label 'host'
        }
        steps {
          stash name: "code", excludes: ".git"
        }  
    }
    stage('Parallel execution') {
      parallel {
        options {
            skipDefaultCheckout()
          }
        stage('Say Hello') {
          steps {
            sh 'echo "Hello, World!"'
          }
        }
        stage('Build app') {
          options {
            skipDefaultCheckout()
          }
          agent {
            docker {
              image 'gradle:jdk11'
            }
          }
          steps {
            unstash "code"
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
          }
          post {
            always {
              // cleanup 
              sh 'ls -lah'
              deleteDir()
              sh 'ls -lah'
            }
          }
        }
        stage('Test') {
          options {
            skipDefaultCheckout()
          }
          agent {
            docker {
              image 'gradle:jdk11'
            }
          }
          steps {
            unstash "code"
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
          post {
            always {
              // cleanup 
              sh 'ls -lah'
              deleteDir()
              sh 'ls -lah'
            }
          }
        }
      }
    }
    stage('docker push') {
      options {
        skipDefaultCheckout()
      }
      environment {
        DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
      }
      steps {
        sh 'echo $DOCKERCREDS_PSW'
        sh 'echo $DOCKERCREDS_USR'
        unstash 'code' //unstash the repository code
        sh 'ci/build-docker.sh'
        sh 'echo $DOCKERCREDS_PSW | docker login -u $DOCKERCREDS_USR --password-stdin' //login to docker hub with the credentials above
        sh 'ci/push-docker.sh'
      }
    }
  }
}