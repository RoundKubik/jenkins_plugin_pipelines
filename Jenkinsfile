#!/usr/bin/env groovy
pipeline {
  agent any

  stages {
    stage("Build") {
      steps {
        echo 'sh \'mvn -v\''
      }
    }

    stage("Testing") {
      parallel {
        stage("Unit Tests") {
          agent any
          steps {
            echo 'sh \'java -version\''
          }
        }
        stage("Functional Tests") {
          agent any
          steps {
            echo 'sh \'java -version\''
          }
        }
        stage("Integration Tests") {
          steps {
            echo 'sh \'java -version\''
          }
        }
      }
    }

    stage("Deploy") {
      steps {
        echo "Deploy!"
      }
    }
  }
}
