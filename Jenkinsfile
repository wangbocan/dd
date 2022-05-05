pipeline { 
    agent { label 'jnlp-slave'}
    stages { 
        stage('Clone') { 
            steps{
          echo "1.Clone Stage" 
          git url: "https://github.com/wangbocan/dd.git" 
          script { 
              build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim() 
        } 
    } 
}
 stage('Test') { 
     steps {
      echo "2.Test Stage" 
    }
 }
 stage('Build') { 
     steps{
       echo "3.Build Docker Image Stage" 
       sh "docker build -t 192.168.92.133/test/jenkins-demo:${build_tag} ." 
   } 
 }
 stage('Push') { 
     steps {
      echo "4.Push Docker Image Stage" 
      withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 
     'dockerHubPassword', usernameVariable: 'dockerHubUser')]) { 
      sh """
      echo "wangcan!123" |  docker login --username ${dockerHubUser} --password-stdin ${dockerHubPassword} 192.168.92.133
      docker push 192.168.92.133/test/jenkins-demo:${build_tag}
      """
        } 
    } 
 }
 stage('Deploy to dev') { 
     steps {
       echo "5. Deploy DEV" 
       sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s-dev.yaml" 
       sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s-dev.yaml" 
       sh "kubectl apply -f k8s-dev.yaml --validate=false" 
       }
     } 
 stage('Promote to qa') { 
     steps {
         script {
       def userInput = input( id: 'userInput', message: 'Promote to qa?', parameters: [[ $class: 'ChoiceParameterDefinition', choices: "YES\nNO", name: 'Env' ]]) 
       echo "This is a deploy step to ${userInput}" 
      if (userInput == "YES") { 
      sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s-qa.yaml" 
      sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s-qa.yaml" 
      sh "kubectl apply -f k8s-qa.yaml --validate=false" 
      sh "sleep 6" 
      sh "kubectl get pods -n qatest" 
      } else { 
   } 
         }
 }
 }
 stage('Promote to pro') { 
     steps {
         script {
       def userInput = input(id: 'userInput',message: 'Promote to pro?',parameters: [[$class: 'ChoiceParameterDefinition',choices: "YES\nNO",name: 'Env']]) 
         
       echo "This is a deploy step to ${userInput}" 
       if (userInput == "YES") { 
       sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s-prod.yaml" 
       sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s-prod.yaml" 
       sh "cat k8s-prod.yaml" 
       sh "kubectl apply -f k8s-prod.yaml --record --validate=false" 
      } 
    }
  } 
 }
}
}
