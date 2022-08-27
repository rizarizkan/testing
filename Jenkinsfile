pipeline {
  agent {
    kubernetes {
      nodeSelector:
             kops.k8s.io/instancegroup: devtools
      yaml '''
        spec:    
          containers:
          - name: maven
            image: maven:alpine
            command:
            - cat
            tty: true
          - name: busybox
            image: busybox
            command:
            - cat
            tty: true
        '''
    }
  }
  stages {
    stage('Run maven') {
      steps {
        container('maven') {
          sh 'mvn -version'
        }
        container('busybox') {
          sh '/bin/busybox'
        }
      }
    }
  }
}
