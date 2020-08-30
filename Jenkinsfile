pipeline
{
	agent any
		tools
		{
			maven 'Maven 3'
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
					withSonarQubeEnv("Test_Sonar")
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
						serverId: '123456789@artifactory',
						releaseRepo: 'myapp-repository-test',
						snapshotRepo: 'myapp-repository-test'
					)
					rtMavenRun (
						pom: 'pom.xml',
						goals: 'clean install',
						deployerId: 'deployer'
					)
					rtPublishBuildInfo (
						serverId: '123456789@artifactory'
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
					bat "docker login -u gaurav7576 -p Ruchi@123"
					bat "docker push gaurav7576/myapp:$Build_NUMBER"
				}
			}
			stage('Stop running container'){
				steps{
					script{
						conatiner = false
						container = bat(script: "@docker ps -aqf name=myapphelloworldapp", returnStdout: true).trim();
						if ("$container"){
							bat "docker stop $container"
							bat "docker rm -f $container"
						}
					}
				}
			}
			stage ('Docker Deployment')
			{
				steps
				{
					bat "docker run --name myapphelloworldapp -d -p 7000:8080 gaurav7576/myapp:$BUILD_NUMBER"
				}
			}
		}
}
