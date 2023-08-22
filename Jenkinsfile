pipeline {
  agent any 
  tools {
    maven 'Maven'
  }
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
    
    stage ('Check-Git-Secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'docker run gesellix/trufflehog --json https://github.com/ddsperera/webapp.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
    
     stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://github.com/yaoit8791/ACS-test/main/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
        
      }
    }
    
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    }
    
   
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }
    
    
    stage ('DAST') {
      steps {
        sshagent(['DAST-server']) {
         sh 'ssh -o  StrictHostKeyChecking=no ubuntu@100.26.52.82 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://54.90.148.114:8080/webapp/" || true'
        }
      }
    }
    
    
    
    
  }
}
