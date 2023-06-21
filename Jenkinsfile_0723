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
      }
    }
    
    stage ('preparing_SCA_sh&SAST_Secret') {
      steps {
         sh 'VERSION=$(curl -s https://jeremylong.github.io/DependencyCheck/current.txt)'
         sh 'wget https://github.com/jeremylong/DependencyCheck/releases/download/v$VERSION/dependency-check-$VERSION-release.zip -O dependency-check.zip'
         sh 'unzip -u -o dependency-check.zip'
         sh 'wget https://github.com/adedayo/checkmate/releases/download/v0.8.3/checkmate_0.8.3_linux_amd64.tar.gz && tar -xvf checkmate_0.8.3_linux_amd64.tar.gz -C checkmate_0.8.3_linux_amd64'
         sh 'sudo apt install ruby && gem install asciidoctor-pdf'
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
                                                                -Dsonar.login=sqp_6d8cc7da466dbdf79353ec7b4a2c575a10bc51fa \
                                                                -Dsonar.java.binaries=/home/kali/TW_ACS_GIT/iad-ecacs2'
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
         stage ('SAST_SQL_cli') {  # https://github.com/gretard/sonar-sql-plugin
     steps {
          sh 'cd ~/sonar_sql && wget https://github.com/gretard/sonar-sql-plugin/releases/download/1.3.0/sql-sca-cli.jar'
          sh 'cp ~/sonar_sql/sql-sca-cli.jar ~/TX && cd ~/TX && java -jar sql-sca-cli.jar --warnOnly > TX_sql_sca.txt'

        }
      }  
    
         stage ('SAST_Secret') {  # https://github.com/gretard/sonar-sql-plugin
     steps {
          sh 'cp ./checkmate_0.8.3_linux_amd64/checkmate secretSearch ~/Source\ Code/TX/3.2.30.10.9/com.tranzaxis.millikartaz/'
          sh 'cp $(find /tmp/ | grep .pdf) .'
     }
      } 
    
    
       stage ('SCA_sh') {  # don't updaiting
      steps {
         sh 'cd ~/TW_ACS_GIT/iad-ecacs2 && bash ~/dependency-check/bin/dependency-check.sh --project "TW_ACS_CORE" --scan "/home/kali/TW_ACS_GIT/iad-ecacs2" --proxyserver proxy.compassplus.ru --proxyport 3128 '
         sh 'cd ~/TW_ACS_GIT/iad-web-app-core && bash ~/dependency-check/bin/dependency-check.sh --project "TW_ACS_WEB" --scan "/home/kali/TW_ACS_GIT/iad-web-app-core" --proxyserver proxy.compassplus.ru --proxyport 3128 '
         sh 'cd ~/TW_ACS_GIT/iad-js-utils && bash ~/dependency-check/bin/dependency-check.sh --project "TW_ACS_JS" --scan "/home/kali/TW_ACS_GIT/iad-js-utils" --proxyserver proxy.compassplus.ru --proxyport 3128 '
      }
    }
    
       stage ('SCA_C&C++') {  
      steps {
         sh 'cd ~/Flora/FloraWare && cppcheck --report-progress -v . --output-file=rep_scan_cpp --xml --force'
         sh 'cd ~/Flora/FloraWare && cppcheck-htmlreport --report-dir=./HTML --source-dir=. --file=rep_scan_cpp'
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

