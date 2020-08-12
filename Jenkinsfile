pipeline {
  agent any
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
        stage('Say Hello') {
          steps {
            sh 'echo "Hello, World!"'
          }
        }
        stage('Build app') {
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
      agent any
      environment {
        DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
      }
      steps {
            unstash 'code' //unstash the repository code
            sh 'ci/build-docker.sh'
            sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
            sh 'ci/push-docker.sh'
      }
    }
  }
}