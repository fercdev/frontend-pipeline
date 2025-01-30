pipeline {
    agent any

    environment {
        VERCEL_TOKEN = credentials('VERCEL_TOKEN')
    }

    parameters {
        string(name: 'BUCKET_FUENTE', defaultValue: 'bucket-codigo-backup', description: 'Nombre del bucket de origen..')
        string(name: 'BUCKET_TARGET', defaultValue: 'bucket-codigo-front', description: 'Nombre del bucket objetivo..')
        string(name: 'CARPETA_USUARIO', defaultValue: 'fernando', description: 'Nombre de la carpeta del usuario..')
        string(name: 'CARPETA_FUENTE', defaultValue: 'VERSION_1.0', description: 'Nombre de la carpeta del bucket origen..')
        string(name: 'CARPETA_RAMA', defaultValue: 'master', description: 'Nombre de la carpeta de la rama del proyecto')
        booleanParam(name: 'DESPLEGAR_A_VERCEL', defaultValue: false, description: 'Deseas desplegar en Vercel?')
    }


    stages {
        stage('Validar parametros...') {
            agent any
            steps {
                script {
                    echo "BUCKET DE ORIGEN: ${params.BUCKET_FUENTE}"
                    echo "BUCKET DE DESTINO: ${params.BUCKET_TARGET}"
                    echo "CARPETA DE USUARIO: ${params.CARPETA_USUARIO}"
                    echo "CARPETA DE ORIGEN: ${params.CARPETA_FUENTE}"
                    echo "CARPETA RAMA: ${params.CARPETA_RAMA}"
                    echo "FLAG PARA DESPLEGAR EN VERCEL: ${params.DESPLEGAR_A_VERCEL}"
                }
            }
        }

        stage('Descargar el S3 hacia carpeta build') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.23.7'
                    args '--entrypoint ""'
                }
            }

            when {
                expression {
                    return params.DESPLEGAR_A_VERCEL
                }
            }

            steps {
                withAWS(credentials: 'aws-credentials-s3', region: 'us-east-1') {
                    script {
                        echo "Crear una carpeta para desplegar a vercel..."
                        sh "mkdir -p build"

                        sh """
                            aws s3 sync s3://${params.BUCKET_FUENTE}/${params.CARPETA_USUARIO}/${params.CARPETA_RAMA}/${params.CARPETA_FUENTE}/ build/
                        """
                        echo "Listando carpetas ..."
                        sh "ls -la"
                    }                   
                }
            }
        }

        stage('Desplegar hacia vercel.....') {
             agent {
                docker { image 'node:18-alpine'}
            }

            when {
                expression {
                    return params.DESPLEGAR_A_VERCEL
                }
            }

            steps {
                echo "Listar carpetas antes del deploy..."
                sh "ls -la"

                echo "desplegando con vercel...."
                sh """
                    npm install -g vercel
                    vercel deploy --prod --name front-vercel --token $VERCEL_TOKEN --yes
                """
            }
        }


        stage('Mover archivos entre buckets s3 AWS ...') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.23.7'
                    args '--entrypoint ""'
                }
            }

            when {
                expression {
                    return !params.DESPLEGAR_A_VERCEL
                }
            }
            
            steps {
                withAWS(credentials: 'aws-credentials-s3', region: 'us-east-1') {
                    script {
                        echo "Moviendo archivos entre buckets s3..."
                        sh """
                            aws s3 mv s3://${params.BUCKET_FUENTE}/${params.CARPETA_USUARIO}/${params.CARPETA_RAMA}/${params.CARPETA_FUENTE}/ s3://${params.BUCKET_TARGET}/ --recursive
                        """
                    }                   
                }
            }
        }
    }
}