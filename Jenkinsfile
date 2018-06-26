pipeline {
  tools {
    maven 'M3' 
  }
  environment {
    BUILD_GIT="https://github.com/renan-miranda/Hygieia.git"
    BUILD_DIR="Hygieia"
    AWS_ENV_FILE=".aws_hosts.env"
    DOCKER_USER=".docker.user"
    ID_PUB_KEY="${HOME}/.ssh/id_rsa_jenkins"
    SSH_USER="ubuntu"
    DB_DIR="db"
    COMPOSE_FILES="${WORKSPACE}/${BUILD_DIR}/docker-compose.*"
  }
  agent any
  stages {
    stage('Preparation') {
      steps {
        sh "mkdir -p $WORKSPACE; git clone $BUILD_GIT"
        sh 'docker images | tail -n +2 | tr -s " " | cut -d " " -f3 | xargs -r docker image remove -f'
      }
    }
    stage('Build') {
      steps {
        sh "cd $BUILD_DIR; mvn -Dmaven.test.failure.ignore clean install package docker:build"
        sh '. ${HOME}/${DOCKER_USER}; \
            echo ${HUB_PASS} | docker login -u ${HUB_USER} --password-stdin; \
            OLD_IFS=${IFS}; \
            IFS=$\'\n\'; \
            for image in $(docker image ls | tail -n +2 | tr -s " " | cut -d " " -f1-3);\
            do \
              NAME=$(echo ${image} | tr -s " " | cut -d " " -f1);\
              TAG=$(echo ${image} | tr -s " " | cut -d " " -f2); \
              HASH=$(echo ${image} | tr -s " " | cut -d " " -f3); \
              docker tag ${HASH} ${HUB_USER}/${NAME}:${TAG}; \
              docker push ${HUB_USER}/${NAME}:${TAG}; \
            done; \
            IFS=${OLD_IFS}'
      }
    }
    stage('Deploy') {
      steps {
        sh '. ${HOME}/${AWS_ENV_FILE}; \
            . ${HOME}/${DOCKER_USER}; \
            ssh -i "${ID_PUB_KEY}" ${SSH_USER}@${HYGIEIA_PUBLIC_IP} "mkdir ${BUILD_DIR}"; \
            scp -i "${ID_PUB_KEY}" -r ${WORKSPACE}/${BUILD_DIR}/${DB_DIR} ${SSH_USER}@${HYGIEIA_PUBLIC_IP}:~/${BUILD_DIR}; \
            scp -i "${ID_PUB_KEY}" ${COMPOSE_FILES} ${SSH_USER}@${HYGIEIA_PUBLIC_IP}:~/${BUILD_DIR}; \
            ssh -i "${ID_PUB_KEY}" ${SSH_USER}@${HYGIEIA_PUBLIC_IP} \
              "docker ps -a | grep Up | cut -d " " -f1 | xargs -r docker container stop;"; \
            ssh -i "${ID_PUB_KEY}" ${SSH_USER}@${HYGIEIA_PUBLIC_IP} \
              "docker ps -a | grep Exited | cut -d " " -f1 | xargs -r docker container rm" \
            ssh -i "${ID_PUB_KEY}" ${SSH_USER}@${HYGIEIA_PUBLIC_IP} \
              "docker images | tail -n +2 | tr -s " " | cut -d " " -f3 | xargs -r docker image remove -f"; \
            ssh -i "${ID_PUB_KEY}" ${SSH_USER}@${HYGIEIA_PUBLIC_IP} \
               "cd ${BUILD_DIR}; echo ${HUB_PASS} | docker login -u ${HUB_USER} --password-stdin; docker-compose up -d"; \
            ssh -i "${ID_PUB_KEY}" ${SSH_USER}@${HYGIEIA_PUBLIC_IP} "rm -Rf ${BUILD_DIR}"'
      }
    }
  }
  post {
    always {
      cleanWs()
    }
  }
}
