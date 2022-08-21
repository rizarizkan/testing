def username = 'Jenkins'
pipeline {
agent {
    kubernetes {
      label 'k8s'
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
    docker {
        image 'maven:3.8.1-adoptopenjdk-11'
        label 'my-defined-label'
        args  '-v /tmp:/tmp'
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
                echo "JOB_URL = ${env.JOB_URL}"
                echo "EXECUTOR_NUMBER = ${env.EXECUTOR_NUMBER}"
                echo "JAVA_HOME = ${env.JAVA_HOME}"
                echo "JENKINS_URL = ${env.JENKINS_URL}"
                echo "JOB_NAME = ${env.JOB_NAME}"
                echo "JOB_BASE_NAME = ${env.JOB_BASE_NAME}"
                echo "NODE_NAME = ${env.NODE_NAME}"
                echo "WORKSPACE = ${env.WORKSPACE}"
                echo "BRANCH_NAME = ${env.BRANCH_NAME}"
                echo "BRANCH_IS_PRIMARY = ${env.BRANCH_IS_PRIMARY}"
                echo "CHANGE_ID = ${env.CHANGE_ID}"
                echo "CHANGE_URL = ${env.CHANGE_URL}"
                echo "CHANGE_TITLE = ${env.CHANGE_TITLE}"
                echo "CHANGE_AUTHOR = ${env.CHANGE_AUTHOR}"
                echo "CHANGE_AUTHOR_DISPLAY_NAME = ${env.CHANGE_AUTHOR_DISPLAY_NAME}"
                echo "CHANGE_AUTHOR_EMAIL = ${env.CHANGE_AUTHOR_EMAIL}"
                echo "CHANGE_TARGET = ${env.CHANGE_TARGET}"
                echo "CHANGE_BRANCH = ${env.CHANGE_BRANCH}"
                echo "CHANGE_FORK = ${env.CHANGE_FORK}"
                echo "TAG_NAME = ${env.TAG_NAME}"
                echo "TAG_TIMESTAMP = ${env.TAG_TIMESTAMP}"
                echo "TAG_UNIXTIME = ${env.TAG_UNIXTIME}"
                echo "TAG_DATE = ${env.TAG_DATE}"
                echo "JOB_DISPLAY_URL = ${env.JOB_DISPLAY_URL}"
                echo "RUN_DISPLAY_URL = ${env.RUN_DISPLAY_URL}"
                echo "RUN_ARTIFACTS_DISPLAY_URL = ${env.RUN_ARTIFACTS_DISPLAY_URL}"
                echo "RUN_CHANGES_DISPLAY_URL = ${env.RUN_CHANGES_DISPLAY_URL}"
                echo "RUN_TESTS_DISPLAY_URL = ${env.RUN_TESTS_DISPLAY_URL}"
                echo "CI = ${env.CI}"
                echo "BUILD_DISPLAY_NAME = ${env.BUILD_DISPLAY_NAME}"
                echo "BUILD_TAG = ${env.BUILD_TAG}"
                echo "EXECUTOR_NUMBER = ${env.EXECUTOR_NUMBER}"
                echo "NODE_NAME = ${env.NODE_NAME}"
                echo "NODE_LABELS = ${env.NODE_LABELS}"
                echo "GIT_COMMIT = ${env.GIT_COMMIT}"
                echo "GIT_PREVIOUS_COMMIT = ${env.GIT_PREVIOUS_COMMIT}"
                echo "GIT_BRANCH = ${env.GIT_BRANCH}"
                echo "GIT_URL = ${env.GIT_URL}"
                echo "GIT_AUTHOR_NAME = ${env.GIT_AUTHOR_NAME}"
                echo "GIT_AUTHOR_EMAIL = ${env.GIT_AUTHOR_EMAIL}"

                sh "exit 1"
            }
        }
   }
         
      // post { 
      //       failure {
      //          googlechatnotification url: 'id:jenkins-notif-gchat', message: '*FAILED* Build Job *${JOB_NAME}* - ${BUILD_URL}', notifyAborted: 'true', notifyFailure: 'true', notifyNotBuilt: 'true', notifySuccess: 'true', notifyUnstable: 'true', notifyBackToNormal: 'true', suppressInfoLoggers: 'true', sameThreadNotification: 'true' 
     // } 
    //}
  }

