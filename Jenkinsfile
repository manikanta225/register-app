pipeline {
    agent any
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    
    stages{
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
		       version: '2.0.0'
            }

        }

    }}
