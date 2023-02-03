pipeline {
    agent any
    tools {
    maven 'M2'
    }
    stages {
        stage('checkout') {
            steps {
                git 'https://github.com/Shi191099/JavaCode.git'
            }
        }
        stage('build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "mvn clean verify sonar:sonar"
                }
            }
    	}
    	stage('Execute Sonarqube Report')  {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "mvn sonar:sonar"
                }  
            }
        }
        stage('Quality Gate Check') {
            steps {
                waitForQualityGate abortPipeline: true    
            }
        }
        stage ('Upload Artifact to Nexus') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'maven-project', classifier: '', file: 'webapp/target/webapp.war', type: 'war']], credentialsId: 'Nexus', groupId: 'com.example.maven-project', nexusUrl: '3.8.92.69:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'Artifact-nexus', version: '${GIT_COMMIT}'
            }
        }
        stage('E-mail Approval') {
            steps {
                emailext mimeType: 'text/html', subject: "[Jenkins]${currentBuild.fullDisplayName}",
	        	to: 'ashi191099@gmail.com',
	        	body: '''<h1>Deployed</h1>'''
	        }
        }
        stage('Publish over SSH') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible-server', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'ansible-playbook -i hosts project.yml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/home/ansadmin', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '**/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])            }
        }
    }
}
