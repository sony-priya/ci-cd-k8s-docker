pipeline{
    agent any
    environment {
        registry = "docker26priya/vprofileapptomcat"
        registryCredential = "dockerhun"
    }

    stages{
        stage('BUild'){
            steps{
                sh 'mvn clean install -Dskiptests'
            }
            post{
                success{
                    echo 'archieving'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('unit test'){
            steps {
                sh 'mvn test'
            }
        }

        stage('build && SonarQube analysis') {
                    environment {
                     scannerHome = tool 'sonar4.7'
                  }
                    steps {
                        withSonarQubeEnv('sonar-pro') {
                         sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                           -Dsonar.projectName=vprofile-repo \
                           -Dsonar.projectVersion=1.0 \
                           -Dsonar.sources=src/ \
                           -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                           -Dsonar.junit.reportsPath=target/surefire-reports/ \
                           -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                           -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                        }
                        timeout(time: 10, unit: 'MINUTES'){
                            waitForQualityGate abortPipeline: true
                        }
                    }
                }
        stage ('BUild app image'){
            steps{
                script{
                    dockerImage = docker.build registry + "v$BUILD_NUMBER"
                }
            }
        }

        stage('upload image'){
            steps{
                script{
                    docker.withRegistry('',registryCredential){
                        dockerImage.push('V$BUILD_NUMBER')
                        dockerImage.push('lastest')
                    }
                }
            }
        }

        stage("Remove unused docker image"){
            steps{
                sh 'docker rmi $registry:V$BUILD_NUMBER'
            }
        }
        stage('kubernetes deolpy'){
            steps{
                sh 'helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod'
            }
        }


    }
}