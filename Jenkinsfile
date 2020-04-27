#!groovy

@Library('fedora-pipeline-library') _

def pipelineMetadata = [
    pipelineName: 'rpmdeplint',
    pipelineDescription: 'Finding errors in RPM packages in the context of their dependency graph',
    testCategory: 'integration',
    testType: 'tier1',
    maintainer: 'Fedora CI',
    docs: 'https://github.com/fedora-ci/rpmdeplint-rawhide',
    contact: [
        irc: '#fedora-ci',
        email: 'ci@lists.fedoraproject.org'
    ],
]
def artifactId


pipeline {

    options {
        buildDiscarder(logRotator(numToKeepStr: '200'))
    }

    agent {
        label 'fedora-ci-agent'
    }

    parameters {
        string(name: 'ARTIFACT_ID', defaultValue: null, trim: true, description: '"koji-build:<taskId>" for Koji builds; Example: koji-build:42376994')
    }

    stages {
        stage('Prepare') {
            steps {
                script {
                    artifactId = params.ARTIFACT_ID

                    if (!artifactId) {
                        abort('ARTIFACT_ID is missing')
                    }
                }
                setBuildNameFromArtifactId(artifactId: artifactId)
                sendMessage(type: 'queued', artifactId: artifactId, pipelineMetadata: pipelineMetadata, dryRun: isPullRequest())
            }
        }

        stage('Test') {
            steps {
                sendMessage(type: 'running', artifactId: artifactId, pipelineMetadata: pipelineMetadata, dryRun: isPullRequest())
                script {
                    def releaseId = getReleaseIdFromBranch()
                    echo "Call Testing Farm here and wait for results..."
                    echo "script: /rpmdeplint/run_rpmdeplint.py -r ${releaseId} -t ${artifactId.split(':')[1]}"
                }
            }
        }
    }

    post {
        always {
            echo 'Archive results. This includes fetching artifacts from Testing Farm (if needed).'
        }
        success {
            echo 'Publish results and send email(s).'
            sendMessage(type: 'complete', artifactId: artifactId, pipelineMetadata: pipelineMetadata, dryRun: isPullRequest())
        }
        failure {
            echo 'Publish results and send email(s).'
            sendMessage(type: 'error', artifactId: artifactId, pipelineMetadata: pipelineMetadata, dryRun: isPullRequest())
        }
        unstable {
            echo 'Publish results and send email(s).'
            sendMessage(type: 'complete', artifactId: artifactId, pipelineMetadata: pipelineMetadata, dryRun: isPullRequest())
        }
    }
}
