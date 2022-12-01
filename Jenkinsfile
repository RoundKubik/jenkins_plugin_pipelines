#!/usr/bin/env groovy
pipeline {
    agent any

    stages {
        stage("Prepare") {
            steps {
                bitbucketStatusNotify buildState: "INPROGRESS"
            }
        }

        stage("Build and start test image") {
            steps {
                echo 'sh "docker-composer build"'
                echo 'sh "docker-compose up -d"'
                waitUntilServicesReady
            }
        }

        stage("Run tests") {
            steps {
                echo 'sh "docker-compose exec -T php-fpm composer --no-ansi --no-interaction tests-ci"'
                echo 'sh "docker-compose exec -T php-fpm composer --no-ansi --no-interaction behat-ci"'
            }
        }

        stage("Determine new version") {
            when {
                branch "pipeline_2"
            }

            steps {
                script {
                    echo 'env.DEPLOY_VERSION = sh(returnStdout: true, script: "docker run --rm -v \'${env.WORKSPACE}\':/repo:ro softonic/ci-version:0.1.0 --compatible-with package.json").trim()'
                    echo 'env.DEPLOY_MAJOR_VERSION = sh(returnStdout: true, script: "echo \'${env.DEPLOY_VERSION}\' | awk -F\'[ .]\' \'{print \$1}\'").trim()'
                    echo 'env.DEPLOY_COMMIT_HASH = sh(returnStdout: true, script: "git rev-parse HEAD | cut -c1-7").trim()'
                    echo 'env.DEPLOY_BUILD_DATE = sh(returnStdout: true, script: "date -u +\'%Y-%m-%dT%H:%M:%SZ\'").trim()'

                    echo 'env.DEPLOY_STACK_NAME = "${env.STACK_PREFIX}-v${env.DEPLOY_MAJOR_VERSION}"'
                    echo 'env.IS_NEW_VERSION = sh(returnStdout: true, script: "[ \'${env.DEPLOY_VERSION}\' ] && echo \'YES\'").trim()'
                }
            }
        }

        stage("Create new version") {
            when {
                branch "pipeline_2"
            }

            steps {
                script {
                    echo 'git pushing'
                }

                echo 'sh "docker login -u=$REGISTRY_AUTH_USR -p=$REGISTRY_AUTH_PSW ${env.REGISTRY_ADDRESS}"'
                echo 'sh "docker-compose -f ${env.COMPOSE_FILE} build"'
                echo 'sh "docker-compose -f ${env.COMPOSE_FILE} push"'
            }
        }

        stage("Deploy to production") {
            agent any

            when {
                branch "pipeline_2"
            }

            steps {
                echo 'sh "docker login -u=$REGISTRY_AUTH_USR -p=$REGISTRY_AUTH_PSW ${env.REGISTRY_ADDRESS}"'
                echo 'sh "docker stack deploy ${env.DEPLOY_STACK_NAME} -c ${env.COMPOSE_FILE} --with-registry-auth"'
            }

            post {
                success {
                    echo 'message about success'
                }

                failure {
                    echo 'message about failure'
                }
            }
        }
    }

    post {
        always {
            echo 'sh "docker-compose down || true"'
        }

        success {
            bitbucketStatusNotify buildState: "SUCCESSFUL"
        }

        failure {
            bitbucketStatusNotify buildState: "FAILED"
        }
    }
}
