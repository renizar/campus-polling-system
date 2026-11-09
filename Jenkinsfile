pipeline {
    agent any

    options {
        timestamps()
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Syntax Check') {
            steps {
                sh '''
                    cd /home/renizar/campus-polling-system
                    python3 -m py_compile backend/app.py
                    docker compose config -q
                '''
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    cd /home/renizar/campus-polling-system
                    docker compose build
                '''
            }
        }

        stage('Automated Deployment') {
            steps {
                sh '''
                    cd /home/renizar/campus-polling-system
                    docker compose up -d --remove-orphans
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    cd /home/renizar/campus-polling-system

                    for i in 1 2 3 4 5 6 7 8 9 10; do
                        if docker compose exec -T frontend wget -qO- http://backend:5000/votes > /dev/null 2>&1; then
                            echo "Health check OK"
                            exit 0
                        fi

                        echo "Menunggu service siap ($i/10)..."
                        sleep 3
                    done

                    echo "Health check GAGAL"
                    docker compose logs --tail=50
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'Deploy sukses: http://localhost:8083'
        }

        failure {
            echo 'Pipeline gagal, cek console output.'
        }
    }
}
