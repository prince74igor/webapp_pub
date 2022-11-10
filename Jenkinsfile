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
         sshCommand remote: remote, command:  'wget "https://github.com/jeremylong/DependencyCheck/releases/download/v7.3.0/dependency-check-7.3.0-release.zip" || true '
         sshCommand remote: remote, command:  'dpkg -s unzip || sudo apt install unzip'
         sshCommand remote: remote, command:  'wget -qO- https://get.pnpm.io/install.sh | sh - || true '
         sshCommand remote: remote, command:  'gem list -i "^bundler-audit$" || sudo gem install bundler-audit && bundle-audit update'
         sshCommand remote: remote, command:  'gem list -i "^yarn$" || sudo gem install yarn '  
         sshCommand remote: remote, command:  'unzip -u dependency-check-7.3.0-release.zip && cd dependency-check/bin && ./dependency-check.sh --project "My App Name" --scan "/home/kali/DependencyCheck/" && cp -R . /home/kali '
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
