pipeline {
  agent any

  stages {
    //--------------------------
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        } 
        //--------------------------
    stage('UNIT test & jacoco ') {
      steps {
        sh "mvn test"
      }
 
  post {

        always {

          junit 'target/surefire-reports/*.xml'

          jacoco execPattern: 'target/jacoco.exec'

        }

      }
    }
    //--------------------------

    stage('Mutation Tests - PIT') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          sh "mvn org.pitest:pitest-maven:mutationCoverage"
        }
      }
        post { 
         always { 
           pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
         }
       }
    }
//--------------------------
        //--------------------------

    stage('SonarQube_scanne') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          withCredentials([string(credentialsId: 'Token_Sonar', variable: 'TOKEN')]){
          sh "mvn sonar:sonar \
  -Dsonar.projectKey=maven-jenkins-pipeline2 \
  -Dsonar.host.url=http://10.117.20.175:9999 \
  -Dsonar.login=${TOKEN}"
        }
      }}
    }
    //--------------------------
    stage('Docker Build and Push') {
      steps {
        withCredentials([string(credentialsId: 'DOCKER_HUB_PASSWORD_Imad', variable: 'DOCKER_HUB_PASSWORD')]) {
          sh 'sudo docker login -u imad403  -p $Password_DockerHub'
          sh 'printenv'
          sh 'sudo docker build -t imad403/devops-app:""$GIT_COMMIT"" .'
          sh 'sudo docker push imad403/devops-app:""$GIT_COMMIT""'
        }
 
      }
    }
	  //-----------------------------------

	      	 stage('Vulnerability Scan - Docker Trivy') {
       steps {
	        withCredentials([string(credentialsId: 'trivy_token_achraf', variable: 'TOKEN')]) {
			 catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                 sh "sed -i 's#token_github#${TOKEN}#g' trivy-image-scan.sh"
                 sh "sudo bash trivy-image-scan.sh"
	       }
		}
       }
     }

    //----------------------------

        stage('Deployment Kubernetes  ') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfigachraf']) {
              sh "sed -i 's#replace#hrefnhaila/devops-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
              sh 'kubectl apply -f k8s_deployment_service.yaml'
        }
      }
    }

//---------------------------------
}
}
