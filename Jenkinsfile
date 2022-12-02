import groovy.json.JsonSlurperClassic

def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}
pipeline {
    agent any
    environment {
        channel='C04BPL2A5E3'
        NEXUS_PASSWORD     = credentials('nexusid')
    }
    stages {
     
        stage("Paso 1: Build && Test"){
            steps {
                script{
                    sh "echo 'Build && Test!'"
                    sh "./mvnw clean package -e"    
                }
            }
        }

        stage("Paso 2: Sonar - Análisis Estático"){
            steps {
                script{
                    sh "echo 'Análisis Estático!'"
                        withSonarQubeEnv('sonarqube') {
                            sh "echo 'Calling sonar by ID!'"
                            // Run Maven on a Unix agent to execute Sonar.
                            sh './mvnw clean verify sonar:sonar -Dsonar.projectKey=custom-project-key -Dsonar.projectName=cejemplo-maven-full-stages -Dsonar.java.binaries=build'
                        }
                        
                }
            }
        }
        
        stage("Paso 3: Curl Springboot maven sleep 20"){
            steps {
                script{
                    sh "nohup bash ./mvnw spring-boot:run  & >/dev/null"
                    sh "sleep 20 && curl -X GET 'http://localhost:8081/rest/mscovid/test?msg=testing'"
                }
            }
        }
        stage("Paso 4: Detener Spring Boot"){
            steps {
                script{
                    sh '''
                        echo 'Process Spring Boot Java: ' $(pidof java | awk '{print $1}')  
                        sleep 20
                        kill -9 $(pidof java | awk '{print $1}')
                    '''
                }
            }
        }

        stage("Paso 4.2: levantar newman "){
            steps {
                script{
                    sh "nohup bash ./mvnw spring-boot:run &"
                    sh " newman run '/tmp/ejemplo-maven.postman_collection.json';"
                }
            }
        }
           stage("Paso 5: Subir Artefacto a Nexus"){
            steps {
                script{
                    nexusPublisher nexusInstanceId: 'nexus',
                        nexusRepositoryId: 'maven-usach-ceres',
                        packages: [
                            [$class: 'MavenPackage',
                                mavenAssetList: [
                                    [classifier: '',
                                    extension: 'jar',
                                    filePath: 'build/DevOpsUsach2020-0.0.1.jar'
                                ]
                            ],
                                mavenCoordinate: [
                                    artifactId: 'DevOpsUsach2020',
                                    groupId: 'com.devopsusach2020',
                                    packaging: 'jar',
                                    version: '0.0.1'
                                ]
                            ]
                        ]
                }
            }
        }
       stage("Paso 6: Descargar Nexus"){
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexusid', passwordVariable: 'NXS_PASSWORD', usernameVariable: 'NXS_USERNAME')]) {
                    sh ' curl -X GET -u $NXS_USERNAME:$NXS_PASSWORD "http://nexus:8081/repository/maven-usach-ceres/com/devopsusach2020/DevOpsUsach2020/0.0.1/DevOpsUsach2020-0.0.1.jar" -O'
                }
                 //script{
                 //    sh ' curl -X GET -u "http://nexus:8081/repository/maven-usach-ceres/com/devopsusach2020/DevOpsUsach2020/0.0.1/DevOpsUsach2020-0.0.1.jar" -O'
                 //}
            }
        }
         stage("Paso 7: Levantar Artefacto Jar en server Jenkins"){
            steps {
                script{
                    sh 'nohup java -jar DevOpsUsach2020-0.0.1.jar & >/dev/null'
                }
            }
        }
          stage("Paso 8: Testear Artefacto - Dormir(Esperar 20sg) "){
            steps {
                script{
                    sh "sleep 20 && curl -X GET 'http://localhost:8081/rest/mscovid/test?msg=testing'"
                }
            }
        }
        stage("Paso 9:Detener Atefacto jar en Jenkins server"){
            steps {
                sh '''
                    echo 'Process Java .jar: ' $(pidof java | awk '{print $1}')  
                    sleep 20
                    kill -9 $(pidof java | awk '{print $1}')
                '''
            }
        }
    }
}