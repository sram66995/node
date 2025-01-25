pipeline {
    agent {
        label '10.100.1.67_DOCKER_MACHINE' 
    }
	environment {
        ADMIN_CRED=credentials('d2f45ade-a7f2-4351-a2b0-826b499794ca')
    }
	stages {
		
	   stage ('SONAR_SCAN') {
            steps {
				sh '''
						sudo docker build --no-cache -t sonar:1.1 /INSURANCE/JENKINS/workspace/DEMO/GIT_CI_CD_DEMO/Sonar
					'''

            }
	    }
	    stage ('BUILD') {
            steps {
				sh '''
			    sudo docker build --no-cache -t cicd_demo:1.1 .
				'''
            }
	    }
		stage ('ARTIFACT') {
			steps {
				sh '''
				sudo docker tag  cicd_demo:1.1 10.100.1.67:32702/cicd_demo:1.1
				sudo docker push 10.100.1.67:32702/cicd_demo:1.1
				'''
			}      
		}
		stage ('DEPLOYMENT APPROVAL') {
					when {
						expression {
							return env.GIT_BRANCH ==~ /(origin\/master|master)/
						}
					}
					steps {
						script {
							timeout(time: 4, unit: 'HOURS') {
								input id: 'CustomID', 
								message: 'Deployment Step', 
								ok: 'Proceed', 
								parameters: [password(defaultValue: 'admin', description: 'Please confirm if you want to proceed with the deployment. You need to be admin to confirm.', name: 'admin')], 
								submitter: 'admin', 
								submitterParameter: 'approvingSubmitter'
							}
						}
					}
				}
		stage ('APP_DEPLOYMENT') {
			steps {
				sh '''
				sudo docker rm -f cicd_demo
				sudo docker run -dt --name  cicd_demo -p 8090:8080 10.100.1.67:32702/cicd_demo:1.1
				'''
			}
		}
	}

	post{
        always{
            emailext body: '<h3> $BUILD_STATUS ${JELLY_SCRIPT,template="html-with-health-and-console"} </h3>',
        	mimeType: 'both',
        	recipientProviders: [[$class: 'DevelopersRecipientProvider'],
        	[$class: 'RequesterRecipientProvider']],
        	subject: '[Insurance]::[GIT_CI_CD_DEMO] : [$GIT_BRANCH] [Build #$BUILD_NUMBER] - $BUILD_STATUS',
        	to: 'release.nsure@kgisl.com'
        }
    }		
}
    



