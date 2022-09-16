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
  
  stages {
    stage('user_input'){
      input{
          message "press ok to continue"
          submitter "kalai,lead"
      parameters{
          string(name:'username1',description: "only kalai and lead has permission")
       }
      }
        steps{
        echo "User : ${username1} said ok"
        }
      }
    }
}
