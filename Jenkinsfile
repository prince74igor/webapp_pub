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
    
       stage ('SAST_SQL_docker') {  # https://github.com/gretard/sonar-sql-plugin
     steps {
          sh 'mkdir ~/sonar_sql && cd ~/sonar_sql && wget https://github.com/gretard/sonar-sql-plugin/releases/download/1.3.0/sonar-sql-plugin-1.3.0.jar'
          sh 'sudo docker cp ~/sonar_sql/sonar-sql-plugin-1.3.0.jar 34377fd7cacc:/opt/sonarqube/extensions/plugins/ && sudo docker restart 34377fd7cacc '
          sh '~/sonar-scanner-4.7.0.2747-linux/bin/sonar-scanner \
                                                           -Dsonar.projectKey=TWI \
                                                           -Dsonar.sources=/home/kali/TWI \
                                                           -Dsonar.host.url=http://localhost:9000 \
                                                           -Dsonar.login=sqp_31683363655c54f72a7b8d5c4a5c8bbc07ed6651 \
                                                           -Dsonar.java.binaries=/home/kali/TWI -Dsonar.sourceEncoding=UTF-8'

        }
      }
    
      stage ('SAST_sh') { # -C -C++
         steps {
              sh 'find . -name '*[! -~]*' -exec rm {} \;' # delete file with ACSI code name
              sh 'echo "sonar.sourceEncoding=UTF-8" >> ~/sonar-scanner-4.7.0.2747-linux/conf/sonar-scanner.properties'
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
                                      -Dsonar.projectKey=flora_lib \
                                      -Dsonar.sources=/home/kali/FloraWare/FloraWare_lib \
                                      -Dsonar.host.url=http://localhost:9000 \
                                      -Dsonar.login=sqp_7f74cf78c819d56c76da66848133a5f2ae6fb73a'
              sh '~/sonar-scanner-4.7.0.2747-linux/bin/sonar-scanner \
                                                                -Dsonar.projectKey=RadixWare \
                                                                -Dsonar.sources=/home/kali/RadixWare/2.1.43.10.9/org.radixware/kernel/ \
                                                                -Dsonar.host.url=http://localhost:9000 \
                                                                -Dsonar.login=sqp_6d8cc7da466dbdf79353ec7b4a2c575a10bc51fa \
                                                                -Dsonar.java.binaries=/home/kali/RadixWare/2.1.43.10.9/org.radixware/kernel/'
         }
      }
       stage ('SCA_sh') {  # don't updaiting
      steps {
         sh 'cd ~/TW_ACS_GIT/iad-ecacs2 && bash ~/dependency-check/bin/dependency-check.sh --project "TW_ACS_CORE" --scan "/home/kali/TW_ACS_GIT/iad-ecacs2" --proxyserver proxy.compassplus.ru --proxyport 3128 '
         sh 'cd ~/TW_ACS_GIT/iad-web-app-core && bash ~/dependency-check/bin/dependency-check.sh --project "TW_ACS_WEB" --scan "/home/kali/TW_ACS_GIT/iad-web-app-core" --proxyserver proxy.compassplus.ru --proxyport 3128 '
         sh 'cd ~/TW_ACS_GIT/iad-js-utils && bash ~/dependency-check/bin/dependency-check.sh --project "TW_ACS_JS" --scan "/home/kali/TW_ACS_GIT/iad-js-utils" --proxyserver proxy.compassplus.ru --proxyport 3128 '
      }
    }
    
     stage ('SCA_docker') {
      steps {
         sh 'sudo rm -r -f ~/odc-reports/'
         sh 'cp ~/webapp_pub/owasp-dependency-check.sh ~/TW_ACS_GIT/iad-ecacs2 && sudo bash ~/TW_ACS_GIT/iad-ecacs2/owasp-dependency-check.sh'
         sh 'sudo zip ~/odc-reports/FloraWare_DCH_lib.zip ~/odc-reports/dependency-check-report.html'
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
          sh 'cd ~/TW_ACS && sudo docker run --rm -d -v $(pwd):/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://10.77.207.92:2028/ -g gen.conf -r ZAP_FULL_REP.html'
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
