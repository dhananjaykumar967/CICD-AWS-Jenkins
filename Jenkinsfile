node {
    def appDir = '/var/www/nextjs-app'

    stage('Clean Workspace') {
        echo 'Cleaning Jenkins Workspace'
        deleteDir()
    }

    stage('Clone Repo') {
        echo 'Cloning the repo'
        git branch: 'main', url: 'https://github.com/dhananjaykumar967/CICD-AWS-Jenkins'
    }

    stage('Deploy to EC2') {
        echo 'Deploying to EC2'
        // Stops Jenkins from killing the app process when the build ends
        withEnv(['JENKINS_NODE_COOKIE=dontKillMe']) {
            sh """
                sudo mkdir -p ${appDir}
                sudo chown -R jenkins:jenkins ${appDir}
                rsync -av --delete --exclude='.git' --exclude='node_modules' ./ ${appDir}
                cd ${appDir}
                npm install
                npm run build

                # Install pm2 once if it isn't already there
                if ! command -v pm2 >/dev/null 2>&1; then
                    sudo npm install -g pm2
                fi

                # Free the port in case a stale process is holding it
                sudo fuser -k 3000/tcp || true

                # Restart if already running, otherwise start fresh
                pm2 delete nextjs-app || true
                pm2 start npm --name nextjs-app -- run start
                pm2 save
            """
        }
    }
}