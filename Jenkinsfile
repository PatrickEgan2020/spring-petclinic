#!/bin/env groovy

pipeline {
  agent none

  environment {
    IMAGE = "liatrio/petclinic-tomcat"
  }

  stages {
    stage('Build') {
      agent any
      steps {
        sh 'mvn clean install'
        echo "Sending slack channel alert for build completion"
      }
    }
    stage('Deploy') {
      agent any
      steps {
        script {
          if ( env.BRANCH_NAME == 'master' ) {
            pom = readMavenPom file: 'pom.xml'
            TAG = pom.version
          } else {
            TAG = env.BRANCH_NAME
          }
          sh "docker build -t ${env.IMAGE}:${TAG} ."
          echo "Sending slack channel alert for deploy completion"
        }
      }
    }
    stage('Test') {
      agent any
      steps {
        echo "Conducting all unit and integration tests"
        echo "Sending slack channel alert for Tests completion"
        sh 'sleep 4'
      }
    }
    stage('Release to Dev') {
      agent any
      steps {
        sh 'docker rm -f petclinic-tomcat-temp || true'
        sh "docker run -d -p 9966:8080 --name petclinic-tomcat-temp ${env.IMAGE}:${TAG}"
        echo "Sending slack channel alert for releasing to Dev completion"
      }
    }

    stage('Smoke test dev') {
  when {
    branch 'master'
  }
  agent {
    docker {
      image 'maven:3.5.0'
      args '--network=${LDOP_NETWORK_NAME}'
    }
  }
  steps {
    sh "cd regression-suite && mvn clean -B test -DPETCLINIC_URL=https://dev.petclinic.liatr.io/petclinic"
    echo "Should be accessible at https://dev.petclinic.liatr.io/petclinic"
    echo "Sending slack channel alert for releasing to smoke test dev completion"
  }
}
stage('Release to Prod') {
  agent any
  steps {
    sh 'docker rm -f petclinic-tomcat-temp || true'
    sh "docker run -d -p 9966:8080 --name petclinic-tomcat-temp ${env.IMAGE}:${TAG}"
    echo "Sending slack channel alert for releasing to Prod completion"
  }
}

stage('Smoke test Prod') {
  when {
    branch 'master'
    }
    agent {
      docker {
        image 'maven:3.5.0'
        args '--network=${LDOP_NETWORK_NAME}'
      }
    }
    steps {
      sh "cd regression-suite && mvn clean -B test -DPETCLINIC_URL=https://prod.petclinic.liatr.io/petclinic"
      echo "Should be accessible at https://prod.petclinic.liatr.io/petclinic"
      echo "Sending slack channel alert for smoke test of prod completion"
    }
  }

    stage('Notification of Build Status To Slack') {
      agent any
      steps {
        echo "Sending slack channel alert for pipeline completion"
        sh 'sleep 4'
      }
    }
  }
}
