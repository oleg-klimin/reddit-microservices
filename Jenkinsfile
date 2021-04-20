pipeline {  
    environment {
        dockerImageComment = 'zombrox/new_comment'
        dockerImagePost = 'zombrox/new_post_py'
        dockerImageUi = 'zombrox/new_ui'
        shortCommit = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
    }  

  agent any
  
  stages {
    stage('Building images') {
      steps{
        script {
          dir ('./comment') {
              sh 'docker build -t $dockerImageComment:$shortCommit -t $dockerImageComment:latest .'
          }            
          dir ('./post-py') {
              sh 'docker build -t $dockerImagePost:$shortCommit -t $dockerImagePost:latest .'
          }            
          dir ('./ui') {
              sh 'docker build -t $dockerImageUi:$shortCommit -t $dockerImageUi:latest .'
          }            
            }
        }
    }

    stage('Pushing image') {
      steps{
        withDockerRegistry([ credentialsId: "Docker hub credentials", url: "" ]) { 
          sh 'docker push $dockerImageComment'
          sh 'docker push $dockerImagePost'
          sh 'docker push $dockerImageUi'
          }            
            }
        }

    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $dockerImageComment:$shortCommit"
        sh "docker rmi $dockerImagePost:$shortCommit"
        sh "docker image prune -f"
          }
        }

    stage('Deploing to minikube') {
      steps{
        script {
          try {
            sh "kubectl delete deploy --all -n dev"
            sh "kubectl delete svc --all -n dev"

          } catch (err) {
            echo "No web deployment found, continuing"
          }
          sh "kubectl apply -f kubernetes/. -n dev"
            }
           }
        }

  }
}
