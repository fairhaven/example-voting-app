pipeline{
  agent none

    stages{
        stage('worker-build'){
            agent {
              docker{
                image 'maven:3.6.1-jdk-8-alpine'
                args '-v $HOME/.m2:/root/.m2'
              }
            }
            when{
                changeset "**/worker/**"
            }
            steps{
                echo 'Compiling application'
                dir('worker'){
                  sh 'mvn compile'
                }
            }
        }
        stage('worker-test'){
            agent {
              docker{
                image 'maven:3.6.1-jdk-8-alpine'
                args '-v $HOME/.m2:/root/.m2'
              }
            }
            when{
              changeset "**/worker/**"
            }
            steps{
                echo 'test application'
                dir('worker'){
                  sh 'mvn clean test'
                }

            }
        }
        stage('worker-package'){
            agent {
              docker{
                image 'maven:3.6.1-jdk-8-alpine'
                args '-v $HOME/.m2:/root/.m2'
              }
            }
            when{
              changeset "**/worker/**"
            }
            steps{
                echo 'worker-package application'
                dir('worker'){
                  sh 'mvn package -DskipTest'
                }

            }
        }
        stage('vote-build'){
           agent {
             docker {
               image 'python:3.9.15-alpine'
               args '--user root'
             }
           }
           when{
               changeset "**/vote/**"
           }
           steps{
              echo 'Compiling vote application'
              dir('vote'){
                sh 'pip install -r requirements.txt'
              }
           }
        }
        stage('vote-test'){
          agent {
           docker {
             image 'python:3.9.15-alpine'
             args '--user root'
           }
          }
          when{
               changeset "**/vote/**"
          }
          steps{
               echo 'test vote application'
               dir('vote'){
                sh 'pip install -r requirements.txt'
                sh 'nosetests -v'
               }
          }
        }
        stage('vote-package'){
             agent none
             when{
               changeset "**/vote/**"
               branch 'master'
             }
             steps{
                 echo 'package vote application with docker'
                 script{
                   docker.WithRegustry('https://index.docker.io/v1/', 'dockerlogin'){
                      def workerImage = docker.build("seyibright/vote:v${env.BUILD_ID}", "./vote")
                      workerImage.push()
                      workerImage.push("${env.BRANCH_NAME}")
                   }
                 }
             }
         }
         stage('result-build'){
          agent {
            docker {
              image 'node:8.16.0-alpine'
            }
          }
          when{
            changeset "**/result/**"
          }
          steps{
              echo 'Compiling result application'
              dir('result'){
                sh 'npm install'
              }
          }
        }
        stage('result-test'){
            agent {
              docker {
                image 'node:8.16.0-alpine'
              }
            }
            when{
              changeset "**/result/**"
            }
            steps{
                echo 'test application'
                dir('result'){
                  sh 'npm install'
                  sh 'npm test'
                }
            }
        }
        stage('result-package'){
            agent none
            when{
              changeset "**/result/**"
              branch 'master'
            }
            steps{
                echo 'package result application with docker'
                script{
                  docker.WithRegustry('https://index.docker.io/v1/', 'dockerlogin'){
                     def workerImage = docker.build("seyibright/result:v${env.BUILD_ID}", "./result")
                     workerImage.push()
                     workerImage.push("${env.BRANCH_NAME}")
                  }
                }
            }
        }
    }

    post{
      always{
         archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
         echo 'Build pipeline application done, try again for update'

      }
      failure{
        slackSend (channel: "jenkin-pipeline", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
      }
      success{
        slackSend (channel: "jenkin-pipeline", message: "Build Successful - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
      }
    }

}
