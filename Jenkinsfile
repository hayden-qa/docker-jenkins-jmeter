pipeline {
    agent any

    parameters {
        choice(name: 'TEST_TYPE', choices: ['smoke', 'full'])
    }

    environment {
        JMETER_HOME = '/opt/jmeter'
    }

    stages {
        stage('Run JMeter') {
            steps {
                script {
                    def threads = params.TEST_TYPE == 'smoke' ? 5 : 50
                    def loops   = params.TEST_TYPE == 'smoke' ? 10 : 100
                    sh "${env.JMETER_HOME}/bin/jmeter -n -t smoke_test.jmx -l results.jtl -e -o reports -Jthreads=${threads} -Jloops=${loops} -Jrampup=5 -JtargetHost=httpbin.org -JtargetPort=443 -JtargetProtocol=https"
                }
            }
        }

        stage('Publish Report') {
            steps {
                perfReport(
                    sourceDataFiles: 'results.jtl',
                    errorFailedThreshold: 5,
                    errorUnstableThreshold: 1,
                    compareBuildPrevious: true,
                    showTrendGraphs: true
                )
            }
        }

        stage('Check Threshold') {
            steps {
                script {
                    def avg = sh(
                        returnStdout: true,
                        script: "awk -F',' 'NR>1 {sum+=\$2; count++} END {printf \"%d\", sum/count}' results.jtl"
                    ).trim().toInteger()
                    def limit = params.TEST_TYPE == 'smoke' ? 2000 : 1000
                    if (avg > limit) {
                        error("Average ${avg}ms exceeded ${limit}ms")
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'results.jtl, reports/**', allowEmptyArchive: true
        }
        failure {
            echo "Notify: ${params.TEST_TYPE} build FAILED - ${env.BUILD_URL}"
        }
    }
}
