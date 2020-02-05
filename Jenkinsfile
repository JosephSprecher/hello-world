node {
    def SBT = "sbt -Dsbt.log.noformat=true -mem 2048"
    def ENABLE_NOTIFICATIONS = false

    if (ENABLE_NOTIFICATIONS) slackSend(channel: "#jenkins", color: '#36A64F', message: "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")

    try {
        stage('Git Checkout') {
            checkout scm
            sh "env"
        }

        docker.image('registry.marathon.mesos:5000/sbt/sbt:1.2.8').inside('') {

            try {
                stage('Static Check') {
                    echo 'Running static checks..'
                    echo sh "${SBT} fmtCheck"
                }

                stage('Build') {
                    echo 'Building..'
                    echo sh "${SBT} clean '+ compile'"
                }

                stage('Unit Test') {
                    echo 'Running Unit Tests..'
                    echo sh "${SBT} coverage '+ test'"
                }

                stage('Integration Test') {
                    echo 'Running Integration Tests..'
                    echo sh "${SBT} coverage '+ it:test'"
                }

                stage('Code Coverage') {
                    echo 'Generating Coverage Reports..'
                    echo sh "${SBT} coverageReport coverageAggregate"
                }

                stage('Publish Artifacts') {
                    if (env.BRANCH_NAME == "master") {
                      echo 'Publishing Artifacts..'
                      echo sh "${SBT} -Drelease=true '+ publish'"
                    } else {
                      echo 'Not in master branch. Not publishing.'
                    }
                }

                if (ENABLE_NOTIFICATIONS) slackSend(channel: "#jenkins", color: '#36A64F', message: "${env.JOB_NAME} Succeeded: ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
            } finally {
                stage('Publish Results') {
                    echo junit '**/target/test-reports/TEST-*.xml'
                    echo step([$class: 'ScoveragePublisher', reportDir: 'target/scala-2.12/scoverage-report', reportFile: 'scoverage.xml'])
                }
            }
        }
    } catch (e) {
        if (ENABLE_NOTIFICATIONS) slackSend(channel: "#jenkins", color: '#f90005', message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        throw e
    }
}
