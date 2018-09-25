pipeline {
  agent any
  tools {
    jdk 'oracle-jdk-8'
  }
  options {
    buildDiscarder(logRotator(artifactNumToKeepStr: '5'))
    timeout(time: 1, unit: 'HOURS')
    timestamps()
  }
  //triggers { pollSCM('H/15 * * * *') }
  stages {
    stage('Build') {
      steps {
        sh 'sloccount --duplicates --wide --details --addlangall ./src > sloccount_report.sc'
        sh './mvnw -B clean verify'
        sh './mvnw -B javadoc:javadoc'
      }
      post {
        always {
          checkstyle pattern: '**/checkstyle-result.xml'
          pmd canComputeNew: false, canRunOnFailed: true, defaultEncoding: '', healthy: '', pattern: '**/target/pmd.xml', unHealthy: ''
          warnings canComputeNew: false, canResolveRelativePaths: false, categoriesPattern: '', consoleParsers: [[parserName: 'Java Compiler (javac)']], defaultEncoding: '', excludePattern: '', healthy: '', includePattern: '', messagesPattern: '', unHealthy: ''
          sloccountPublish encoding: '', pattern: 'sloccount_report.sc'
          dry canComputeNew: false, defaultEncoding: '', healthy: '', pattern: '**/target/cpd.xml', unHealthy: ''
          publishHTML([allowMissing: true, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target/site/apidocs', reportFiles: 'index.html', reportName: 'Javadoc', reportTitles: ''])
          junit '**/*-reports/**/*.xml'
          jacoco classPattern: '**/target/**/classes', execPattern: '**/target/jacoco-aggregate.exec'
        }
        success {
          archiveArtifacts "target/*.zip"
        }
      }
    }
  }
  post {
    always {
      step([$class                  : 'Mailer',
            notifyEveryUnstableBuild: true,
            recipients              : emailextrecipients([[$class: 'CulpritsRecipientProvider'],
                                                          [$class: 'RequesterRecipientProvider']])])
    }
  }
}

