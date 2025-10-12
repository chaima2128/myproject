pipeline {
  agent any
  options { skipDefaultCheckout(true); timestamps() }

  environment {
    SONARQUBE_SERVER = 'sq1'   // le nom EXACT configuré dans System > SonarQube servers
  }

  stages {
    stage('Checkout (Git)') {
      steps {
        checkout([$class: 'GitSCM',
          userRemoteConfigs: [[
            url: 'https://github.com/chaima2128/myproject.git',
            credentialsId: 'github-chaima'
          ]],
          branches: [[name: '*/main']]
        ])
      }
    }

    stage('Clean') {
      steps { sh 'chmod +x mvnw || true && ./mvnw -q clean' }
    }

    stage('Compile') {
      steps { sh './mvnw -q -DskipTests compile' }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv("${env.SONARQUBE_SERVER}") {
          withCredentials([string(credentialsId: 'jenkins-sonar', variable: 'SONAR_TOKEN')]) {
            sh '''
              ./mvnw -q -DskipTests sonar:sonar \
                -Dsonar.projectKey=myproject \
                -Dsonar.host.url=$SONAR_HOST_URL \
                -Dsonar.token=$SONAR_TOKEN
            '''
          }
        }
      }
    }

    stage('Package (JAR)') {
      steps { sh './mvnw -q -DskipTests package' }
      post {
        success { archiveArtifacts artifacts: 'target/*.jar', fingerprint: true }
      }
    }
  }
}
