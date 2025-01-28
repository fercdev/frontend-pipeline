pipeline {
    agent any

    stages {
        stage ('Instalar dependencias...') {
            agent {
                docker { image 'node:16-alpine'}
            }
            steps {
                sh 'npm install'
            }
        }

        stage ('Construir proyecto con archivos estaticos...') {
            agent {
                docker { image 'node:16-alpine'}
            }
            steps {
                sh 'npm run build'
            }
        }

        stage('Validar conexion AWS ...') {
            agent {
                docker { image 'amazon/aws-cli:2.23.7'}
            }
            steps {
                withAWS(credentials: 'aws-credentials-s3', region: 'us-east-1') {
                    script {
                        def buckets  = sh(returnStdout: true, script: 'aws s3 ls').trim()
                        echo "Buckets disponibles en aws: \n${buckets}"
                    }
                }
            }

        }

        stage('Subir proyecto al bucket s3 AWS ...') {
            agent {
                docker { image 'amazon/aws-cli:2.23.7'}
            }
            steps {
                withAWS(credentials: 'aws-credentials-s3', region: 'us-east-1') {
                    s3Upload(bucket: 'bucket-codigo-front', includePathPattern: '**/*', path: 'build/')
                                        
                }
            }
        }
    }
}