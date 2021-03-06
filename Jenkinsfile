pipeline {
	agent any
	tools {
		maven 'Maven'
	}
	stages {
		stage('Checkout') {
			steps {
				echo "Check out project. No required but if you want to see ouput"
				sleep 3
			}
		}
		stage('Build') {
			steps {
				sh 'mvn clean package -B -DskipTests'
			}
		}
		/*stage('Test Failure') {
			steps {
				echo 'Fail'
				exit 1
			}
		}*/
		/*stage('Conditional Stage') {
			when {
				expression {
					return false
				}
			}
			steps {
				echo 'Skipped Stage'
			}
		}*/
		stage('Archive') {
			steps {
				echo 'Save artifact to Archiva'
				sleep 5
			}
		}
		stage ('Dev Deploy') {
			parallel('Dev Deploy') {
				stage('Stage') {
					steps {
						echoTest("Dev")
					}
				}
				stage('QA') {
					steps {
						echoTest("QA")
					}
				}
				stage('CDE') {
					steps {
						echoTest("CDE")
					}
				}
			}
		}
		stage('Test') {
			parallel {
				stage('Selenium') {
					steps {
						sh 'mvn test'	
					}
					post {
						always {
							junit '**/target/surefire-reports/**/*.xml'
						}
					}				
				}
				stage('Sonarqube') {
					environment {
        				scannerHome = tool 'SonarQubeScanner'
    				}
					steps {
						withSonarQubeEnv('sonarqube') {
            				//sh "${scannerHome}/bin/sonar-scanner"
            				sh 'mvn sonar:sonar'
        				}
        				timeout(time: 10, unit: 'MINUTES') {
            				waitForQualityGate abortPipeline: true
        				}
					}
				}
			}
		}
		stage('Approval') {
			steps {
				timeout(15) {
					input message: 'Deploy to production?', ok: 'Fire away!'
				}
				echo 'go'
			}
		}
		stage ('Deploy to Production') {				
			parallel('Prod') {
				stage('HQ1') {
					steps {
						echoTest("HQ1")
					}
				}
				stage('HQ2') {
					steps {
						echoTest("HQ2")
					}
				}
			}	
		}
	}
	post {
		always {
			echo 'One way or another, I have finished'
			deleteDir()
			sleep 5
		}
		success {
			echo 'I succeeded!'
			sleep 3
		}
		aborted {
			echo 'Aborted!'
			sleep 2
		}
		failure {
			echo 'I failed :('
			sleep 5
		}
	}	
}

def echoTest(String environmentName) {
	echo 'Test Deploy Prod ' + environmentName
	echo 'Download artifact'
	echo 'Untar it!'
	echo 'Restart HTTPD'
	echo 'Run Smoke Tests'
	sleep 10
}