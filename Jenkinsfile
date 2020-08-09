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
					sh "mvn install"
				}
			}
			stage ('Unit Testing')
			{
				steps
				{
					echo "build in master branch - 3"
					sh "mvn test"
				}
			}
			stage ('Sonar Analysis')
			{
				steps
				{
					echo "build in master branch - 4"
					withSonarQubeEnv("SonarToken")
					{
						sh "mvn sonar:sonar"
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
						releaseRepo: 'CI-Automation-JAVA',
						snapshotRepo: 'CI-Automation-JAVA'
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
		}
}