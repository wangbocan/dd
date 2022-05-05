pipeline {
  agent {
    node {
      label 'maven'
    }
  }
 
  parameters {
      string(name:'TAG_NAME',defaultValue: 'latest',description:'')
  }
 
  environment {
      DOCKER_CREDENTIAL_ID = 'dockerhub-id'
      GITHUB_CREDENTIAL_ID = 'gitlab-id'
      KUBECONFIG_CREDENTIAL_ID = 'iot-kubeconfig'
      REGISTRY = 'www.xxx.com'
      DOCKERHUB_NAMESPACE = 'dev'
      REGISTRY_USERNAME = 'user'
      REGISTRY_PASSWORD = 'pass'
      GITHUB_ACCOUNT = 'account'
      APP_NAME = 'devops-sample'
      imageName = 'app1'
      dockerFile = 'src/app1/Dockerfile'
      ImageTag = 'latest'
      localFullImageName = 'app1:latest'
      remoteImage = 'www.xxxx.com/dev/app1'
  }
 
  stages {
    stage('拉代码') {
      steps {
        git(url: 'http://gitlab.xxxxx.com.cn/app1.git', credentialsId: 'gitlab-id', branch: 'develop', changelog: true, poll: false)
      }
    }
    stage('构建镜像') {
      steps {
        container ('maven') {
          sh 'docker build -f $dockerFile -t $imageName:$ImageTag ./src/'
          withCredentials([usernamePassword(passwordVariable : 'REGISTRY_PASSWORD' ,usernameVariable : 'REGISTRY_USERNAME' ,credentialsId : "$DOCKER_CREDENTIAL_ID" ,)]) {
            sh 'echo "$REGISTRY_PASSWORD" | docker login $REGISTRY -u "$REGISTRY_USERNAME" --password-stdin'
          }
          
        }
          
      }
    }
   
    stage('推送镜像') {
      steps {
        container ('maven') {
          sh 'docker tag $localFullImageName $remoteImage:latest '
          sh 'docker push $remoteImage:latest '
        }
      }
    }
    
    stage('deploy to dev') {
      when{
        branch 'develop'
      }
      steps {
        input(id: 'deploy-to-dev', message: 'deploy to dev?')
        kubernetesDeploy(configs: 'deploy/dev-ol/**', enableConfigSubstitution: true, kubeconfigId: "$KUBECONFIG_CREDENTIAL_ID")
      }
    }
  }
}
