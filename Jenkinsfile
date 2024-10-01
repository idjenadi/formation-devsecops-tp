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
          sh "mvn sonar:sonar \
  -Dsonar.projectKey=maven-jenkins-pipeline2 \
  -Dsonar.host.url=http://10.117.20.175:9999 \
  -Dsonar.login=4bed601a67560b7d4902ebcd7d9c655c29b1c1b6"
        }
      }
    }
    }
}
