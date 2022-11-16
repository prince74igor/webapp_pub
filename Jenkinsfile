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
 
   stage ('preparing_tools') {
      steps {
         sh 'dpkg -s unzip || sudo apt install unzip'
         sh 'dpkg -s npm || sudo apt install npm -y'
         sh 'gem list -i "^bundler-audit$" || sudo gem install --http-proxy http://proxy.compassplus.ru:3128 bundler-audit'
         sh 'gem list -i "^yarn$" || sudo gem install --http-proxy http://proxy.compassplus.ru:3128 yarn'  
         sh 'dpkg -s npm || wget -qO- https://get.pnpm.io/install.sh | sh - || true '
         sh 'bundle-audit update'        
      }
    }  
    
       stage ('preparing_docker') {
      steps {
         sh 'dpkg -s docker || sudo apt install -y docker.io && sudo systemctl enable docker --now'
         sh 'la | grep webapp_pub || git clone https://github.com/prince74igor/webapp_pub.git && bash ~/webapp_pub/docker_proxy'
      }
    } 
       
       stage ('preparing_DVWA&WebApp') {
      steps {
         sh 'git clone https://github.com/digininja/DVWA.git --progress'
         sh 'git clone https://github.com/prince74igor/webapp_pub.git --progress'
      }
    }
    
    stage ('SCA_sh') {
      steps {
         sh 'wget https://github.com/jeremylong/DependencyCheck/releases/download/v7.3.0/dependency-check-7.3.0-release.zip '
         sh 'unzip -u dependency-check-7.3.0-release.zip'
         sh 'bash ~/dependency-check/bin/dependency-check.sh --project "DVWA" --scan "~/DVWA" --proxyserver proxy.compassplus.ru --proxyport 3128 '
      }
    }
    
     stage ('SCA_docker') {
      steps {
         sh 'cp ~/webapp_pub/owasp-dependency-check.sh ~/DVWA && bash ~/DVWA/owasp-dependency-check.sh'
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
    
    stage ('SAST_source_docker') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'sudo docker ps | grep sonar || sudo docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest'
          sh 'sudo docker run \
                                         --rm \
                                         -e SONAR_HOST_URL="http://172.17.0.2:9000" \
                                         -e SONAR_SCANNER_OPTS="-Dsonar.projectKey=a4" \
                                         -e SONAR_LOGIN="sqp_b57a4d6e622456352ccb502efbc4e0be3ecf5bce" \
                                         -e SONAR_JAVA_BINARIES="/home/kali/TW_ACS/" \
                                         -v "/home/kali/DVWA:/usr/src" \
                                         sonarsource/sonar-scanner-cli'
        }
      }
    }
    
      stage ('SAST_source_sh') {
         steps {
           withSonarQubeEnv('sonar') {
              sh 'wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.7.0.2747-linux.zip'
              sh 'unzip -u sonar-scanner-cli-4.7.0.2747-linux.zip'
              sh './sonar-scanner-4.7.0.2747-linux/bin/sonar-scanner \
                 -Dsonar.projectKey=TW_ACS \
                 -Dsonar.sources=/home/kali/TW_ACS/ \
                 -Dsonar.host.url=http://10.7.2.224:9000 \
                 -Dsonar.login=sqp_08c08cc4e5cac12dca23a6886fde7bb5e47f9f4e \
                 -Dsonar.java.binaries=/home/kali/TW_ACS/ '
        }
      }
    }
    
    stage ('Build') {
      steps {
      sh 'cd ~/webapp_pub && mvn clean package'
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
