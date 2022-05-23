pipeline {
    agent { label "Windows" }
    environment {
        // General
        COMPILER = 'C:\\dev\\DataFlex 20.1\\Bin64\\DfCompConsole.exe'
    }
    stages {
        stage("Pipeline") {
            parallel {
                stage("Unit Tests") {
                    environment {
                        WORKSPACE_NAME = "DFUnit-Sample"
                        UNITTEST_NAME = "RunAllTests"

                        WORKSPACE_DIR = "DFUnit Example Workspace"
                        WORKSPACE_OUTPUT = "${WORKSPACE_DIR}\\Programs"
                        UNITTEST_SWS = "${WORKSPACE_DIR}\\${WORKSPACE_NAME}.sws"
                        UNITTEST_SRC = "AppSrc\\${UNITTEST_NAME}.src"
                        UNITTEST_JUNIT_OUTPUT = "test_results.xml"
                    }
                    stages {
                        stage("Build") {
                            steps {
                                script {
                                    bat "\"${COMPILER}\" -x\"${UNITTEST_SWS}\" -c \"${UNITTEST_SRC}\""
                                    stash name: 'build', includes: "**"
                                }
                            }
                        }
                        stage('Test') {
                            steps {
                                unstash 'build'
                                powershell(script: """
                                    \$executable = "${WORKSPACE_OUTPUT}\\${UNITTEST_NAME}.exe"
                                    \$arguments = "--console -o ${UNITTEST_JUNIT_OUTPUT}"
                                    \$process = Start-Process -Wait -PassThru \$executable \$arguments
                                    if (\$process.ExitCode -ne 0) {
                                        echo "Unit Tests failed with exit code: \$(\$process.ExitCode)."
                                        exit -1
                                    }
                                """)
                            }
                            post {
                                always {
                                    junit "${UNITTEST_JUNIT_OUTPUT}"
                                    archiveArtifacts artifacts: "${UNITTEST_JUNIT_OUTPUT}", fingerprint: true
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}