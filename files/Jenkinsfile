//Declarative pipeline V2 - JIRA 'DOR-59'
@Library('SharedLibraries')_
pipeline {
agent none

//tools used in build
tools {
maven 'mvn-3-5-0'
jdk 'jdk-1-8'
}


options {
buildDiscarder(logRotator(numToKeepStr:'5'))
timeout(time: 5, unit: 'MINUTES')
}

//options {
//  timeout(5, HOURS)
//}

stages {

  stage("preparing environment") {
  agent any
    // stage must have a steps block containing at least one step.
    // def response = httpRequest 'http://www.google.co.uk'
   steps {
    // response = "httpRequest consoleLogResponseBody: true, ignoreSslErrors: true, outputFile: 'test', responseHandle: 'NONE', url: 'http://ci.agile-operations.co.uk:8080/login', validResponseCodes: '200'"
     echo "prep stage"
     //sh 'mvn clean'

        //println("Status: "+response.validResponseCodes)
        //println("Content: "+response.content)
     }
   }

  stage('Pull source code') {
    agent any
    steps {
	    checkout scm
      echo "Pull Stage"
      script {
      result = sh (script: "git log -1 | grep -o -m 1 'SPRING-[0-9][0-9]*'", returnStdout: true).trim()
      if (result != 0) {
        echo result
      } else {
        echo "Jira ID Number not found"
        }
        }
     }
  }

  stage('build source code ') {
  agent any
    steps {
      //send build started notifications
      echo "build stage"
      sh 'mvn package'
    }
  }

  stage('test source code ') {
    steps {
      echo "Test Stage"

    }
  }


  stage('JIRA') {
  agent any
    steps{
    script{
    try{
      TransitionJira('191' , result)
      script {
      transitioni = 'Send to UAT'
      def transitions = jiraGetIssueTransitions idOrKey: result, site: 'site'
      transitiontext = transitions.data.toString()

      transitionid = sh (script: """echo '$transitiontext' | grep -o '[0-9][0-9][0-9], name:$transitioni' | head -c3 """, returnStdout: true)
      echo transitionid
      }
    }
    catch (err){
    echo "err"
    return True
    }
    }
  }
  }

  stage('Create') {
  agent any
  when {
    branch "Development"
  }
    steps {
      slackSend (color: '#FFFF00', message: "STARTED: Job ${env.JOB_NAME} on branch: '${env.BRANCH_NAME} Build Number: [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      //echo "archive Stage"
      archive 'target/*.jar'
    }
  }

  stage('deploy') {
    agent any
    when {
      branch "CI"
    }
      steps {
        slackSend (color: '#FFFF00', message: "STARTED: Job ${env.JOB_NAME} on branch: '${env.BRANCH_NAME} Build Number: [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        //echo "archive Stage"
        echo "deploying to CI"
      }
  }

  //communication
  }

  post {
    failure {
      NewJira('SPRING', 'New JIRA Created due to Jenkins Build failure.', 'New JIRA Created due to Jenkins Build failure.', 'Task')
      }
    }
}
