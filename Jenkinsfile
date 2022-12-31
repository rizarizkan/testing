pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            some-label: some-label-value
        spec:
          volumes:
           - name: docker
             hostPath:
               path: /var/run/docker.sock
               type: Socket
           - name: jenkins-slave
             hostPath:
               path: /var/lib/bundle
               type: DirectoryOrCreate
          containers:
          - name: node
            image: node:latest
            command:
              - cat
            tty: true
            volumeMounts:
              - name: jenkins-slave
                mountPath: /var/lib/bundle
          - name: docker
            image: docker:stable
            command:
              - cat
            tty: true
            volumeMounts:
              - name: docker
                mountPath: /var/run/docker.sock
          - name: kubectl
            image: bitnami/kubectl
            command:
              - cat
            tty: true
            volumeMounts:
              - name: jenkins-slave
                mountPath: /var/lib/bundle
        '''
    }
  }
  
  environment {
    GITHUB_COMMON_CREDS = credentials('github-itmi')
    HARBOR_CREDENTIALS = credentials('harbor-registry')
}
  
  stages {
    stage('Cloning Git') {
      steps{
          checkout scm
          withCredentials(bindings: [usernamePassword(credentialsId: 'github-itmi', passwordVariable: 'GITHUB_COMMON_CREDS_USR', usernameVariable: 'GITHUB_COMMON_CREDS_PSW')]) {
        }
      }
    }
    stage('Build Image') {
      steps{
        container('docker') {
          script{
            dockerImage = docker.build "registry.rizkan.xyz/glm/itmi-core" + ":$BUILD_NUMBER"
             }
           }  
         }
       }
    stage('Deploy Image'){
      steps{
        container(name: 'docker') {
          script {
            withDockerRegistry(registry: [url: 'https://registry.rizkan.xyz', credentialsId: 'harbor-registry']) {
              dockerImage.push()
                }
              }
            }
          }
        }
    stage('Remove Unused docker image') {
      steps{
        container(name: 'docker') {
          sh "docker rmi registry.rizkan.xyz/glm/itmi-core" + ":$BUILD_NUMBER"
          }
        }
      }
    stage('kubectl') {
      steps{
        container(name: 'kubectl') {
          sh "ls"
          sh "cat /etc/issue"
          sh "df -h"
          sh "kubectl --version"
        }
      }
    }
    stage('default') {
      steps{
          sh "ls"
          sh "cat /etc/issue"
          sh "df -h"
      }
    }

  }
  
}
