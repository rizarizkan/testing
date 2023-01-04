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
          - name: helm
            image: alpine/helm:3.9.3
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
   stage('Get K8s Yaml files') {
     steps {
        checkout([$class: 'GitSCM', 
            branches: [[name: '*/main']], 
            doGenerateSubmoduleConfigurations: false, 
            extensions: [[
                $class: 'RelativeTargetDirectory',
                relativeTargetDir: 'itmi-core']],
            submoduleCfg: [], 
            userRemoteConfigs: [[
                credentialsId: 'github-itmi',
                url: 'https://github.com/rizarizkan/helm-k8s.git']]])
         }
       }
   stage('gpg') {
     steps {
        container(name: 'helm') {
            withCredentials([file(credentialsId: 'gpg', variable: 'itmigpg')]) {
             sh "curl -o /usr/local/bin/sops -s https://github.com/mozilla/sops/releases/download/v3.7.3/sops-v3.7.3.linux.amd64"
             sh "chmod +x /usr/local/bin/sops"
             sh "echo 'http://dl-cdn.alpinelinux.org/alpine/edge/testing' >> /etc/apk/repositories"
             sh "sed -i '/edge/s/^#//' /etc/apk/repositories"
             sh "apk --no-cache add ca-certificates curl"
             sh "apk add --no-cache gnupg"
             sh "cp \$itmigpg gpg-production.asc"
             sh "gpg --import gpg-production.asc"
             sh "helm plugin install https://github.com/jkroepke/helm-secrets.git --version v4.2.0"
        }
      }
    }
  }
   stage('Deploy to Kubernetes') {
     steps {
        container(name: 'helm') {
            dir('itmi-core/itmi-core/') {
             sh "helm secrets upgrade --install core . -f helm_vars/secrets.yaml" 
          }
        }
      }
    }



  }
  
}
