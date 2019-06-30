pipeline
{
  agent any

  tools { 
        maven 'mvn' 
        jdk 'jdk8' 
    }
	
  options {
	ansiColor("xterm")
    }

    environment
    {
        GIT_BRANCH_NAME = "1"
        MY_BUILD_VERSION = "1"
    }

 

    stages
    {
        stage('Build')
        {
            steps
            {
                script
                {
                    echo "another branch"
                    c = checkout changelog: false, poll: true, scm: [$class: 'GitSCM', branches: [[name: '**']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git_creds', url: 'https://github.com/knrchowdary/myapp.git']]]
                    echo "${c}"
                    MY_BUILD_VERSION = c.GIT_COMMIT[0..4]
                    echo MY_BUILD_VERSION
                    GIT_BRANCH_NAME = c.GIT_BRANCH

 

                    sh "mvn -Drevision=${MY_BUILD_VERSION} clean install"
                    
                }
            }

 

        }
        
        stage("Publish")
        {
        
            when
             {
                    expression {
                        GIT_BRANCH_NAME ==~ /.*master|.*feature|.*develop|.*hotfix/
                    }
            }
            
            steps
            {
                script
		    {
		    
		  sh "mvn -Drevision=${MY_BUILD_VERSION} clean deploy"
           
                }
            }

 

        }
        stage('DEV')
        {
            when
                 {
                        expression {
                            GIT_BRANCH_NAME ==~ /.*master|.*feature|.*develop|.*hotfix/
                        }
                }
            steps
            {
                script
                {
                    echo 'Deploy stage goes here'
                
                    timeout(time:5, unit:'DAYS')
                    {
                      input message : 'Approval for staging needed'
                    }
                  
                }
		
                sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'ansible-playbook sample.yml -i myinv -l DEV -u vagrant ', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'sample.yml, myinv')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            
            }
        }

 

        stage('QA')
        {
            when
                 {
                        expression {
                            GIT_BRANCH_NAME ==~ /.*master|.*feature|.*develop|.*hotfix/
                        }
                }

 

            steps
            {
		    script
                {
                    echo 'Deploy stage goes here'
                
                    timeout(time:5, unit:'DAYS')
                    {
                      input message : 'Approval for staging needed'
                    }
                  
                }
         
                sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'ansible-playbook sample.yml -i myinv -l QA -u vagrant ', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'sample.yml, myinv')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }

 

        stage('PRD')
        {
            when
                 {
                        expression {
                            GIT_BRANCH_NAME ==~ /.*master|.*feature|.*develop|.*hotfix/
                        }
                }

 

            steps
            {
                script
                {
                    echo 'PRD stage goes here'
                
                    timeout(time:5, unit:'DAYS')
                    {
                      input message : 'Approval for staging needed'
                    }
                   
                }
				 sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'ansible-playbook sample.yml -i myinv -l PROD -u vagrant ', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'sample.yml, myinv')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
				
            }
        }
    }

 

}
