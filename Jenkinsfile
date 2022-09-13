pipeline {
  agent {
    kubernetes {
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
        nodeSelector:
             kops.k8s.io/instancegroup: devtools
        '''
    }
  }
  
  parameters {
        string(name: 'Greeting', defaultValue: 'Hello', description: 'How should I greet the world?')
    }
  
  stages {
    stage('Run maven') {
      steps {
        container('maven') {
          sh 'mvn -version'
        }
        container('busybox') {
          sh '/bin/busybox'

	script {
        def port=echo $((port=$(awk '/patch/{print}' .semver | cut -d " " -f2) +1)) 
        echo "${port}"
        }

        }
      }
    }
  }
}
