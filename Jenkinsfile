pipeline {
    agent {
        label 'node2'   // 👈 your second node label
    }

    tools {
        jdk 'jdk8'
        maven 'maven3'
    }

    parameters {
        choice(
            name: 'ENV',
            choices: ['dev', 'qa', 'prod'],
            description: 'Select environment'
        )
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/opstree/spring3hibernate.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean verify -Ddependency-check.skip=true'
            }
        }

        stage('Archive Build Artifacts') {
            steps {
                archiveArtifacts artifacts: '**/target/*.war', fingerprint: true
            }
        }

        stage('GitLeaks Security Scan (Final Stage)') {
            steps {
                script {
                    def status = sh(
                        script: '''
                        gitleaks detect \
                          --source . \
                          --report-format json \
                          --report-path gitleaks-report.json
                        ''',
                        returnStatus: true
                    )

                    sh '''
                    if [ ! -f gitleaks-report.json ]; then
                      echo "[]" > gitleaks-report.json
                    fi
                    '''

                    if (status != 0) {
                        echo "⚠️ Secrets detected"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Archive GitLeaks Report') {
            steps {
                archiveArtifacts artifacts: 'gitleaks-report.json', fingerprint: true
            }
        }
    }

    post {
        always {
            echo "Pipeline completed on node2 for ENV: ${params.ENV}"
            cleanWs()
        }
    }
}
