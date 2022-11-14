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
 
   stage ('preparing') {
      steps {
         sh 'dpkg -s unzip || sudo apt install unzip'
         sh 'dpkg -s npm || sudo apt install npm -y'
         sh 'gem list -i "^bundler-audit$" || sudo gem install --http-proxy http://proxy.compassplus.ru:3128 bundler-audit'
         sh 'gem list -i "^yarn$" || sudo gem install --http-proxy http://proxy.compassplus.ru:3128 yarn'  
         sh 'dpkg -s npm || wget -qO- https://get.pnpm.io/install.sh | sh - || true '
         sh 'bundle-audit update'
         sh 'service docker status | grep running || sudo service docker start '
      }
    }    

    stage ('SCA_sh') {
      steps {
         sh 'wget https://github.com/jeremylong/DependencyCheck/releases/download/v7.3.0/dependency-check-7.3.0-release.zip '
         sh 'unzip -u dependency-check-7.3.0-release.zip'
         sh './dependency-check/bin/dependency-check.sh --project "DVWA" --scan "/home/kali/DVWA" --proxyserver proxy.compassplus.ru --proxyport 3128 '
      }
    }
    
     stage ('SCA_docker') {
      steps {
         sh 'wget https://github.com/prince74igor/webapp_pub/blob/master/owasp-dependency-check.sh && cp webapp_pub/owasp-dependency-check.sh ~/DVWA && cd ~/DVWA && cp sudo sh owasp-dependency-check.sh '
      }
    }
    
    stage ('SAST_maven') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'sudo docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest'
          sh 'dpkg -s maven || sudo apt install maven -y'
          sh 'sudo docker ps | grep sonar || sudo docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest'
          sh 'wget https://github.com/prince74igor/webapp_pub.git '
          sh 'cd ~/webapp_pub && mvn sonar:sonar -Dsonar.projectKey=12312312sdsdf \
                                       -Dsonar.sources=. \
                                       -Dsonar.host.url=http://127.0.0.1:9000 \
                                       -Dsonar.login=sqp_2b05ee5c9d069b7222144f8a8cf534047493dd28'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    }
    
    stage ('SAST_source') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'sudo docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest'
          sh 'dpkg -s maven || sudo apt install maven -y'
          sh 'sudo docker ps | grep sonar || sudo docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest'
          sh 'wget https://github.com/prince74igor/webapp_pub.git '
          sh 'cd ~/webapp_pub && mvn sonar:sonar -Dsonar.projectKey=12312312sdsdf \
                                       -Dsonar.sources=. \
                                       -Dsonar.host.url=http://127.0.0.1:9000 \
                                       -Dsonar.login=sqp_2b05ee5c9d069b7222144f8a8cf534047493dd28'
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
