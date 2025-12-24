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
                echo "=== Checking out source code ==="
                checkout scm
            }
        }

        stage('Install Dependencies Locally') {
            steps {
                echo "=== Installing dependencies locally for test ==="
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
                echo "=== Running basic Flask import test ==="
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
                echo "=== Deploying to Flask EC2 ==="
                sh '''
                # Create the target directory on the EC2 machine
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${FLASK_EC2} "mkdir -p ${APP_DIR}"

                # Copy application files and templates directory recursively
                scp -r -i ${SSH_KEY} -o StrictHostKeyChecking=no \
                    app.py requirements.txt templates \
                    ${FLASK_EC2}:${APP_DIR}/

                # SSH and start the app on EC2
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${FLASK_EC2} "
                  cd ${APP_DIR} &&

                  # Create python venv if it is missing
                  if [ ! -d venv ]; then
                    python3 -m venv venv
                  fi &&

                  # Activate and install requirements
                  . venv/bin/activate &&
                  pip install --upgrade pip &&
                  pip install -r requirements.txt &&

                  # Stop any existing Flask app
                  pkill -f 'venv/bin/python app.py' || true &&

                  # Start Flask in the background and log output
                  nohup venv/bin/python app.py > flask.log 2>&1 &
                "
                '''
            }
        }
    }

    post {
        always {
            echo "=== Pipeline finished ==="
        }
        success {
            echo "=== Deployment succeeded! Visit http://${FLASK_EC2.split('@')[1]}:5000 ==="
        }
        failure {
            echo "=== Deployment failed — check logs! ==="
        }
    }
}
