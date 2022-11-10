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
 
    stage ('Source Composition Analysis') {
      steps {
         sh 'dpkg -s unzip || sudo apt install unzip'
         sh 'dpkg -s npm || sudo apt install npm -y'
         sh 'gem list -i "^bundler-audit$" || sudo gem install --http-proxy http://proxy.compassplus.ru:3128 bundler-audit'
         sh 'bundle-audit update'
         sh 'gem list -i "^yarn$" || sudo gem install --http-proxy http://proxy.compassplus.ru:3128 yarn'  
         sh 'wget -qO- https://get.pnpm.io/install.sh | sh - || true '
         sh 'unzip -u dependency-check-7.3.0-release.zip && cd dependency-check/bin && ./dependency-check.sh --project "My App Name" --scan "/home/kali/DependencyCheck/" && cp -R . /home/kali '
      }
    }
    
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'dpkg -s maven || sudo apt install maven -y'          
          sh 'mvn clean install'
          sh 'mvn sonar:sonar -Dsonar.login=myAuthenticationToken'
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
                sh 'scp -o StrictHostKeyChecking=no target/*.war kali@192.168.129:/prod/apache-tomcat-8.5.39/webapps/webapp.war'
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
        sh ' "rm trufflehog" || true && docker run gesellix/trufflehog --json https://github.com/prince74igor/webapp_pub.git > trufflehog" || true && cat trufflehog" '
        }
      }
    }      
    
  }
}
