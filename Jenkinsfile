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
        '''
    }
  }
  
  environment {
    GITHUB_COMMON_CREDS = credentials('github-itmi')
    HARBOR_CREDENTIALS = credentials('harbor-registry')
}
  
  stages {
    stage('Cloning Git') {
      steps {
          checkout scm
          withCredentials(bindings: [usernamePassword(credentialsId: 'github-itmi', passwordVariable: 'GITHUB_COMMON_CREDS_USR', usernameVariable: 'GITHUB_COMMON_CREDS_PSW')]) {
        }
      }
    }
    stage('Run node') {
      steps {
        container('node') {
          sh 'node -v'
         }
        container(name: 'docker') {
            script {
            //docker.withRegistry('https://registry.rizkan.xyz', 'harbor-registry') {
            docker.withRegistry('https://registry.rizkan.xyz') {
            //withDockerRegistry(registry: [url: 'https://registry.rizkan.xyz', credentialsId: 'harbor-registry']) {
            def customImage = docker.build("itmi-core:${env.BUILD_ID}")
            //dockerImage = docker.build "registry.rizkan.xyz/glm/itmi-core" + ":staging"
            dockerImage.push()
                  }
                }
              }
      }
    }
  }
  

}
