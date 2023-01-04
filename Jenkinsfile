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
    stage('curl') {
      steps{
        container(name: 'curl') {
          sh 'curl -s -LO "https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl"'  
          sh "chmod +x ./kubectl"  
          sh "./kubectl version"
        }
      }
    }
   stage('Get K8s Yaml files') {
     steps {
        echo 'Getting kubernetes files from git...'
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
            sh "apk add --update gpg"
            sh "apk add --update gpg-agent"
            sh "cp \$itmigpg gpg-production.asc"
            sh "gpg --import gpg-production.asc"
          }
        }
      }
    }
   stage('Deploy to Kubernetes') {
     steps {
        container(name: 'helm') {
            dir('itmi-core/itmi-core') {
            sh "helm plugin install https://github.com/jkroepke/helm-secrets.git --version v4.2.0"
            sh "cp bin/sops /usr/local/bin/"
            sh "helm secrets upgrade --install core . -f helm_vars/secrets.yaml" 
          }
        }
      }
    }



  }
//     post {
//        always {
//          container(name: 'helm') {
//            dir('itmi-core') {
//              sh "helm list"
              //sh "helm upgrade --install core . -n default -f values.yml"
//             }
//          }
//       }
//    }
  
}
