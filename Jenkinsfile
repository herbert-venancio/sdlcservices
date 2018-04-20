#!groovy
import groovy.transform.Field

@Library("liferay-sdlc-jenkins-lib") import static org.liferay.sdlc.SDLCPrUtilities.*

@Field final gitRepository = 'objective-solutions/sdlcservices'
@Field final projectName = "SDLCSERV"
@Field final projectKey  = "sdlcservices"

def onError() {
	handleError(gitRepository, "gabriel.takeuchi@objective.com.br", "github_objective-solutions_sdlcservices")
}

node ("sdlcservices||shared-pool") {
	try {
		stage('Checkout') {
			checkout scm
		}

		stage('Setup') {
			if (fileExists("bundles"))
				deleteRecursive: "bundles"

			prInit(projectKey, projectName);
			
			gradlew 'clean'
		}

		stage('Init Bundle') {
			gradlew 'initBundle'
		}

		stage('Build') {
			try {
				gradlew 'build -x test'	
			} catch (exc) {
				onError()
				throw exc
			}
		}

		stage('Test') {
			try {
				gradlew 'test'
			} catch (exc) {
				onError()
				throw exc
			} finally {
				junit '**/build/test-results/test/*.xml'
			}
		}

		stage('Sonar') {
			if (isPullRequest()) {
				println "Will evaluate the Pull Request"
				sonarqube "-Dsonar.analysis.mode=preview -Dsonar.github.pullRequest=${CHANGE_ID} -Dsonar.github.oauth=${GithubOauth} -Dsonar.github.repository=${gitRepository}"
			}
			else
				sonarqube ""
		}
	}finally {
		stage('Cleanup') {
			dir(workspace) {
				deleteDir();
			}
		}
	}
}
