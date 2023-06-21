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
