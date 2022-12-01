#!/usr/bin/env groovy
pipeline {

    /*
     * Run everything on an existing agent configured with a label 'docker'.
     * This agent will need docker, git and a jdk installed at a minimum.
     */
    agent any

    // using the Timestamper plugin we can add timestamps to the console log

    //Use Pipeline Utility Steps plugin to read information from pom.xml into env variables

    stages {
        stage('Build') {
            agent any
            steps {
                echo 'build pipeline 3'
                // using the Pipeline Maven plugin we can set maven configuration settings, publish test results, and annotate the Jenkins console
                echo 'maven configuration'
            }
            post {
                success {
                    // we only worry about archiving the jar file if the build steps are successful
                    echo 'archiveArtifacts(artifacts: \'**/target/*.jar\', allowEmptyArchive: true)'
                }
            }
        }

        stage('Quality Analysis') {
            parallel {
                // run Sonar Scan and Integration tests in parallel. This syntax requires Declarative Pipeline 1.2 or higher
                stage('Integration Test') {
                    agent any  //run this stage on any available agent
                    steps {
                        echo 'Run integration tests here...'
                    }
                }
                stage('Sonar Scan') {
                    agent any
                    steps {
                        echo 'set environment'
                        echo 'sh \'mvn sonar:sonar -Dsonar.login=$SONAR_PSW\''
                    }
                }
            }
        }

        stage('Build and Publish Image') {
            when {
                branch 'pipeline_3'  //only run these steps on the master branch
            }
            steps {
                /*
                 * Multiline strings can be used for larger scripts. It is also possible to put scripts in your shared library
                 * and load them with 'libaryResource'
                 */
                echo 'sh """docker build -t ${IMAGE} .docker tag ${IMAGE} ${IMAGE}:${VERSION}docker push ${IMAGE}:${VERSION}"""'
            }
        }
    }

    post {
        failure {
            // notify users when the Pipeline fails
            echo 'mail to: \'team@example.com\', subject: "Failed Pipeline: ${currentBuild.fullDisplayName}", body: "Something is wrong with ${env.BUILD_URL}"'
        }
    }
}
