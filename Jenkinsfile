pipeline {
    agent any
    tools {
        maven 'Maven 3.3.9'
        jdk 'jdk-8'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('Sonar') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar ' +
                            '-Dsonar.login=$SONAR_UN ' +
                            '-Dsonar.password=$SONAR_PW ' +
                            '-Dsonar.test.inclusions=**/*Test*/** ' +
                            '-Dsonar.exclusions=**/*Test*/**'
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'mvn deploy'
            }
        }
        stage('Build and deploy Docker to nexus') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    dir('web') {
                        docker.withRegistry('http://localhost:8082', 'nexus-credentials') {
                            def webImage = docker.build("web:${pom.version}")
                            webImage.push()
                        }
                    }
                }
            }
        }
    }
}
