pipeline {
    agent any

    environment {
        FLASK_EC2 = "ubuntu@3.109.207.92"
        APP_DIR = "/home/ubuntu/flask-app"
        SSH_KEY = "/var/lib/jenkins/.ssh/id_ed25519"
    }

    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies Locally') {
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
                python - << 'EOF'
import flask
print("Flask OK:", flask.__version__)
EOF
                '''
            }
        }

        stage('Deploy to Flask EC2') {
            steps {
                sh '''
                # Create app directory on remote
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${FLASK_EC2} \
                    "mkdir -p ${APP_DIR}"

                # Copy the application files
                scp -i ${SSH_KEY} -o StrictHostKeyChecking=no \
                    app.py requirements.txt templates ${FLASK_EC2}:${APP_DIR}/

                # Set up virtual environment and start Flask
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${FLASK_EC2} "
                  cd ${APP_DIR} &&

                  # create venv if missing
                  if [ ! -d venv ]; then
                    python3 -m venv venv
                  fi &&

                  # activate and install
                  . venv/bin/activate &&
                  pip install --upgrade pip &&
                  pip install -r requirements.txt &&

                  # stop old app process
                  pkill -f 'venv/bin/python app.py' || true &&

                  # start app in background with logging
                  nohup venv/bin/python app.py > flask.log 2>&1 &
                "
                '''
            }
        }
    }
}
