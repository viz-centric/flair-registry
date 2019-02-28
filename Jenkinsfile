node {
    def server = Artifactory.server "artifactory"
    def rtMaven = Artifactory.newMavenBuild()
    def buildInfo
    def image

    try {
        notifySlack()

        stage ('Checkout') {
            checkout scm
        }

        stage ('Artifactory configuration') {
            rtMaven.tool = "maven" // Tool name from Jenkins configuration
            rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local', server: server
            rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot', server: server
            rtMaven.deployer.deployArtifacts = false // Disable artifacts deployment during Maven run

            buildInfo = Artifactory.newBuildInfo()
        }

        stage ('Test') {
            // TODO: Enable tests
            //rtMaven.run pom: 'pom.xml', goals: 'clean test'
            sh 'echo "Enable tests"'
        }

        stage ('Package') {
            rtMaven.run pom: 'pom.xml', goals: 'clean package -U -Pprod -DskipTests', buildInfo: buildInfo
        }

        stage ('Deploy') {
            rtMaven.deployer.deployArtifacts buildInfo
            fingerprint 'target/*.war'
        }

        stage ('Build Image') {
            app = docker.build("flair-registry", "./target")
        }

        stage ('Register Image') {
                sh 'echo "register docker image"'
            docker.withRegistry('http://localhost:10003') {
                app.push("${env.BUILD_NUMBER}")
                app.push("latest")
            }
        }

        stage ('Publish build info') {
            server.publishBuildInfo buildInfo
        }

        stage ('Start container'){
            withCredentials([usernamePassword(credentialsId: 'bitbucket-manoj-cred', passwordVariable: 'BITBUCKET_PASS', usernameVariable: 'BITBUCKET_USER')]) {
                sh '''
                docker-compose -f src/main/docker/deploy.yml down
                env BITBUCKET_URI="https://manoj10@bitbucket.org/vizcentricadmin/flair-configuration.git" docker-compose -f src/main/docker/deploy.yml up -d
                '''
            }
        }
    } catch (e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        notifySlack(currentBuild.result)
    }
}

def notifySlack(String buildStatus = 'STARTED') {
    // Build status of null means success.
    buildStatus = buildStatus ?: 'SUCCESS'

    def color

    if (buildStatus == 'STARTED') {
        color = '#D4DADF'
    } else if (buildStatus == 'SUCCESS') {
        color = '#BDFFC3'
    } else if (buildStatus == 'UNSTABLE') {
        color = '#FFFE89'
    } else {
        color = '#FF9FA1'
    }

    def msg = "${buildStatus}: `${env.JOB_NAME}` #${env.BUILD_NUMBER}:\n${env.BUILD_URL}"

    slackSend(color: color, message: msg)
}
