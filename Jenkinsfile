pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('1ede0dfb-20f1-46cb-9599-1dd484d9b50e')
        IMAGE_TAG = "v.0.0.${env.BUILD_NUMBER}-stable"
        IMAGE_NAME = "geraldakenji/s3-browser:${IMAGE_TAG}"
        
    }
    stages {
        // stage('Git Checkout') {
        //     steps {
        //         checkout([$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[url: 'https://github.com/GeraldAkenji/s3-browser.git']]])
        //     }
        // }
        stage("Trivy File Scan") {
            steps {
                script {
                    def trivyScan = sh(script: "trivy fs . --exit-code 0 --severity HIGH --no-progress", returnStatus: true)
                    if (trivyScan != 0) {
                        error("Trivy scan failed")
                    }
                    else {
                        echo "Trivy scan passed"
                    }
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    sh "docker build -t $IMAGE_NAME ."
                }
            }
        }
        stage("Trivy Image Scan") {
            steps {
                script {
                    sh "trivy image --severity CRITICAL,HIGH --ignore-unfixed --format json -o trivy_report.json $IMAGE_NAME"
                    def scanResults = readJSON file: 'trivy_report.json'
                    echo "Trivy scan results: ${scanResults}"
                    int highVulns = 0
                    int criticalVulns = 0

                    scanResults.Results.each { result ->
                        result.Vulnerabilities?.each { vuln ->
                            if (vuln.Severity == "HIGH") {
                                highVulCount++
                            } else if (vuln.Severity == "CRITICAL") {
                                criticalVulCount++
                            }
                        }
                    }

                    if (highVulns > 0 || criticalVulns > 0) {
                        error("Pipeline failed due to high or critical vulnerabilities")
                    }
                    else {
                        echo "No high or critical vulnerabilities found"
                    }
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                    sh "docker push $IMAGE_NAME"
                }
            }
        }
    }
}