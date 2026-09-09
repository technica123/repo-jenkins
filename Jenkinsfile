pipeline {

    agent any

    parameters {
        booleanParam(
            name: 'SKIP_STABILITY',
            defaultValue: false,
            description: 'Skip code stability scan'
        )

        booleanParam(
            name: 'SKIP_QUALITY',
            defaultValue: false,
            description: 'Skip code quality scan'
        )

        booleanParam(
            name: 'SKIP_COVERAGE',
            defaultValue: false,
            description: 'Skip code coverage scan'
        )
    }

    stages {

        stage('Code Checkout') {
            steps {
                echo 'Checking out Java project...'
                checkout scm
            }
        }

        stage('Parallel Scans') {
            parallel {

                stage('Code Stability') {
                    when {
                        expression {
                            return !params.SKIP_STABILITY
                        }
                    }

                    steps {
                        echo 'Running Code Stability...'

                        sh 'mvn clean test'
                    }
                }

                stage('Code Quality Analysis') {
                    when {
                        expression {
                            return !params.SKIP_QUALITY
                        }
                    }

                    steps {
                        echo 'Running Code Quality Analysis...'

                        withSonarQubeEnv('SonarQube') {
                            sh '''
                                mvn sonar:sonar \
                                -Dsonar.projectKey=java-ci-project \
                                -Dsonar.projectName=java-ci-project
                            '''
                        }
                    }
                }

                stage('Code Coverage Analysis') {
                    when {
                        expression {
                            return !params.SKIP_COVERAGE
                        }
                    }

                    steps {
                        echo 'Running Code Coverage Analysis...'

                        sh 'mvn test jacoco:report'
                    }
                }
            }
        }

        stage('Generate Reports') {
            steps {
                echo 'Generating test and coverage reports...'

                junit(
                    testResults: '**/target/surefire-reports/*.xml',
                    allowEmptyResults: true
                )

                jacoco(
                    execPattern: '**/target/*.exec',
                    classPattern: '**/target/classes',
                    sourcePattern: '**/src/main/java'
                )

                archiveArtifacts(
                    artifacts: '**/target/*.jar',
                    allowEmptyArchive: true
                )
            }
        }

        stage('Approval') {
            steps {
                input(
                    message: 'Do you approve publishing the artifact?',
                    ok: 'Approve'
                )
            }
        }

        stage('Publish Artifacts') {
            steps {
                echo 'Publishing artifact...'

                sh '''
                    mkdir -p published-artifacts
                    cp target/*.jar published-artifacts/
                '''

                archiveArtifacts(
                    artifacts: 'published-artifacts/*.jar',
                    fingerprint: true
                )

                echo 'Artifact published successfully.'
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully.'

            slackSend(
                channel: '#jenkins',
                message: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER} - Artifact published successfully."
            )

            emailext(
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Pipeline completed successfully.

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Status: SUCCESS

Artifact published successfully.

Build URL:
${env.BUILD_URL}
""",
                to: 'YOUR_EMAIL@gmail.com'
            )
        }

        failure {
            echo 'Pipeline failed.'

            slackSend(
                channel: '#jenkins',
                message: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER} - Pipeline failed."
            )

            emailext(
                subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Pipeline failed.

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Status: FAILURE

Please check Jenkins console output.

Build URL:
${env.BUILD_URL}
""",
                to: 'YOUR_EMAIL@gmail.com'
            )
        }

        aborted {
            echo 'Publication was denied or pipeline was aborted.'

            slackSend(
                channel: '#jenkins',
                message: "ABORTED: ${env.JOB_NAME} #${env.BUILD_NUMBER} - Publication denied or build aborted."
            )

            emailext(
                subject: "ABORTED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Pipeline was aborted.

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Status: ABORTED

Publication was not completed.

Build URL:
${env.BUILD_URL}
""",
                to: 'er.megha006@gmail.com'
            )
        }
    }
}
