podTemplate(containers: [
    containerTemplate(
        name: 'maven', 
        image: 'maven:3.8.1-jdk-8', 
        command: 'sleep', 
        args: '30d'
        ),
    containerTemplate(
        name: 'docker', 
        image: 'docker:stable', 
        command: 'sleep', 
        args: '30d')
  ]) {

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

    node(POD_LABEL) {
        stage('Get a Maven project') {
            git 'https://github.com/spring-projects/spring-petclinic.git'
            container('maven') {
                stage('Build a Maven project') {
                    sh '''
                    echo "maven build"
                    '''
                }
            }
        }

        stage('Get a Docker Project') {
            git url: 'https://github.com/rizarizkan/testing.git', branch: 'main'
            withCredentials(bindings: [usernamePassword(credentialsId: 'github-itmi', passwordVariable: 'GITHUB_COMMON_CREDS_USR', usernameVariable: 'GITHUB_COMMON_CREDS_PSW')]) {
            container('docker') {
                stage('Build a Docker project') {
                dockerImage = docker.build "registry.rizkan.xyz/library/itmi-core" + ":${IMAGE_TAG}"
                }
            }
          }
        }

    }
}
