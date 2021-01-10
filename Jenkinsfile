pipeline {
    agent {
        docker { image 'timbru31/node-alpine-git:14' }
    }
    environment {
        GIT_LATEST_COMMIT_EDITOR= sh(
            returnStdout:true,
            script: 'git show -s --pretty=%cn '
        ).trim()
        GIT_SSH_URL = sh(
            returnStdout: true,
            script: "git config --get remote.origin.url | sed 's/https:\\/\\/github.com\\//git@github.com:/g'"
        )
        GIT_CURRENT_BRANCH = sh(
            returnStdout: true,
            script: "git rev-parse --abbrev-ref HEAD"
        )
	  }

    parameters {
        string(name: 'SLACK_CHANNEL', defaultValue:'#platform', description: 'slack channel to notify')
    }

    stages {
        stage ('Show commit author') {
            steps {
                sh "echo '${env.GIT_LATEST_COMMIT_EDITOR}'"
            }
        }
        stage ('Execute CI pipeline') {
            stages{
                stage ('Copy node modules folder'){
                    steps {
                        sh 'cp -r /app/* .'
                    }
                }
                stage('NPM build'){
                    steps {
                        sh 'npm run-script build --prod'
                    }
                }
                stage ('Unit tests') {
                    steps{
                        sh 'ng test --karma-config karma.conf.ci.js'
                    }
                }
                stage ('Artifacts') {
                    steps {
                        archiveArtifacts artifacts: 'dist/**/*.*'
                    }
                }
                stage ('Trigger CD pipeline'){
                    steps{
                        build job: 'cd-pipeline', parameters: [
                            string(name: 'REPO_URL', value: "${env.GIT_SSH_URL}"),
                            string(name: 'REPO_BRANCH', value: "${env.GIT_CURRENT_BRANCH}")
                        ], wait: false
                    }
                }
            }
        }
    }
}