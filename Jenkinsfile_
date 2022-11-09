pipeline {
  agent any 
  tools {
    maven '3.8.6'
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
    
    stage('Remote SSH') {
         steps {
           script {
      def remote = [:]
      remote.name = 'kali'
      remote.host = '10.7.2.224'
      remote.user = 'kali'
      remote.password = 'kali'
      remote.allowAnyHosts = true
      sshCommand remote: remote, command: "rm owasp-dependency-check* || true"
      sshCommand remote: remote, command: "wget https://raw.githubusercontent.com/prince74igor/webapp_pub/master/owasp-dependency-check.sh"
      sshCommand remote: remote, command: "chmod +x owasp-dependency-check.sh"
      sshCommand remote: remote, command: "sudo ./owasp-dependency-check.sh --purge"
      sshCommand remote: remote, command: "cat OWASP-Dependency-Check/reports/dependency-check-report.xml"
        }
                     }
                }
      
    
    stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp-dependency-check* || true'
         sh 'wget "https://raw.githubusercontent.com/prince74igor/webapp_pub/master/owasp-dependency-check.sh" '
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
    
    stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war drake@10.211.55.9:/prod/apache-tomcat-8.5.39/webapps/webapp.war'
              }      
           }       
    }
    
    
    stage ('DAST') {
      steps {
        sshagent(['zap']) {
         sh '"docker run -t owasp/zap2docker-stable zap-baseline.py -t http://10.211.55.9:8080/webapp/" || true'
        }
      }
    }
    
        stage ('Check-Git-Secrets') {
      steps {
        sshagent(credentials: ['secrets']) {
        sh 'ssh -o  StrictHostKeyChecking=no drake@10.211.55.9 && "rm trufflehog" || true && docker run gesellix/trufflehog --json https://github.com/prince74igor/webapp_pub.git > trufflehog" || true && cat trufflehog" '
        }
      }
    }      
    
  }
}
