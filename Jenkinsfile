pipeline {
  agent {
    node {
      label 'maven'
    }
  }
  environment {
        DOCKER_CREDENTIAL_ID = 'dockerhub-id'
        GITHUB_CREDENTIAL_ID = 'github-id'
        KUBECONFIG_CREDENTIAL_ID = 'demo-kubeconfig'
        REGISTRY = 'docker.io'
        DOCKERHUB_NAMESPACE = 'heimao0307'
        GITHUB_ACCOUNT = 'heimao0307'
        APP_NAME = 'devops-java-sample'
  }
  stages {
    stage('拉代码') {
      steps {
        git(url: 'https://github.com/heimao0307/demo-java.git', credentialsId: 'github-id', branch: 'master', changelog: true, poll: false)
      }
    }
    stage('打包') {
      steps {
        container('maven') {
          sh 'mvn clean package'
        }
      }
    }
    stage('构建Docker镜像') {
      steps {
        container('maven') {
          sh 'docker build -t $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER .'
        }
      }
    }
    stage('推送Docker镜像') {
      steps {
        container('maven') {
          echo 'hello'
          withCredentials([usernamePassword(credentialsId : 'dockerhub-id' ,passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,)]) {
            sh 'echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
            sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER'
          }
        }
      }
    }
    stage('推送 latest') {
      steps {
        container('maven') {
          sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
          sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
        }
      }
    }
    stage('部署到测试环境') {
      steps {
        input(id: 'deploy-to-dev', message: '确定要部署到测试环境吗?')
        kubernetesDeploy(configs: 'deploy/dev/**', enableConfigSubstitution: true, kubeconfigId: '"$KUBECONFIG_CREDENTIAL_ID"', deleteResource: false)
      }
    }
  }
}


