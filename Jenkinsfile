pipeline {
  environment {
    registry = "645851037944.dkr.ecr.us-west-2.amazonaws.com/cdoe-capstone-proj"
    registryCredential = 'ecr:us-west-2:aws-ecr-creds'
  }
  agent any
  stages {
    stage('Setup Python Environment') {
      steps {
        sh '''
          cd ./App/
          pwd && ls -lamo
          make setup 
          . ~/.capstone/bin/activate
          make install
        '''
      }
    }
    stage('Lint Python App') {
      steps {
        sh '''
          . ~/.capstone/bin/activate 
          pylint --disable=R,C,W1203,W1309 ./App/app.py
        '''  
      }
    }
    stage('Lint HTML App') {
      steps {
        sh 'tidy -q -e ./App/templates/*.html'
      }
    }
    stage('Build Docker Image'){
      steps{
        script{
          docker.withRegistry("https://" + registry, registryCredential) {
            def capstoneImage = docker.build registry + ":${env.BUILD_ID}"
            capstoneImage.push()
            capstoneImage.push('latest')
          }
        }
      }
    }
    stage('Deploy Image To K8s') {
      steps{
        withAWS(credentials: 'aws-ecr-creds', region: 'us-west-2') {
          sh "aws eks --region us-west-2 update-kubeconfig --name basic-cluster"
          sh "kubectl get deployments"
          sh "kubectl get pods"
          sh "kubectl set image deployments/capstone-web cdoe-capstone-proj=645851037944.dkr.ecr.us-west-2.amazonaws.com/cdoe-capstone-proj:${env.BUILD_ID}"
        }
        /*
        steps{    
          sh """
            kubectl --kubeconfig /home/ubuntu/.kube/config set image deployments/capstone-web cdoe-capstone-proj=645851037944.dkr.ecr.us-west-2.amazonaws.com/cdoe-capstone-proj:${env.BUILD_ID}
          """
        }
        */
      }
    }
  }
}