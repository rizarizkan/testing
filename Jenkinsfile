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
    
   environment {
              GCHAT_NOTIF = credentials('jenkins-notif-gchat')
              }
    
   stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    
        stage('display') {
            steps {
                echo "BUILD_ID = ${env.BUILD_ID}"
                echo "BUILD_NUMBER = ${env.BUILD_NUMBER}"
                echo "BUILD_TAG = ${env.BUILD_TAG}"
                echo "BUILD_URL = ${env.BUILD_URL}"
                echo "EXECUTOR_NUMBER = ${env.EXECUTOR_NUMBER}"
                echo "JAVA_HOME = ${env.JAVA_HOME}"
                echo "JENKINS_URL = ${env.JENKINS_URL}"
                echo "JOB_NAME = ${env.JOB_NAME}"
                echo "NODE_NAME = ${env.NODE_NAME}"
                echo "WORKSPACE = ${env.WORKSPACE}"
            }
        }
    }
    
    post { 
        success { 
            withCredentials([string(credentialsId: 'jenkins-notif-gchat', variable: 'GCHAT_NOTIF_JENKINS')]) {
            googlechatnotification url: 'https://chat.googleapis.com/v1/spaces/AAAAmk6FRVM/messages?key=AIzaSyDdI0hCZtE6vySjMm-WEfRq3CPzqKqqsHI&token=\$GCHAT_NOTIF_JENKINS', message: '*SUCCESS* Build Job *${JOB_NAME}* - ${BUILD_URL}', notifyAborted: 'true', notifyFailure: 'true', notifyNotBuilt: 'true', notifySuccess: 'true', notifyUnstable: 'true', notifyBackToNormal: 'true', suppressInfoLoggers: 'true', sameThreadNotification: 'true' 
            }
        }
    }
}

