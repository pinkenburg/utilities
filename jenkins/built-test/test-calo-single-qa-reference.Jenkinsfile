pipeline 
{
	agent any
    
//    environment { 
//        JenkinsBase = 'jenkins/test/'
//    }
       
	stages { 
			
		stage('Initialize') 
		{
			
            
			steps {
				timestamps {
					ansiColor('xterm') {
					
						sh("env");
						
						script{
							//if(${ref_build_id} == null)
							//{
								//echo("Build failed because of ref_build_id is empty (${ref_build_id})");
								//error("Build failed because of ref_build_id is empty")				
							//}
						}
					
						echo("Build ref_build_id = (${ref_build_id})");
						slackSend (color: '#FFFF00', message: "STARTED: Job with reference build ${ref_build_id} '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
						
						deleteDir()						

					}
				}
			}
		}
		
		stage('test-calo-single-qa')
		{
			//input {
			//	message "Pick a reference build in ${test-calo-single-qa}?"
      //          parameters {
      //              string(name: 'ref_build_id', description: 'Reference build ID?')
      //          }
      //      }
					
			steps 
			{
					
		   	copyArtifacts(projectName: "${test-calo-single-qa}", selector: specific("${ref_build_id}"));
		   	
		   	dir('macros/macros/g4simulations/')
		   	{
		   		stash name: "test-calo-single-qa-stash", includes: "*"
		   	}
		   	
				deleteDir()		
		   			
		   	unstash "test-calo-single-qa-stash"
		   	archiveArtifacts artifacts: '*.*', onlyIfSuccessful: true	
		   	
				slackSend (color: '#00FF00', message: "Selected reference build ${ref_build_id}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
			}				
		}//		stage('test-calo-single-qa')
		
		
	}
	
	post {

		success {
			slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
		}
		failure {
			slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
		}
		unstable {
			slackSend (color: '#FFF000', message: "UNSTABLE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
		}
	}
}//pipeline 

