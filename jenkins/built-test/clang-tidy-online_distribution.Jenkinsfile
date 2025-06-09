pipeline 
{
	agent any
           
	stages { 
		stage('Checkrun update') 
		{
		
			steps {
				build(job: 'github-commit-checkrun',
				parameters:
				[
					string(name: 'checkrun_repo_commit', value: "${checkrun_repo_commit}"), 
					string(name: 'src_Job_id', value: "${env.JOB_NAME}/${env.BUILD_NUMBER}"),
					string(name: 'src_details_url', value: "${env.BUILD_URL}"),
					string(name: 'checkrun_status', value: "in_progress")
				],
				wait: false, propagate: false)
			} // steps
		} // stage('Checkrun update') 
		
		stage('Initialize') 
		{
			
            
			steps {
				timestamps {
					ansiColor('xterm') {
																
						script {
						
							currentBuild.displayName = "${env.BUILD_NUMBER} - ${sha_online_distribution}"
							currentBuild.description = "${upstream_build_description} / <a href=\"${git_url_online_distribution}\">online_distribution</a> # ${sha_online_distribution}" 
							
						}
										
						dir('online_distribution') {
							deleteDir()
						}
						dir('report') {
							deleteDir()
						}
						sh('rm -fv clang-tidy-result.txt')

						dir('build') {
							sh('chmod -R +w ./')
							deleteDir()
						}
					
						sh('hostname')
						sh('pwd')
						sh('env')
						sh('ls -lvhc')

					}
				}
			}
		}

		stage('Git Checkout')
		{
			
			steps 
			{
				timestamps { 
					ansiColor('xterm') {
						
						dir('online_distribution') {
							// git credentialsId: 'sPHENIX-bot', url: 'https://github.com/sPHENIX-Collaboration/online_distribution.git'
							
							checkout(
								[
						 			$class: 'GitSCM',
						   		extensions: [               
							   		[$class: 'CleanCheckout'],     
							     	[
							   			$class: 'PreBuildMerge',
							    		options: [
											mergeRemote: 'origin',
							  			mergeTarget: "$mergeTarget_online_distribution"
							  			]
							    	],
						   		],
							  	branches: [
							    	[name: "${sha_online_distribution}"]
							    ], 
							  	userRemoteConfigs: 
							  	[[
							    	//credentialsId: 'sPHENIX-bot', url: 'https://github.com/sPHENIX-Collaboration/online_distribution.git'
							     	credentialsId: 'sPHENIX-bot', 
							     	url: '${git_url_online_distribution}',
							     	refspec: ('+refs/pull/*:refs/remotes/origin/pr/* +refs/heads/master:refs/remotes/origin/master'), 
							    	branch: ('*')
							  	]]
								] //checkout
							)//checkout
						}//						dir('online_distribution') {


					}
				}
			}
		}//stage('SCM Checkout')
		
	    stage ('Git mining') {
		steps {
			discoverGitReferenceBuild (requiredResult: hudson.model.Result.SUCCESS)
			mineRepository()
			gitDiffStat()
			}
	    }
			
	    stage('clang-tidy')
	    {
	      steps 
	      {
            writeFile file: "build.sh", text:'''#!/usr/bin/env bash

                source /opt/sphenix/core/bin/sphenix_setup.sh -n new;

                which clang-tidy; 
                env;

                cd online_distribution/newbasic
                echo "// *** this is a generated file. Do not commit, do not edit" > ioselect.h
                echo "#ifndef __IOSELECT_H__" >> ioselect.h
                echo "#define __IOSELECT_H__" >> ioselect.h
                echo "#define STREAMBUF_NEW_IOSTREAM" >> ioselect.h
                echo  >> ioselect.h
                echo "#endif" >> ioselect.h

                cd ${WORKSPACE}
                pwd
                ls -lhcv

                if test -f clang-tidy-result.txt; then
                    mv -fv clang-tidy-result.txt clang-tidy-result.txt.backup
                fi

                shopt -s globstar
                clang-tidy ./online_distribution/**/*.cc  ./online_distribution/**/*.cxx -- -Wall -Werror -Wshadow -std=c++17 -Wno-dangling -isystem${WORKSPACE}/online_distribution/newbasic -isystem${WORKSPACE}/online_distribution/pmonitor -isystem$OFFLINE_MAIN/include -isystem$ROOTSYS/include -isystem$G4_MAIN/include -isystem$G4_MAIN/include/Geant4  -isystem$OPT_SPHENIX/include -DLinux= -DHAVE_GETOPT_H= > clang-tidy-result.txt

                ls -hvl $PWD/clang-tidy-result.txt
                wc -l clang-tidy-result.txt
                head -n 10 clang-tidy-result.txt

                ''';//  writeFile file: "build.sh", 

            sh('chmod +x build.sh');
            sh('$singularity_exec_sphenix bash -v build.sh')


	
	      }
	    }// Stage - cpp check
	
	    stage('clang-tidy-analysis')
	    {	      
	      steps 
	      {
	        archiveArtifacts artifacts: 'clang-tidy-result.txt'
	        recordIssues qualityGates: [[threshold: 1, type: 'NEW', unstable: true], [threshold: 1, type: 'NEW_HIGH', unstable: true], [threshold: 40, type: 'NEW', unstable: false], [threshold: 1, type: 'NEW_HIGH', unstable: false]], tools: [clangTidy(pattern: 'clang-tidy-result.txt')]
	      }										
	    } // 				stage('sPHENIX-Build')

	}//stages
		
	post {
		always{
		  
			dir('report')
			{
				sh('ls -lvhc')
			  writeFile file: "clang-tidy.md", text: "* [![Build Status ](${env.JENKINS_URL}/buildStatus/icon?job=${env.JOB_NAME}&build=${env.BUILD_NUMBER})](${env.BUILD_URL}) `clang-tidy` [is ${currentBuild.currentResult}](${env.BUILD_URL}), [:bar_chart:`clang-tidy` report (full)](${env.BUILD_URL}/clang-tidy/)/[(new)](${env.BUILD_URL}/clang-tidy/new/)"				
			}
		  		  
			archiveArtifacts artifacts: 'report/*.md'
			
			build(job: 'github-commit-checkrun',
				parameters:
				[
					string(name: 'checkrun_repo_commit', value: "${checkrun_repo_commit}"), 
					string(name: 'src_Job_id', value: "${env.JOB_NAME}/${env.BUILD_NUMBER}"),
					string(name: 'src_details_url', value: "${env.BUILD_URL}"),
					string(name: 'checkrun_status', value: "completed"),
					string(name: 'checkrun_conclusion', value: "${currentBuild.currentResult}"),
					string(name: 'output_title', value: "sPHENIX Jenkins Report for ${env.JOB_NAME}"),
					string(name: 'output_summary', value: "* [![Build Status ](${env.JENKINS_URL}/buildStatus/icon?job=${env.JOB_NAME}&build=${env.BUILD_NUMBER})](${env.BUILD_URL}) `clang-tidy` [is ${currentBuild.currentResult}](${env.BUILD_URL}), [:bar_chart:`clang-tidy` report (full)](${env.BUILD_URL}/clang-tidy/)/[(new)](${env.BUILD_URL}/clang-tidy/new/)"),
					string(name: 'output_text', value: "${currentBuild.displayName}\n\n${currentBuild.description}")
				],
				wait: false, propagate: false
			) // build(job: 'github-commit-checkrun',
		
		}
	
	}
	
}//pipeline 
