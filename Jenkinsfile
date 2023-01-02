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
          - name: curl
            image: pstauffer/curl
            command:
              - cat
            tty: true
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
            dockerImage = docker.build "registry.rizkan.xyz/glm/itmi-core" + ":develop"
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
          sh "docker rmi registry.rizkan.xyz/glm/itmi-core" + ":develop"
          }
        }
      }
    stage('curl') {
      steps{
        container(name: 'curl') {
          sh 'curl -s -LO "https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl"'  
          sh "chmod +x ./kubectl"  
          sh "./kubectl version"
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        container('helm') {
          git changelog: false, credentialsId: 'github-itmi', poll: false, url: 'https://github.com/rizarizkan/helm-k8s.git'
          dir('itmi-core')
          sh "helm upgrade --wait --timeout 15m0s --install core . -n default -f values.yml"
        }
      }
    }

  }
  
}
