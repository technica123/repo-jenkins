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
                checkout scm
            }
        }

        // baaki stages...
    }
}
