// https://confluence.cgm.ag/display/g3his/Jenkinsfile+Content
// https://jenkins.io/doc/book/pipeline/shared-libraries/

@Library('jenkins2-pipeline@6.0-latest') _

def jobConfig = defaultFastlaneJobConfig(CLEAN: true)
addUpstreamVersionParameter(jobConfig, "HELIXCONTRIB")
addUpstreamVersionParameter(jobConfig, "MDM_API")
addUpstreamVersionParameter(jobConfig, "PST_API")
addUpstreamVersionParameter(jobConfig, "POTS_API")
addUpstreamVersionParameter(jobConfig, "G3HIS_CORE_API")
applyFastlaneJobConfig(jobConfig)

def verify() {
    if (params.TESTS) {
        try {
            gradle_mvn "verify -Dbuildnumber.skip -Denforcer.skip"
        }
        finally {
            junit(allowEmptyResults: true, testResults:'**/target/failsafe-reports/*.xml')
        }
    } else {
        echo "skip tests"
    }
}

echo "params: $params"

node_label="NABU"

pipeline {
    agent { node { label 'NABU' } }

    options {
        gitLabConnection('gitlab-test')
    }

    tools {
        maven 'Maven-3.5.4'
    }

    stages {
        stage('Preparation') {
            steps {
                gitlabBuilds(builds: ["0. Preparation", "1. Build", "2. Unit Tests", "3. Integration Tests", "4. Publish"]) {
                    updateGitlabCommitStatus name: '0. Preparation', state: 'running'
                    fastlanePreparation()
                }
            }
            post {
                success {
                    updateGitlabCommitStatus name: '0. Preparation', state: 'success'
                }
                failure {
                    updateGitlabCommitStatus name: '0. Preparation', state: 'failed'
                    updateGitlabCommitStatus name: '1. Build', state: 'failed'
                    updateGitlabCommitStatus name: '2. Unit Tests', state: 'failed'
                    updateGitlabCommitStatus name: '3. Integration Tests', state: 'failed'
                    updateGitlabCommitStatus name: '4. Publish', state: 'failed'
                }
            }
        }

        stage('Build') {
            steps {
                updateGitlabCommitStatus name: '1. Build', state: 'running'
                gradle_mvn "${params.CLEAN ? 'clean' : ''} install -DskipTests"
            }
            post {
                success {
                    updateGitlabCommitStatus name: '1. Build', state: 'success'
                }
                failure {
                    updateGitlabCommitStatus name: '1. Build', state: 'failed'
                    updateGitlabCommitStatus name: '2. Unit Tests', state: 'failed'
                    updateGitlabCommitStatus name: '3. Integration Tests', state: 'failed'
                    updateGitlabCommitStatus name: '4. Publish', state: 'failed'

                }
            }
        }

        stage('Unit Tests') {
            steps {
                updateGitlabCommitStatus name: '2. Unit Tests', state: 'running'
                mvnTestArchiveResults()
            }
            post {
                success {
                    updateGitlabCommitStatus name: '2. Unit Tests', state: 'success'
                }
                failure {
                    updateGitlabCommitStatus name: '2. Unit Tests', state: 'failed'
                    updateGitlabCommitStatus name: '3. Integration Tests', state: 'failed'
                    updateGitlabCommitStatus name: '4. Publish', state: 'failed'
                }
            }
        }

        stage('Integration Tests') {
            steps {
                updateGitlabCommitStatus name: '3. Integration Tests', state: 'running'
                verify()
            }
            post {
                success {
                    updateGitlabCommitStatus name: '3. Integration Tests', state: 'success'
                }
                failure {
                    updateGitlabCommitStatus name: '3. Integration Tests', state: 'failed'
                    updateGitlabCommitStatus name: '4. Publish', state: 'failed'
                }
            }
        }

        stage('Publish Fastlane') {
            steps {
                gradle_mvn "deploy -DskipTests"
                updateGitlabCommitStatus name: '4. Publish', state: 'success'
            }
            post {
                success {
                    updateGitlabCommitStatus name: '4. Publish', state: 'success'
                }
                failure {
                    updateGitlabCommitStatus name: '4. Publish', state: 'failed'
                }
            }
        }
    }
}