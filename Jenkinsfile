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
			stage('Docker Image'){
				steps{
					echo "Test: $Build_NUMBER"
					bat "docker build -t gaurav7576/myapp:$Build_NUMBER --no-cache -f Dockerfile ."
				}
			}
			stage ('Push to DTR')
			{
				steps
				{
					bat 'docker push gaurav7576/myapp:${BUILD_NUMBER}'
				}
			}
			stage ('Docker Deployment')
			{
				steps
				{
					bat 'docker run --name myapphelloworldapp -d -p 7000:8090 gaurav7576/myapp:${BUILD_NUMBER}'
				}
			}
		}
}