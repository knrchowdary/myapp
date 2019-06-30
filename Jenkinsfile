pipeline
{

	agent any

 

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
		
                ansiblePlaybook(
                    playbook: 'sample.yml',
                    inventory: 'myinv',
					limit: '${QA}',
					disableHostKeyChecking: true,
					credentialsId: 'ansiblekey',
					colorized: true)
            
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
         
                ansiblePlaybook(
                    playbook: 'sample.yml',
                    inventory: 'myinv',
					limit: '${QA}',
					disableHostKeyChecking: true,
					credentialsId: 'ansiblekey',
					colorized: true)
     
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
				 ansiblePlaybook(
                    playbook: 'sample.yml',
                    inventory: 'myinv',
					limit: '${PROD}',
					disableHostKeyChecking: true,
					credentialsId: 'ansiblekey',
					colorized: true)
				
            }
        }
    }

 

}
