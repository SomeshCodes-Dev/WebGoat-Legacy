pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'jdk8'
    }

    environment {
        ARTEFACT_NAME = "${WORKSPACE}/target/WebGoat-${BUILD_VERSION}.war"
        IQ_SCAN_URL = ""
        // BUILD_TAG = "org.demo"
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -Dproject.version=$BUILD_VERSION -Dmaven.test.failure.ignore clean package'
            }
            post {
                success {
                    echo 'Now archiving...'
                    archiveArtifacts artifacts: "**/target/*.war"
                }
            }
        }

        stage('Nexus IQ Scan') {
            steps {
                script {
                    def policyEvaluation = nexusPolicyEvaluation (
                        advancedProperties: '',
                        enableDebugLogging: false,
                        failBuildOnNetworkError: false,
                        failBuildOnScanningErrors: false,
                        //iqApplication: selectedApplication('webgoat'),
                        iqInstanceId: 'nxiq',
                        iqOrganization: 'e10a8b63f64d40c49c492f5d5ad6eef6',
                        iqScanPatterns: [[scanPattern: '**/*.war']],
                        iqStage: 'build',
                        //jobCredentialsId: 'sonatypeIQ',
                        reachability: [
                            javaAnalysis: [
                                enable: true
                            ]
                        ]
                    )

                    echo "Nexus IQ scan succeeded: ${policyEvaluation.applicationCompositionReportUrl}"
                    IQ_SCAN_URL = "${policyEvaluation.applicationCompositionReportUrl}"
                }
            }
        }

        stage("Publish to Repo") {
            steps {
                script {
                    nexusPublisher nexusInstanceId: 'nxrm3',
                        nexusRepositoryId: "maven-releases",
                        packages: [[$class: 'MavenPackage',
                            mavenAssetList: [[
                                classifier: '',
                                extension: 'war',
                                filePath: "${ARTEFACT_NAME}"
                            ]],
                            mavenCoordinate: [
                                artifactId: 'WebGoat',
                                groupId: 'org.demo',
                                packaging: 'war',
                                version: "${BUILD_VERSION}"
                            ]]],
                        tagName: "${BUILD_TAG}"
                }
            }
        }
    }
}
