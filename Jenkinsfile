pipeline {
    agent any
    environment {
  // Specify your environment variables.
  APP_VERSION = '1'
  // This can be nexus3 or nexus2
  NEXUS_VERSION = "nexus3"
  // This can be http or https
  NEXUS_PROTOCOL = "http"
  // Where your Nexus is running. In my case:
  NEXUS_URL = "192.168.0.30:8081"
  // Repository where we will upload the artifact
  NEXUS_REPOSITORY = "maven-snapshots"
  // Jenkins credential id to authenticate to Nexus OSS
  NEXUS_CREDENTIAL_ID = "nexus-credentials"
  /*
    Windows: set the ip address of docker host. In my case 192.168.99.100.
    to obtains this address : $ docker-machine ip
    Linux: set localhost to SONARQUBE_URL
  */
  SONARQUBE_URL = "http://192.168.0.30"
  SONARQUBE_PORT = "9000"
    }
    stages {
        stage('Build') {
            steps {
                // Print all the environment variables.
                sh 'printenv'
                sh 'echo $GIT_BRANCH'
                sh 'echo $GIT_COMMIT'
                echo 'Install non-dev composer packages and test a symfony cache clear'
                sh 'docker-compose -f build.yml up --exit-code-from fpm_build --remove-orphans fpm_build'
                echo 'Building the docker images with the current git commit'
                sh 'docker build -f Dockerfile-php-production -t $NEXUS_URL/symfony_project_fpm:$GIT_COMMIT .'
                sh 'docker build -f Dockerfile-nginx -t $NEXUS_URL/symfony_project_nginx:$GIT_COMMIT .'
                sh 'docker build -f Dockerfile-db -t $NEXUS_URL/symfony_project_db:$GIT_COMMIT .'
            }
        }
        stage('Test') {
            steps {
                echo 'PHP Unit tests'
                sh 'docker-compose -f test.yml up -d --build --remove-orphans'
                sh 'sleep 5'
                sh 'docker-compose -f test.yml exec -T fpm_test bash build/php_unit.sh'
            }
        }
        stage('Push') {
            when {
                branch 'master'
            }
            steps {
                echo 'Deploying docker images'
                sh 'docker tag $NEXUS_URL/symfony_project_fpm:$GIT_COMMIT $NEXUS_URL/symfony_project_fpm:$APP_VERSION'
                sh 'docker tag $NEXUS_URL/symfony_project_fpm:$GIT_COMMIT $NEXUS_URL/symfony_project_fpm:latest'
                sh 'docker push $NEXUS_URL/symfony_project_fpm:$APP_VERSION'
                sh 'docker push $NEXUS_URL/symfony_project_fpm:latest'
                sh 'docker tag $NEXUS_URL/symfony_project_nginx:$GIT_COMMIT $NEXUS_URL/symfony_project_nginx:$APP_VERSION'
                sh 'docker tag $NEXUS_URL/symfony_project_nginx:$GIT_COMMIT $NEXUS_URL/symfony_project_nginx:latest'
                sh 'docker push $NEXUS_URL/symfony_project_nginx:$APP_VERSION'
                sh 'docker push $NEXUS_URL/symfony_project_nginx:latest'
                sh 'docker tag $NEXUS_URL/symfony_project_db:$GIT_COMMIT $NEXUS_URL/symfony_project_db:$APP_VERSION'
                sh 'docker tag $NEXUS_URL/symfony_project_db:$GIT_COMMIT $NEXUS_URL/symfony_project_db:latest'
                sh 'docker push $NEXUS_URL/symfony_project_db:$APP_VERSION'
                sh 'docker push $NEXUS_URL/symfony_project_db:latest'
            }
        }
    }
    post {
        always {
            // Always cleanup after the build.
            sh 'docker-compose -f build.yml down'
            sh 'docker-compose -f test.yml down'
            sh 'rm .env'
        }
    }
}
