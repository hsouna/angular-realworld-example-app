pipeline {
    agent any
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
        HOME = '.'
	  }


    stages {
        stage ('Show commit author') {
            steps {
                sh "echo '${env.GIT_LATEST_COMMIT_EDITOR}'"
            }
        }
        stage ('Execute CI pipeline') {
            agent {
                docker { image 'node:12-buster-slim' }
            }
            stages{
                stage ('npm install'){
                    steps {
                        sh "npm install"
                    }
                }
                stage('NPM build'){
                    steps {
                        sh 'npm run-script build --prod'
                    }
                }
                stage ('Artifacts') {
                    steps {
                        sh "tar czvf dist.tar.gz dist"
                        sh 'tar -czvf app.${BUILD_ID}.tar.gz dist'
                        archiveArtifacts "**/*.tar.gz"
                    }
                }
            }
        }
    }
}