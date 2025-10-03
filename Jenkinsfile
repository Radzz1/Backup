pipeline {
    agent any

    environment {
        JENKINS_HOME = "/var/lib/jenkins"
        BACKUP_DIR   = "/tmp/jenkins_backup"
        DATE         = sh(returnStdout: true, script: 'date +%F_%H-%M-%S').trim()
        BACKUP_NAME  = "jenkins_backup_${DATE}.tar.gz"
        GIT_REPO     = "https://${GITHUB_TOKEN}@github.com/Radzz1/Backup.git"
        BRANCH       = "main"
    }

    triggers {
        cron('H 2 * * *')  
        // Runs daily at 2 AM (H spreads load across nodes)
    }

    stages {
        stage('Create Backup') {
            steps {
                sh '''
                    echo "[INFO] Creating Jenkins backup..."
                    mkdir -p $BACKUP_DIR
                    tar --exclude='workspace/*' --exclude='logs/*' --exclude='caches/*' \
                        -czf $BACKUP_DIR/$BACKUP_NAME -C $JENKINS_HOME .
                '''
            }
        }

        stage('Push to GitHub') {
            steps {
                sh '''
                    cd $BACKUP_DIR

                    if [ ! -d ".git" ]; then
                        echo "[INFO] Cloning backup repo..."
                        rm -rf $BACKUP_DIR/*
                        git clone -b $BRANCH $GIT_REPO $BACKUP_DIR
                        cd $BACKUP_DIR
                    fi

                    git pull origin $BRANCH || true
                    git add $BACKUP_NAME
                    git commit -m "Daily Jenkins Backup: $DATE" || echo "[INFO] Nothing to commit"
                    git push origin $BRANCH
                '''
            }
        }
    }
}
