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
               path: /var/run/crio/crio.sock
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
            resources:
              requests:
                memory: 128Mi
                cpu: 50m
          - name: docker
            image: docker:stable
            command:
              - cat
            tty: true
            volumeMounts:
            - name: docker
              mountPath: /var/run/crio/crio.sock
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
    HARBOR_URL = 'dev-registry.itmi.id'
    HARBOR_PROJECT = 'dev-registry.itmi.id/library'
    NAMESPACE = 'devel'
    BRANCH = 'devel'
    IMAGE_TAG = 'devel'
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
  node {
    stage('Build') {
       checkout scm
       docker.withRegistry('${HARBOR_URL}', 'harbor-registry') {
          def customImage = docker.build("my-image:${env.BUILD_ID}")
          customImage.push()
       }
    }
}




  }
  
}
