pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                // Your build steps here
                // Generate HTML report
                
                // For example, if the HTML report is generated in 'reports' directory:
                sh 'mkdir -p reports'
                sh 'echo "<html><body><h1>Sample HTML Report</h1></body></html>" > reports/sample.html'
            }
        }
          
stage('Publish HTML Report') {
            steps {
                // Publish HTML report
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'sample.html',
                    reportName: 'My HTML Report'
                ])
            }
        }
    }
}
