def username = 'Jenkins'
pipeline {
agent {
    kubernetes {
      activeDeadlineSeconds 60
      nodeSelector 'beta.kubernetes.io/os=linux,devtools=true'
      yaml'''
      spec:
        tolerations:
          - key: devtools
            operator: Exists
            effect: NoSchedule
        serviceAccountName: helm
        volumes:
          - name: docker
            hostPath:
              path: /var/run/docker.sock
              type: Socket

          - name: jenkins-slave
            hostPath:
              path: /usr/lib/bundle
              type: DirectoryOrCreate
        imagePullSecrets:
          - name: depozitare
          - name: prtsdev
        containers:
          - name: helm
            image: depozitare.printerous.com/helm
            imagePullPolicy: IfNotPresent
            command:
              - cat
            tty: true

          - name: docker
            image: docker:stable
            imagePullPolicy: IfNotPresent
            command:
              - cat
            tty: true
            volumeMounts:
              - name: docker
                mountPath: /var/run/docker.sock

          - name: ruby
            image: depozitare.printerous.com/ruby:2.7.2
            imagePullPolicy: IfNotPresent
            command:
              - cat
            tty: true
            volumeMounts:
              - name: jenkins-slave
                mountPath: /usr/lib/bundle
      '''
    }
  }
    
    
    stages {
        stage('Checkout Source') {
             steps {
                 git url:'https://github.com/rizarizkan/testing.git', branch:'main'
           }   
        }   
        
         stage('Deploy') {
            when {
              expression {
                currentBuild.result == null || currentBuild.result == 'SUCCESS' 
              }
            }
            steps {
                sh 'echo "sukses oi"'
                echo 'Hello Mr. ${username}'
                echo "I said, Hello Mr. ${username}"
                echo 'Hello World'
                    
                post { 
                 success { 
                    googlechatnotification url: 'https://chat.googleapis.com/v1/spaces/AAAAmk6FRVM/messages?key=AIzaSyDdI0hCZtE6vySjMm-WEfRq3CPzqKqqsHI&token=lLyEB-ioqic3E2Q8ftdAdpkaqbMGO_yqsIBW2dXrQ5U%3D', message: 'message to be sent', notifyAborted: 'false', notifyFailure: 'false', notifyNotBuilt: 'false', notifySuccess: 'true', notifyUnstable: 'false', notifyBackToNormal: 'false', suppressInfoLoggers: 'false', sameThreadNotification: 'false'
                }
                }     
            }
        }
    }
}
