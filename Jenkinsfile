pipeline {
    agent none
    stages {
        stage('Build pipline_1') {
            steps {
                echo 'Building pipline_1 ...'
            }
        }
        stage('Run Tests pipeline_1') {
            parallel {
                stage('Test On Windows') {
                    agent {
                        label "windows"
                    }
                    steps {
                        echo 'for example bat "run-tests.bat"'
                    }
                    post {
                        always {
                            echo 'junit "**/TEST-*.xml"'
                        }
                    }
                }
                stage('Test On Linux') {
                    agent {
                        label "linux"
                    }
                    steps {
                        echo 'sh "run-tests.sh"'
                    }
                    post {
                        always {
                            echo 'junit "**/TEST-*.xml"'
                        }
                    }
                }
            }
        }
        stage('Deploy pipeline_1') {
            steps {
                echo 'Deploying pipeline_1'
            }
        }
    }
}
