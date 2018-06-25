pipeline {
  tools {
    maven 'M3' 
  }
  environment {
    BUILD_GIT="https://github.com/renan-miranda/Hygieia.git"
  }
  agent any
  stages {
    stage('Preparation') {
      steps {
        sh "mkdir -p $WORKSPACE/repo;\
           git clone $BUILD_SCRIPTS_GIT repo/$BUILD_SCRIPTS
        "
        sh 'IMAGES=$(docker images |  tail -n +2 | tr -s " " | cut -d " " -f3); \
            [ ! -z ${IMAGES} ] && $(echo ${IMAGES} | xargs docker rmi) || \
            echo "No docker images to remove"
        '
      }
    }
    stage('Build') {
      steps {
        sh "mvn -Dmaven.test.failure.ignore clean install package docker:build"
        sh "OLD_IFS=${IFS}; \
            IFS=$'\n'; \
            for image in $(docker image ls);\
            do \
              NAME=$(echo ${image} | tr -s " " | cut -d " " -f1); TAG=$(echo ${image} | tr -s " " | cut -d " " -f2); \
              HASH=$(echo ${image} | tr -s " " | cut -d " " -f3); [ "${TAG}" == "latest" ] && IMG_NAME="${NAME}" \\
               || IMG_NAME="${NAME}-${TAG}"; \
              docker tag ${HASH} ${repository}/${IMG_NAME}:firsttry; \
              docker push ${repository}/${IMG_NAME}; \
            done; \
            IFS=${OLD_IFS}
        "
      }
    }
  }
  post {
    always {
      cleanWs()
    }
  }
}
