pipeline {
	agent any
	
	tools {
		jdk 'Java17'
		maven 'Maven3'
	}
	environment 
	{
		APP_NAME = "register-app"
                RELEASE = "1.0.0"
                DOCKER_USER = "manikanta225"
                DOCKER_PASS = 'dockerhub'
                IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
                IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
	}
	stages
	{
	    stage("Cleanup Workspace"){
                steps {
			cleanWs()
                }
        }
	    stage("Checkout from SCM"){
                steps {
			git branch: 'main', credentialsId: 'github', url: 'https://github.com/manikanta225/register-app.git'
                }
        }
	    stage("Build Application"){
		    steps {
			    sh "mvn clean package"
            }

       }
	    stage("Test Application"){
		    steps {
			    sh "mvn test"
           }
       }
	    stage("SonarQube Analysis"){
		    steps {
			    script {
				    withSonarQubeEnv(credentialsId: 'sonar-jenkins') { 
					    sh "mvn sonar:sonar"
		        }
	           }	
           }
       }
	stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-jenkins'
                }	
            }

        }

	stage("upload war to nexus"){
           steps {
               nexusArtifactUploader artifacts: [
		       [
			       artifactId: 'maven-project', 
			       classifier: '', 
			       file: 'webapp/target/webapp.war', 
			       type: 'war'
		       ]
	       ], 
		       credentialsId: 'nexus3', 
		       groupId: 'com.example.maven-project',
		       nexusUrl: '15.207.112.122:8081',
		       nexusVersion: 'nexus3', 
		       protocol: 'http',
		       repository: 'maven-releases', 
		       version: ("4.0.${env.BUILD_NUMBER}")
            }

        }
	stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }

       }
	    stage("Trivy Scan") {
		    steps {
			    script {
				    sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image manikanta225/register-app:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }
	    
	     stage ('Cleanup Artifacts') {
		     steps {
			     script {
				     sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
				     sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }
	    stage("Trigger CD Pipeline") {
		    steps {
			    script {
				    sh "curl -v -k --user mani:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-43-205-111-28.ap-south-1.compute.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'"
                }
            }
       }
    

    

    }}

