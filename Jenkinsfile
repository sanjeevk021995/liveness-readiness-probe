pipeline {

agent {
    kubernetes {
        yamlFile 'kubernetes-pod-template.yaml'

    }
}


environment {

    DOCKER_REPO = "docker.io/sanjeevk95"
    IMAGE_NAME = "springboot-k8s-probe-demo"
    IMAGE_TAG = "${BUILD_NUMBER}"

  }
  stages {

    stage('Maven') {
      steps {
        container('maven') {
          sh 'mvn clean package -DskipTests'
        }
      }
    }


    stage('kaniko') {
      steps {
        container('kaniko') {
          sh '''
          /kaniko/executor \
          --context $WORKSPACE \
          --dockerfile $WORKSPACE/Dockerfile \
          --destination=$DOCKER_REPO/$IMAGE_NAME:$IMAGE_TAG 

          echo " image pushed to dockerhub successfully"
          '''
        }
      }
    }
    stage('Deployment') {
      steps {
        container('kubectl') {
          
          
          sh '''
            sed -i "s/TAG/${BUILD_NUMBER}/g" deployment.yaml
            kubectl apply -f deployment.yaml -n demo
            kubectl apply -f service.yaml -n demo

         '''
         
        }
      }
    }

  }


}