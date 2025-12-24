pipeline {
    agent any

    environment {
        APP_DIR = "/home/ubuntu/flask-app"
        VENV_DIR = "venv"
        FLASK_PORT = "5000"
        FLASK_EC2 = "ubuntu@3.109.207.92"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Thilakeshaws27/flaskdemoapp.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run Basic Test') {
            steps {
                sh '''
                . venv/bin/activate
                python -c "import flask; print('Flask OK')"
                '''
            }
        }

        stage('Deploy to Flask EC2') {
            steps {
                sh '''
                ssh ${FLASK_EC2} "mkdir -p ${APP_DIR}"
                scp -r * ${FLASK_EC2}:${APP_DIR}

                ssh ${FLASK_EC2} "
                  cd ${APP_DIR} &&
                  python3 -m venv venv &&
                  . venv/bin/activate &&
                  pip install -r requirements.txt &&
                  pkill -f app.py || true &&
                  nohup venv/bin/python app.py &
                "
                '''
            }
        }
    }
}