#!/usr/bin/env groovy
pipeline {
    // run on jenkins nodes tha has java 8 label
    agent { label 'java8' }
    // global env variables
    environment {
        EMAIL_RECIPIENTS = 'mahmoud.romeh@test.com'
    }
    stages {

        stage('Build with unit testing') {
            steps {
                // Run the maven build
                script {
                    // Get the Maven tool.
                    // ** NOTE: This 'M3' Maven tool must be configured
                    // **       in the global configuration.
                    echo 'Pulling...' + env.BRANCH_NAME
                    def mvnHome = tool 'Maven 3.3.9'
                    if (isUnix()) {
                        def targetVersion = getDevVersion()
                        print 'target build version...'
                        print targetVersion
                        echo 'sh "\'${mvnHome}/bin/mvn\' -Dintegration-tests.skip=true -Dbuild.number=${targetVersion} clean package"'
                        echo 'def pom = readMavenPom file: \'pom.xml\''
                        // get the current development version
                        echo 'developmentArtifactVersion = "${pom.version}-${targetVersion}"'
                        echo 'print pom.version'
                        // execute the unit testing and collect the reports
                        echo 'junit \'**//*target/surefire-reports/TEST-*.xml\''
                        echo 'archive \'target*//*.jar\''
                    } else {
                        echo 'bat(/"${mvnHome}\bin\mvn" -Dintegration-tests.skip=true clean package/)'
                        echo 'def pom = readMavenPom file: \'pom.xml\''
                        echo 'print pom.version'
                        echo 'junit \'**//*target/surefire-reports/TEST-*.xml\''
                        echo 'archive \'target*//*.jar\''
                    }
                }

            }
        }
        stage('Integration tests') {
            // Run integration test
            steps {
                script {
                    def mvnHome = tool 'Maven 3.3.9'
                    if (isUnix()) {
                        // just to trigger the integration test without unit testing
                        echo 'sh "\'${mvnHome}/bin/mvn\'  verify -Dunit-tests.skip=true"'
                    } else {
                        echo 'bat(/"${mvnHome}\bin\mvn" verify -Dunit-tests.skip=true/)'
                    }

                }
                // cucumber reports collection
                echo 'cucumber buildStatus: null, fileIncludePattern: \'**/cucumber.json\', jsonReportDirectory: \'target\', sortingMethod: \'ALPHABETICAL\''
            }
        }
        stage('Sonar scan execution') {
            // Run the sonar scan
            steps {
                script {
                    def mvnHome = tool 'Maven 3.3.9'
                    withSonarQubeEnv {

                        echo 'sh "\'${mvnHome}/bin/mvn\'  verify sonar:sonar -Dintegration-tests.skip=true -Dmaven.test.failure.ignore=true"'
                    }
                }
            }
        }
        // waiting for sonar results based into the configured web hook in Sonar server which push the status back to jenkins
        stage('Sonar scan result check') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    retry(3) {
                        script {
                            def qg = waitForQualityGate()
                            if (qg.status != 'OK') {
                                error "Pipeline aborted due to quality gate failure: ${qg.status}"
                            }
                        }
                    }
                }
            }
        }
        stage('Development deploy approval and deployment') {
            steps {
                script {
                    if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
                        timeout(time: 3, unit: 'MINUTES') {
                            // you can use the commented line if u have specific user group who CAN ONLY approve
                            //input message:'Approve deployment?', submitter: 'it-ops'
                            input message: 'Approve deployment?'
                        }
                        timeout(time: 2, unit: 'MINUTES') {
                            //
                            if (developmentArtifactVersion != null && !developmentArtifactVersion.isEmpty()) {
                                // replace it with your application name or make it easily loaded from pom.xml
                                def jarName = "application-${developmentArtifactVersion}.jar"
                                echo "the application is deploying ${jarName}"
                                // NOTE : CREATE your deployemnt JOB, where it can take parameters whoch is the jar name to fetch from jenkins workspace
                                echo 'build job: \'ApplicationToDev\', parameters: [[$class: \'StringParameterValue\', name: \'jarName\', value: jarName]]'
                                echo 'the application is deployed !'
                            } else {
                                error 'the application is not  deployed as development version is null!'
                            }

                        }
                    }
                }
            }
        }
        stage('DEV sanity check') {
            steps {
                // give some time till the deployment is done, so we wait 45 seconds
                sleep(45)
                script {
                    if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
                        timeout(time: 1, unit: 'MINUTES') {
                            script {
                                def mvnHome = tool 'Maven 3.3.9'
                                //NOTE : if u change the sanity test class name , change it here as well
                                echo 'sh "\'${mvnHome}/bin/mvn\' -Dtest=ApplicationSanityCheck_ITT surefire:test"'
                            }

                        }
                    }
                }
            }
        }
        stage('Release and publish artifact') {
            when {
                // check if branch is master
                branch 'master'
            }
            steps {
                // create the release version then create a tage with it , then push to nexus releases the released jar
                script {
                    def mvnHome = tool 'Maven 3.3.9' //
                    if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
                        def v = getReleaseVersion()
                        releasedVersion = v;
                        if (v) {
                            echo "Building version ${v} - so released version is ${releasedVersion}"
                        }

                        echo 'sh "\'${mvnHome}/bin/mvn\' -Dmaven.test.skip=true  versions:set  -DgenerateBackupPoms=false -DnewVersion=${v}"'
                        echo 'sh "\'${mvnHome}/bin/mvn\' -Dmaven.test.skip=true clean deploy"'

                    } else {
                        error "Release is not possible. as build is not successful"
                    }
                }
            }
        }
        stage('Deploy to Acceptance') {
            when {
                // check if branch is master
                branch 'master'
            }
            steps {
                script {
                    if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
                        timeout(time: 3, unit: 'MINUTES') {
                            //input message:'Approve deployment?', submitter: 'it-ops'
                            input message: 'Approve deployment to UAT?'
                        }
                        timeout(time: 3, unit: 'MINUTES') {
                            //  deployment job which will take the relasesed version
                            if (releasedVersion != null && !releasedVersion.isEmpty()) {
                                // make the applciation name for the jar configurable
                                def jarName = "application-${releasedVersion}.jar"
                                echo "the application is deploying ${jarName}"
                                // NOTE : DO NOT FORGET to create your UAT deployment jar , check Job AlertManagerToUAT in Jenkins for reference
                                // the deployemnt should be based into Nexus repo
                                echo 'build job: \'AApplicationToACC\', parameters: [[$class: \'StringParameterValue\', name: \'jarName\', value: jarName], [$class: \'StringParameterValue\', name: \'appVersion\', value: releasedVersion]]'
                                echo 'the application is deployed !'
                            } else {
                                error 'the application is not  deployed as released version is null!'
                            }

                        }
                    }
                }
            }
        }
        stage('ACC E2E tests') {
            when {
                // check if branch is master
                branch 'master'
            }
            steps {
                // give some time till the deployment is done, so we wait 45 seconds
                sleep(45)
                script {
                    if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
                        timeout(time: 1, unit: 'MINUTES') {

                            script {
                                def mvnHome = tool 'Maven 3.3.9'
                                // NOTE : if you change the test class name change it here as well
                                echo 'sh "\'${mvnHome}/bin/mvn\' -Dtest=ApplicationE2E surefire:test"'
                            }

                        }
                    }
                }
            }
        }
    }
    post {
        // Always runs. And it runs before any of the other post conditions.
        always {
            // Let's wipe out the workspace before we finish!
            deleteDir()
        }
        success {
            echo 'sendEmail("Successful");'
        }
        unstable {
            echo 'sendEmail("Unstable");'
        }
        failure {
            echo 'sendEmail("Failed");'
        }
    }
}

