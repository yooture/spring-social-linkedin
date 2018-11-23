#!/usr/bin/env groovy
@Library("yooture")
import yooture.jenkins.yooture.YooBuild

def yooBuild = new YooBuild(this)


pipeline {

  options {
    // General Jenkins job properties
    buildDiscarder(logRotator(numToKeepStr:'10'))
    // "wrapper" steps that should wrap the entire build execution
    timeout(time: 10, unit: 'MINUTES')
  }

  agent {
      docker {
          image 'yooture/yoo-java-build'
          args '-v $HOME/.m2:/root/.m2'
      }
  }
  stages {
  
    stage('build') {    
      when { not { branch 'master' } }
      steps {
        withMaven(mavenSettingsConfig: 'DEFAULT_YOOTRE_MAVEN_SETTINGS', options: [artifactsPublisher(disabled: true)]) {
          sh """
#!/bin/sh

./gradlew build

find . -name "spring-social-linkedin*jar"

export PATH=$MVN_CMD_DIR:$PATH && mvn -f wagon-webdav-pom.xml deploy:deploy-file -DgroupId=org.springframework.social \
  -DartifactId=spring-social-linkedin \
  -Dversion=1.0.3-YOOTURE \
  -Dpackaging=jar \
  -Dfile=spring-social-linkedin/build/libs/spring-social-linkedin-1.0.3.BUILD-SNAPSHOT.jar \
  -DrepositoryId=yooture-private-release-repository \
  -Durl=http://repo.yooture.net/repository/yooture-releases/

          """
        }
      }
    }  

  }

  post {
    always {
      junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
      cleanWs()
    }
    // failure {
    //   hipchatSend color: 'RED', room: 'yooture', notify: true, message: "@all build failed, please fix!!! ${currentBuild.displayName}: ${currentBuild.result} - ${currentBuild.absoluteUrl}"
    // }
    // unstable {
    //   hipchatSend color: 'RED', room: 'yooture', notify: true, message: "@all build unstable, please fix!!! ${currentBuild.displayName}: ${currentBuild.result} - ${currentBuild.absoluteUrl}"
    // }
  }
}
