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
           - name: kaniko-secret
             secret:
                secretName: dockercred
                items:
                - key: .dockerconfigjson
                  path: config.json
          containers:
          - name: node
            image: node:latest
            command:
              - cat
            tty: true
            resources:
              requests:
                memory: 128Mi
                cpu: 50m
          - name: kaniko
            image: gcr.io/kaniko-project/executor:debug
            command:
            - sleep
            args:
            - 9999999
            volumeMounts:
            - name: kaniko-secret
              mountPath: /kaniko/.docker
            resources:
              requests:
                memory: 128Mi
                cpu: 50m
          - name: curl
            image: pstauffer/curl
            command:
              - cat
            tty: true
            resources:
              requests:
                memory: 64Mi
                cpu: 50m
          - name: helm
            image: dtzar/helm-kubectl:3.9.3
            imagePullPolicy: Always
            command:
              - cat
            tty: true
            resources:
              requests:
                memory: 128Mi
                cpu: 50m
        '''
    }
  }

  environment {
    GITHUB_COMMON_CREDS = credentials('github-itmi')
    HARBOR_CREDENTIALS = credentials('harbor-registry')
    HARBOR_URL = 'https://dev-registry.itmi.id'
    HARBOR_PROJECT = 'dev-registry.itmi.id/glm'
    NAMESPACE = 'staging'
    BRANCH = 'staging'
    IMAGE_TAG = 'staging'
    RELEASE = 'core'
}

  stages {
    stage('Cloning Git') {
      steps{
          checkout scm
          withCredentials(bindings: [usernamePassword(credentialsId: 'github-itmi', passwordVariable: 'GITHUB_COMMON_CREDS_USR', usernameVariable: 'GITHUB_COMMON_CREDS_PSW')]) {
        }
      }
    }
    stage('Build Image with Kaniko') {
      steps {
       container('kaniko') {
         script{
          sh "/kaniko/executor --dockerfile `pwd`/Dockerfile --context `pwd` --destination ${HARBOR_PROJECT}/itmi-core:${IMAGE_TAG}"
          }
        }
      }
    }
   stage('Get K8s Yaml files') {
     steps {
        checkout([$class: 'GitSCM',
            branches: [[name: '*/master']],
            doGenerateSubmoduleConfigurations: false,
            extensions: [[
                $class: 'RelativeTargetDirectory',
                relativeTargetDir: 'itmi-infra-repository']],
            submoduleCfg: [],
            userRemoteConfigs: [[
                credentialsId: 'github-itmi',
                url: 'https://github.com/itmi-id/itmi-infra.git']]])
         }
       }
   stage('gpg') {
     steps {
        container(name: 'helm') {
            withCredentials([file(credentialsId: 'gpg', variable: 'itmigpg')]) {
             sh "wget https://github.com/mozilla/sops/releases/download/v3.7.3/sops-v3.7.3.linux.amd64"
             sh "cp sops-v3.7.3.linux.amd64 /usr/local/bin/sops"
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
           dir('itmi-infra-repository/helm/itmi-core') {
           sh "helm secrets upgrade --install --set image.tag=${IMAGE_TAG} -n ${NAMESPACE} core . -f helm_vars/secrets-${BRANCH}.yaml"
           sh "kubectl rollout restart -n ${NAMESPACE} deployment ${RELEASE}"
          }
        }
      }
    }



  }

}
