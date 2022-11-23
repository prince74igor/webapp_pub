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
    
    stage ('Source Update') {
      steps {
         sh 'cd ~/TW_ACS_GIT/iad-ecacs2 && git pull git@git.compassplus.com:cp/iad-ecacs2.git'
         sh 'cd ~/TW_ACS_GIT/iad-web-app-core && git pull git@git.compassplus.com:cp/iad-web-app-core.git'
         sh 'cd ~/TW_ACS_GIT/iad-js-utils && git pull git@git.compassplus.com:cp/iad-js-utils.git'
      }
    }
    
    stage ('preparing_SCA_sh') {
      steps {
         sh 'VERSION=$(curl -s https://jeremylong.github.io/DependencyCheck/current.txt)'
         sh 'wget https://github.com/jeremylong/DependencyCheck/releases/download/v$VERSION/dependency-check-$VERSION-release.zip -O dependency-check.zip'
         sh 'unzip -u -o dependency-check.zip'
      }
    }  
   

   stage ('SAST_source_docker') { 
     steps {
          sh 'sudo docker ps | grep sonar || sudo docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest'
          sh 'sudo docker run \
                                      --rm \
                                      -e SONAR_HOST_URL="http://172.17.0.2:9000" \
                                      -e SONAR_SCANNER_OPTS="-Dsonar.projectKey=temp2" \
                                      -e SONAR_LOGIN="sqp_ab37c8f4db8b28537e92889a9e409d9fac7af86d" \
                                      -e SONAR_JAVA_BINARIES="/home/kali/1-0-3.1.44.3" \         # don't work 
                                      -v "/home/kali/1-0-3.1.44.3:/usr/src" \
                                      sonarsource/sonar-scanner-cli'
        }
      }
    
      stage ('SAST_sh') { # -C -C++
         steps {
              sh '~/sonar-scanner-4.7.0.2747-linux/bin/sonar-scanner \
                                      -Dsonar.projectKey=TW_ACS_CORE \
                                      -Dsonar.sources=/home/kali/TW_ACS_GIT/iad-ecacs2 \
                                      -Dsonar.host.url=http://localhost:9000 \
                                      -Dsonar.login=sqp_905ffe1d5b3316e808b8a19ae6591c5d69210476 \
                                      -Dsonar.java.binaries=/home/kali/TW_ACS_GIT/iad-ecacs2'
              sh '~/sonar-scanner-4.7.0.2747-linux/bin/sonar-scanner \
                                      -Dsonar.projectKey=TW_ACS_WEB \
                                      -Dsonar.sources=/home/kali/TW_ACS_GIT/iad-web-app-core \
                                      -Dsonar.host.url=http://localhost:9000 \
                                      -Dsonar.login=sqp_b6aaa0ae304206af981a8b13d6f5c9e1f3191a07 \
                                      -Dsonar.java.binaries=/home/kali/TW_ACS_GIT/iad-web-app-core'
              sh '~/sonar-scanner-4.7.0.2747-linux/bin/sonar-scanner \
                                      -Dsonar.projectKey=TW_ACS_JS \
                                      -Dsonar.sources=/home/kali/TW_ACS_GIT/iad-js-utils \
                                      -Dsonar.host.url=http://localhost:9000 \
                                      -Dsonar.login=sqp_b6e6792b5f85b4f8b1b14f507bf60488a2f852f4 \
                                      -Dsonar.java.binaries=/home/kali/TW_ACS_GIT/iad-js-utils'
         }
      }
       stage ('SCA_sh') {
      steps {
         sh 'cd ~/TW_ACS_GIT/iad-ecacs2 && bash ~/dependency-check/bin/dependency-check.sh --project "TW_ACS_CORE" --scan "/home/kali/TW_ACS_GIT/iad-ecacs2" --proxyserver proxy.compassplus.ru --proxyport 3128 '
         sh 'cd ~/TW_ACS_GIT/iad-web-app-core && bash ~/dependency-check/bin/dependency-check.sh --project "TW_ACS_WEB" --scan "/home/kali/TW_ACS_GIT/iad-web-app-core" --proxyserver proxy.compassplus.ru --proxyport 3128 '
         sh 'cd ~/TW_ACS_GIT/iad-js-utils && bash ~/dependency-check/bin/dependency-check.sh --project "TW_ACS_JS" --scan "/home/kali/TW_ACS_GIT/iad-js-utils" --proxyserver proxy.compassplus.ru --proxyport 3128 '
      }
    }
    
     stage ('SCA_docker') {
      steps {
         sh 'cp ~/webapp_pub/owasp-dependency-check.sh ~/TW_ACS_GIT/iad-ecacs2 && sudo bash ~/TW_ACS_GIT/iad-ecacs2/owasp-dependency-check.sh'
         sh 'sudo zip ~/TW_ACS_GIT/iad-ecacs2/odc-reports/TW_ACS_DCH.zip ~/TW_ACS_GIT/iad-ecacs2/odc-reports/dependency-check-report.html'
      }
    }
  
    stage ('Build_TW_ACS') {
      steps {
      sh 'cd ~/webapp_pub && npm install && npm run build && gdradle build'
       }
    }
    
    
    stage ('DAST') {
      steps {
        sshagent(['zap']) {
         sh '"sudo docker run -t owasp/zap2docker-stable zap-full-scan.py -t http://10.77.207.92:2028/" || true'
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
