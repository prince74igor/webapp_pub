pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                // Your build steps here
                // Generate HTML report
                
                // For example, if the HTML report is generated in 'reports' directory:
                sh 'cd ~/test && python3 test_html.py'
            }
        }
          
stage('Publish HTML Report') {
            steps {
                // Publish HTML report
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: true,
                    reportDir: '~/test',
                    reportFiles: 'example.html',
                    reportName: 'My HTML Report'
                ])
            }
        }
    }
}
