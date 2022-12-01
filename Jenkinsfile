#!/bin/env groovy

pipeline {
  agent any

  stages {
    stage('Init') {
      steps {
        echo "version_a: {env.versiona_a}"
        echo sh(script: 'echo \'env|sort\'', returnStdout: true)
        echo "Variable params: ${params}"
        //echo "params.PARAM_1: ${params.PARAM_1}"

      }
    }
    stage('Build') {
      when {
        expression { false }  //disable this stage
      }
      steps {
        echo 'sh \'./mvnw -B clean package -DskipTests\''
        echo 'archive includes: \'**/target/*.jar\''
        echo 'stash includes: \'**/target/*.jar\', name: \'jar\''
      }
    }

  }
}