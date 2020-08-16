pipeline
{
	agent any
		tools
		{
			maven 'Maven3'
		}
		options
		{
			// Append timestamp to console outputs
			timestamps()
			
			timeout(time: 1, unit: 'HOURS')
			
			// Do not automatically checkout the SCM on every stage. We stash what
			// we need to save time
			skipDefaultCheckout()
			
			// Discard old builds after 10 days or 30 builds count
			buildDiscarder(logRotator(daysToKeepStr: '10', numToKeepStr: '30'))
			
			// To avoid concurrent builds to avoid multiple checkouts
			disableConcurrentBuilds()
		}
		stages
		{
			stage ('checkout')
			{
				steps
				{
					echo "build in master branch - 1"
					checkout scm
				}
			}
			stage ('Build')
			{
				steps
				{
					echo "build in master branch - 2"
					bat "mvn install"
				}
			}
			stage ('Unit Testing')
			{
				steps
				{
					echo "build in master branch - 3"
					bat "mvn test"
				}
			}
			stage ('Sonar Analysis')
			{
				steps
				{
					echo "build in master branch - 4"
					withSonarQubeEnv("SonarToken")
					{
						bat "mvn sonar:sonar"
					}
				}
			}
			stage ('Upload to Artifactory')
			{
				steps
				{
					echo "build in master branch - 5"
					rtMavenDeployer (
						id: 'deployer',
						serverId: 'myapp@123',
						releaseRepo: 'myapp-repository-test',
						snapshotRepo: 'myapp-repository-test'
					)
					rtMavenRun (
						pom: 'pom.xml',
						goals: 'clean install',
						deployerId: 'deployer'
					)
					rtPublishBuildInfo (
						serverId: 'myapp@123'
					)
				}
			}
			stage ('Docker Image')
			{
				steps
				{
					bat '/bin/docker build -t dtr.nagarro.com:443/myapp:${BUILD_NUMBER} --no-cache -f DockerFile .'
				}
			}
			stage ('Push to DTR')
			{
				steps
				{
					bat '/bin/docker push dtr.nagarro.com:443/myapp:${BUILD_NUMBER}'
				}
			}
			stage ('Stop Running Container')
			{
				steps
				{
					[cmdletbinding(DefaultParameterSetName='container')]

					param (
						[Parameter(ParameterSetName='container')]
[string]$cID = $(docker ps | grep 7000 | cut -d " " -f 1)
					)
					if ($cID) {
						docker stop $cID && docker rm -f $cID
					}
				}
			}
			
			stage ('Docker Deployment')
			{
				steps
				{
					bat 'docker run --name myapphelloworldapp -d -p 7000:8090 dtr.nagarro.com:443/myapp:${BUILD_NUMBER}'
				}
			}
		}
}