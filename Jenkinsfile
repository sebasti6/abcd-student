pipeline {
    agent any
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-pat', url: 'https://github.com/sebasti6/abcd-student', branch: 'main'
                }
            }
        }
        stage('Workspace Jenkins') {
            steps {
                sh 'mkdir -p results'
            }
        }
        stage('ZAP Scan') {
            steps {
                sh '''
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v /home/sebastianc/abcd-student/.zap/:/zap/wrk/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive_scan.yaml" || true
                '''
            }
        }
    }
    post {
        always {
            // ${WORKSPACE} RESOLVES TO /var/jenkins_home/workspace/ABCD
            sh '''
                docker stop juice-shop
                docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                docker rm zap
            '''


        }
        success {
                echo 'Archiving results'
                archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
                echo 'Sending reports to DefectDojo'
                defectDojoPublisher(artifact: 'results/zap_xml_report.xml', 
                    productName: 'Juice Shop', 
                    scanType: 'ZAP Scan', 
                    engagementName: 'sebasti6@o2.pl')
        }
    }
}
