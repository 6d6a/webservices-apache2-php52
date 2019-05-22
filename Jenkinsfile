@Library('mj-shared-library') _
pipeline {
        agent { label 'master' }
        environment {
            PROJECT_NAME = gitRemoteOrigin.getProject()
            GROUP_NAME = gitRemoteOrigin.getGroup()
        }
        options {
            gitLabConnection(Constants.gitLabConnection)
            gitlabBuilds(builds: ['Build ftpserver', 'Push Docker image', 'Push latest Docker image'])
            buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
        }
        stages {
            stage('Build php52') {
                steps {
                    gitlabCommitStatus(STAGE_NAME) {
                         sh ' . /var/jenkins_home/.nix-profile/etc/profile.d/nix.sh && docker load --input $(nix-build --cores 8 default.nix --show-trace | grep tar ) '
                    }
                }
            }
            stage('Push Docker image') {
                when { branch 'master' }
                steps {
                    gitlabCommitStatus(STAGE_NAME) {
                          sh 'docker push docker-registry.intr/webservices/php52:master'
                    }
                }
            }
            stage('Push latest Docker image') {
                when { not { branch 'master' }}
                steps {
                    gitlabCommitStatus(STAGE_NAME) {
                          sh 'docker push docker-registry.intr/webservices/php52:latest'
                    }
                }
            }
        }
        post {
            success { cleanWs() }
            failure { notifySlack "Build failled: ${JOB_NAME} [<${RUN_DISPLAY_URL}|${BUILD_NUMBER}>]", "red" }
        }
    }