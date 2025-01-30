pipeline {
    agent any

    environment {
        GITHUB_CREDENTIALS = credentials('GITHUB_CREDENTIALS')
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Docker Compose Plugin') {
            steps {
                sh '''
                # Install Docker Compose as a CLI plugin
                mkdir -p ~/.docker/cli-plugins/
                curl -fsSL "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o ~/.docker/cli-plugins/docker-compose
                chmod +x ~/.docker/cli-plugins/docker-compose
                docker compose version
                '''
            }
        }

        stage('Install JDK 21') {
            steps {
                sh '''
                # Check if JDK 21 is already installed
                if ! java -version 2>&1 | grep "21" > /dev/null; then
                    echo "Installing JDK 21..."
                    apt-get install -y wget tar
                    wget -q https://github.com/adoptium/temurin21-binaries/releases/download/jdk-21%2B35/OpenJDK21U-jdk_x64_linux_hotspot_21_35.tar.gz -O /tmp/jdk21.tar.gz
                    mkdir -p /usr/lib/jvm
                    tar -xzf /tmp/jdk21.tar.gz -C /usr/lib/jvm --strip-components=1
                    update-alternatives --install /usr/bin/java java /usr/lib/jvm/bin/java 1
                    update-alternatives --set java /usr/lib/jvm/bin/java
                else
                    echo "JDK 21 is already installed."
                fi

                # Verify JDK installation
                java -version
                '''
            }
        }

        stage('Start Test Docker Containers') {
            steps {
                sh '''
                docker compose -f $WORKSPACE/src/test/resources/docker-compose.yml up -d
                '''
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                mvn clean package -DskipTests -s ~/.m2/settings.xml \
                    -Dgithub.username=$(echo $GITHUB_CREDENTIALS | cut -d: -f1) \
                    -Dgithub.token=$(echo $GITHUB_CREDENTIALS | cut -d: -f2)
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                mvn test -s ~/.m2/settings.xml \
                    -Dgithub.username=$(echo $GITHUB_CREDENTIALS | cut -d: -f1) \
                    -Dgithub.token=$(echo $GITHUB_CREDENTIALS | cut -d: -f2)
                '''
            }
        }
    }

    post {
        always {
            sh '''
            docker compose -f $WORKSPACE/src/test/resources/docker-compose.yml down
            '''
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
