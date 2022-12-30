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
          - name: maven
            image: maven:alpine
            command:
            - cat
            tty: true
          - name: nginx
            image: nginx:latest
            command:
            - cat
            tty: true
          - name: ruby
            image: ruby:latest
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
        '''
    }
  }
  
  environment {
    GITHUB_COMMON_CREDS = credentials('github-itmi')
}
  
  stages {
    stage('Cloning Git') {
      steps {
            checkout scm
        //withCredentials(bindings: [usernamePassword(credentialsId: 'github-itmi', passwordVariable: 'GITHUB_COMMON_CREDS_USR', usernameVariable: 'GITHUB_COMMON_CREDS_PSW')]) {
        //git 'https://github.com/itmi-id/itmi-infra.git'
        //}
      }
    }
    stage('Run maven') {
      steps {
        container('maven') {
          sh 'mvn -version'
        }
        container('nginx') {
          sh 'nginx -v'
        }
        container(name: 'docker') {
            script {
            dockerImage = docker.build "registry.rizkan.xyz/itmi-core" + ":staging"
            //docker.withRegistry('https://registry.rizkan.xyz', 'docker-registry') {
            withDockerRegistry(registry: [url: 'https://registry.rizkan.xyz/', credentialsId: 'docker-registry']) {
            dockerImage.push()
                  }
                }
              }
      }
    }
  }
  

}
