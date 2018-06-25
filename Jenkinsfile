pipeline {
   agent any

   def mvnHome
   stages{
    stage('Preparation') { 
      steps{ // for display purposes
        // Get some code from a GitHub repository
        git 'https://github.com/renan-miranda/Hygieia.git'
        // Get the Maven tool.
        // ** NOTE: This 'M3' Maven tool must be configured
        // **       in the global configuration.           
        mvnHome = tool 'M3'
        sh '''IMAGES=$(docker images |  tail -n +2 | tr -s " " | cut -d " " -f3)
          [ ! -z ${IMAGES} ] && $(echo ${IMAGES} | xargs docker rmi) || echo "No docker images to remove"'''
        }
     }
     stage('Build') {
       // Run the maven build
       sh '''
         '${mvnHome}'/bin/mvn -Dmaven.test.failure.ignore clean install package docker:build
         OLD_IFS=${IFS}
         IFS=$\'\\n\'
         for image in $(docker image ls)
         do
           NAME=$(echo ${image} | tr -s " " | cut -d " " -f1); TAG=$(echo ${image} | tr -s " " | cut -d " " -f2)
           HASH=$(echo ${image} | tr -s " " | cut -d " " -f3); [ "${TAG}" == "latest" ] && IMG_NAME="${NAME}" \\
             || IMG_NAME="${NAME}-${TAG}"
           docker tag ${HASH} ${repository}/${IMG_NAME}:firsttry
           docker push ${repository}/${IMG_NAME}
         done
         IFS=${OLD_IFS}
       '''
     }
  }
}
