 pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }
  environment {
    DOCKER_CREDS = credentials('amazeeiojenkins-dockerhub-password')
    COMPOSE_PROJECT_NAME = "laravel-distro-${BUILD_ID}"
  }
  stages {
    stage('Docker login') {
      steps {
        sh '''
        docker login --username amazeeiojenkins --password $DOCKER_CREDS
        '''
      }
    }
    stage('Docker Build') {
      steps {
        sh '''
        docker network create amazeeio-network || true
        docker-compose config -q
        docker-compose down
        docker-compose up -d --build "$@"
        '''
      }
    }
    stage('Waiting') {
      steps {
        sh """
        sleep 5s
        """
      }
    }
    stage('Verification') {
      steps {
        sh '''
        docker-compose exec -T cli drush status
        docker-compose exec -T cli curl http://nginx:8080 -v
        if [ $? -eq 0 ]; then
          echo "OK!"
        else
          echo "FAIL"
          /bin/false
        fi
        docker-compose down
        '''
      }
    }
    stage('Docker Push') {
      steps {
        sh '''
        echo "Branch: $GIT_BRANCH"
        docker images | head

        for variant in ''; do
            echo docker tag laravel-distro$variant amazeeiodevelopment/laravel-distro$variant:$GIT_BRANCH
            echo docker push amazeeiodevelopment/laravel-distro$variant:$GIT_BRANCH

            if [ $GIT_BRANCH = "develop" ]; then
              echo docker tag laravel-distro$variant amazeeiodevelopment/laravel-distro$variant:latest
              echo docker push amazeeiodevelopment/laravel-distro$variant:latest
            fi

        done
        '''
      }
    }
  }
}
