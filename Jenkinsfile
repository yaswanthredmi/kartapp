pipeline {
    agent any
    
    tools {
        jdk 'jdk11'
        maven 'Maven3'
        jfrog 'jfrog-cli'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        DOCKER_IMAGE_NAME = "devopsjfrog18.jfrog.io/ekart1-docker/ekart:1.0.0"
    }

    stages {
        stage('Git Checkout') {
            steps {
               git branch: 'main', changelog: false, poll: false, url: 'https://github.com/vasusav/Ekart.git'
            }
        }
        
        stage('Maven Build') {
            steps {
                sh "chmod +x scripts/mvnw"
               sh "scripts/mvnw clean package -Dmaven.test.skip=true"
            }
        }
       
        stage ('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh 'mvn sonar:sonar'
                }
            }
        }   

        /*stage('SonarQube Analysis') {
            steps {
                         sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.url=http://20.171.64.13:9000/ -Dsonar.login=squ_347ceab9cb82184e348d45a1681ff85a979808fd -Dsonar.projectName=Ekart
                         -Dsonar.java.binaries=. \
                         -Dsonar.projectKey=Ekart '''
            }
        }*/
        // stage('Build Application') {
        //     steps {
        //       sh "mvn clean install"
        //     }
       // }
       stage ('Build docker image') {
            steps {
                script {
                    sh 'ls -lrth'
                    //sh 'docker login -u vasusav -p ${hub_pwd}'
                    sh 'docker build -t vasusav/ekart:latest .'
                   // sh 'docker build -t devopsjfrog18.jfrog.io/ekart1-docker .'
                    sh 'docker images'
                  //  sh "docker push vasusav/ekart:latest"
                    
                }
                
            }
       }
       
       stage('Build jfrog Docker image') {
			steps {
				script {
					docker.build("$DOCKER_IMAGE_NAME", './')
				}
			}
		}
       
            
        stage('Build & Push Docker Image') {
            steps {
               script{
                   //withCredentials([string(credentialsId: 'jfrog-cred', variable: 'jfrog-cred')]) {
                       // sh 'docker login -u devopsjfrog18@gmail.com devopsjfrog18.jfrog.io -p Devops18'
                       // sh 'docker login -u devopsjfrog18@gmail.com devopsjfrog18.jfrog.io -p cmVmdGtuOjAxOjE3Mzc2OTYwMjg6NmZMc2Z5b0NmR1VTVnphOTJIbXMxWHZhQmZZ'
                        sh 'docker login -u vasusav -p 114338@Vad docker.io'
                        sh "docker push docker.io/vasusav/ekart:latest"
                  //  }
                }
            }
        }
        
    	stage('Scan and push to jfrog') {
			steps {
				dir('${WORKSPACE}') {
					// Scan Docker image for vulnerabilities
				//	jf 'docker scan $DOCKER_IMAGE_NAME'

					// Push image to Artifactory
					jf 'docker push $DOCKER_IMAGE_NAME'
				}
			}
		}

		stage('Publish build info') {
			steps {
				jf 'rt build-publish'
			}
		}
    }
}
