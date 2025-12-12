pipeline {
  agent any

  environment {
    // set names matching Jenkins Global Tool Configuration or use full path
    JAVA_HOME = tool name: 'jdk11', type: 'jdk'
    MAVEN_HOME = tool name: 'maven3', type: 'maven'
    // If you need to use GitHub token or Nexus, refer by credentialsId
    GIT_CREDENTIALS = 'github-pat'   // optional: used if you need to push or use APIs
    // NEXUS_CREDS = 'nexus-creds'    // optional
  }

options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    timestamps()
}


  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        withEnv(["PATH+MAVEN=${MAVEN_HOME}/bin", "JAVA_HOME=${JAVA_HOME}"]) {
          // run mvn clean package (runs tests by default)
          sh "${MAVEN_HOME}/bin/mvn -B -DskipTests=false clean package"
        }
      }
    }

    stage('Unit tests & results') {
      steps {
        // publish JUnit test reports
        junit '**/target/surefire-reports/*.xml'
      }
      post {
        always {
          // collect surefire/coverage artifacts if needed
          archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
        }
      }
    }

    stage('Static analysis (optional)') {
      when {
        expression { fileExists('pom.xml') }
      }
      steps {
        echo "Run optional SonarQube / Checkstyle steps here"
      }
    }

    stage('Archive') {
      steps {
        archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
      }
    }

    stage('Deploy (optional)') {
      when {
        branch 'main'
      }
      steps {
        echo "Add deployment steps (scp, Nexus upload, Helm, etc.)"
        // Example: upload to Nexus using mvn deploy - settings.xml with server creds
      }
    }
  }

  post {
    success {
      echo "Build completed successfully."
      // e.g. notify Slack / email
    }
    failure {
      echo "Build failed."
    }
    always {
      cleanWs()
    }
  }
}

