pipeline {
    agent any

    triggers {
        cron('0 18 * * *')   // Run daily at 6:00 AM
    }

    stages {
        stage('Backup Jenkins') {
            steps {
                script {
                    def JENKINS_HOME = "/var/lib/jenkins"
                    def WORKSPACE_DIR = "${env.WORKSPACE}"
                    def DATE = sh(script: "date +%F_%H-%M-%S", returnStdout: true).trim()
                    def BACKUP_NAME = "jenkins_backup_${DATE}.tar.gz"

                    sh """
                        echo "[INFO] Creating Jenkins backup..."
                        tar -czf $WORKSPACE_DIR/$BACKUP_NAME -C $JENKINS_HOME .
                    """
                }
            }
        }

        stage('Commit & Push to GitHub') {
            steps {
                sh """
                    cd $WORKSPACE
                    git add *.tar.gz
                    git commit -m "Daily Jenkins Backup: ${DATE}" || echo "No changes to commit"
                    git push origin main
                """
            }
        }
    }
}
