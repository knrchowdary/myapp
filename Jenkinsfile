def GIT_BRANCH
def PROJ_VERSION
def VERSION
def BRANCH
pipeline {
    agent any
	tools {
        maven 'MAVAN-3.3.9'
        jdk 'JDK891'
    }
	options {
	ansiColor("xterm")
    }
	environment {
		MAVEN_OPTS = '-Xmx2048m'
	}
    stages {
        stage ('Read version') {
            steps {
                script {		
                  GIT_BRANCH = "${BRANCH_NAME}"				
				  PROJ_VERSION = readMavenPom(file: 'api-hotels/pom.xml').getVersion()	
                  BRANCH =  sh(returnStdout: true, script: "echo $GIT_BRANCH | sed 's@/@-@g'").trim()	
                  echo "$BRANCH"			  
                }
            } 
		} 
        stage ('version set') { 
		    steps {
			    script {
				  if ( GIT_BRANCH ==~ /.*develop/ ){
				     sh "mvn --batch-mode -f api-hotels versions:set -DnewVersion=$PROJ_VERSION-SNAPSHOT"
					}
				  if ( GIT_BRANCH ==~ /.*feature\/.*/ ) {
				     sh "mvn --batch-mode -f api-hotels versions:set -DnewVersion=$PROJ_VERSION-$BRANCH-SNAPSHOT"
					}
				  if ( GIT_BRANCH ==~ /.*release\/.*|.*hotfix\/.*/ ) {
				     sh "mvn --batch-mode -f api-hotels versions:set -DnewVersion=$PROJ_VERSION-RC${BUILD_ID}"
					}
				  if ( GIT_BRANCH ==~ /.*master/ ) {
				     sh "mvn --batch-mode -f api-hotels versions:set -DnewVersion=$PROJ_VERSION${BUILD_ID}"
					}
				}
            }
        }
		stage ('Build') {
		    steps{
			  script {
				sh 'mvn --batch-mode -f api-hotels clean deploy -s .m2.settings.xml'
				VERSION = readMavenPom(file: 'api-hotels/pom.xml').getVersion()
			    echo "$VERSION"
			  }
			}
		}
		stage (' DEV deploy') {
            when
                {
                  expression {
                        GIT_BRANCH ==~ /.*develop|.*feature\/.*/
                    }
                }
		   steps {
		      script {
				  echo "Deploy application on developmment environment"
				  ansibleTower( towerServer: 'awx', towerCredentialsId: 'ansible-awx', templateType: 'job', jobTemplate: 'DEV dpax', importTowerLogs: true, inventory: '', jobTags: '', skipJobTags: '', limit: '', removeColor: false, verbose: true, credential: '', extraVars: '''--- 
version: '''+ VERSION +''' ''', async: false )
			    }  
            }
		}
        stage (' QA deploy') {	
            when
                {
                  expression {
                        GIT_BRANCH ==~ /.*release\/.*|.*hotfix\/.*/
                    }
                }
		   steps {
		      script {
				  echo 'Deploy application on qa environment'
                  timeout(time:5, unit:'DAYS') {
                      input message : 'Approval for staging needed' }
				  ansibleTower( towerServer: 'awx', towerCredentialsId: 'ansible-awx', templateType: 'job', jobTemplate: 'QA dpax', importTowerLogs: true, inventory: '', jobTags: '', skipJobTags: '', limit: '', removeColor: false, verbose: true, credential: '', extraVars: '''--- 
version: '''+ VERSION +''' ''', async: false )
				}
			}
        }			
        stage (' UAT deploy') {	
            when
                {
                  expression {
                        GIT_BRANCH ==~ /.*release\/.*|.*hotfix\/.*/
                    }
                }
		    steps {
		      script {
				  echo 'Deploy application on uat environment'
                  timeout(time:5, unit:'DAYS') {
                      input message : 'Approval for staging needed' }
				  ansibleTower( towerServer: 'awx', towerCredentialsId: 'ansible-awx', templateType: 'job', jobTemplate: 'UAT dpax', importTowerLogs: true, inventory: '', jobTags: '', skipJobTags: '', limit: '', removeColor: false, verbose: true, credential: '', extraVars: '''--- 
version: '''+ VERSION +''' ''', async: false )
				}
			}
		}	
        stage (' PROD deploy') {	
            when
                {
                  expression {
                        GIT_BRANCH ==~ /.*release\/.*|.*hotfix\/.*/
                    }
                }
		    steps {
		      script {
				  echo 'Deploy application on prod environment'
                  timeout(time:5, unit:'DAYS') {
                      input message : 'Approval for staging needed' }
				  ansibleTower( towerServer: 'awx', towerCredentialsId: 'ansible-awx', templateType: 'job', jobTemplate: 'PROD dpax', importTowerLogs: true, inventory: '', jobTags: '', skipJobTags: '', limit: '', removeColor: false, verbose: true, credential: '', extraVars: '''--- 
version: '''+ VERSION +''' ''', async: false )
				}      				
            }		
		}
	}
	post {
        failure {
            slackSend baseUrl: 'https://hooks.slack.com/services/', channel: '#eng_jenkins_builds', color: 'danger', message: "Build failed in Jenkins: ${JOB_NAME} - ${BUILD_NUMBER}, url: (<${BUILD_URL}|Open>)", teamDomain: 'accommodationsplus', tokenCredentialId: 'slack-dpax' 
            }
        unstable {
            slackSend baseUrl: 'https://hooks.slack.com/services/', channel: '#eng_jenkins_builds', color: 'warning', message: "Unstable build in Jenkins: ${JOB_NAME} - ${BUILD_NUMBER}, url: (<${BUILD_URL}|Open>)", teamDomain: 'accommodationsplus', tokenCredentialId: 'slack-dpax' 
            }
    }		
}    
