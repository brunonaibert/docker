pipeline {
	agent {
		label 'buildtestmed'
	}
	stages {
		stage ('Initializing Variables') {
			steps {
				script {
					current_version = getOfficialVersion("updates")
					med_id = getEC2Metadata("instance-id")
				}
			}
		}
		stage ('Build') {
			parallel {
				stage ('Build Simplerisk Ubuntu 18.04') {
					steps {
						sh """
							sudo docker build -t simplerisk/simplerisk -f simplerisk/bionic/Dockerfile simplerisk/
							sudo docker build -t simplerisk/simplerisk:$current_version -f simplerisk/bionic/Dockerfile simplerisk/
							sudo docker build -t simplerisk/simplerisk:$current_version-bionic -f simplerisk/bionic/Dockerfile simplerisk/
						"""
					}
				}
				stage ('Build Simplerisk Ubuntu 20.04') {
					steps {
						sh """
							sudo docker build -t simplerisk/simplerisk:$current_version-focal -f simplerisk/bionic/Dockerfile simplerisk/
						"""
					}
				}
			}
			post {
				failure {
					node("jenkins") {
						terminateInstance("${med_id}")
					}
				}
			}
		}
	}
}

def getCurrentDate() {
	return sh(script: "date +\"%Y%m%d-001\"", returnStdout: true).trim()
}

def getEC2Metadata(String attribute){
	return sh(script: "echo \$(TOKEN=`curl -s -X PUT \"http://169.254.169.254/latest/api/token\" -H \"X-aws-ec2-metadata-token-ttl-seconds: 21600\"` && curl -s -H \"X-aws-ec2-metadata-token: \$TOKEN\" http://169.254.169.254/latest/meta-data/$attribute)", returnStdout: true).trim()
}

def getOfficialVersion(String updatesDomain="updates-test") {
	return sh(script: "echo \$(curl -sL https://$updatesDomain\\.simplerisk.com/Current_Version.xml | grep -oP '<appversion>(.*)</appversion>' | cut -d '>' -f 2 | cut -d '<' -f 1)", returnStdout: true).trim()
}

void terminateInstance(String instanceId, String region="us-east-1") {
	sh "aws ec2 terminate-instances --instance-ids $instanceId --region $region"
}
