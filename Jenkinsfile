pipeline {
  agent any
  stages {
    stage('Setup Python Environment') {
      steps {
        sh '''
          ./App/make setup 
          ~/.capstone/bin/activate
          ./App/make install
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

  }
}