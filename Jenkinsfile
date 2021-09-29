pipeline {

    agent {
        docker {
            image 'androidsdk/android-30'
        }
    }
    /* agent { label 'mac' } */

    environment {
        branch = 'main'
        url = 'https://github.com/wesleymessiassilva91/pipeline-mobile-android.git'
    }

    stages {

        stage('Checkout git') {
            steps {
                git branch: branch, credentialsId: 'Login-github', url: url
            }
        }

        stage('Lint') {
            steps {
                sh "./gradlew lint"
            }
        }

        stage('Test') {
            steps {
                sh "./gradlew test --stacktrace"
            }
        }

        // Manage Jenkins > Credentials > Add > Secret file or Secret Text
        stage('Credentials') {
            steps {
                withCredentials([file(credentialsId: 'ANDROID_KEYSTORE_FILE', variable: 'ANDROID_KEYSTORE_FILE')]) {
                    sh "cp '${ANDROID_KEYSTORE_FILE}' app/hellokey.jks"
                    sh "cat app/hellokey.jks"
                }
                withCredentials([file(credentialsId: 'FIREBASE_SERVICE_ACCOUNT_FILE', variable: 'FIREBASE_SERVICE_ACCOUNT_FILE')]) {
                    sh "cp '${FIREBASE_SERVICE_ACCOUNT_FILE}' app/helloandroid-key-admin.json"
                    sh "cat app/helloandroid-key-admin.json"
                }
            }
        }

        stage('Build') {
            steps {
                sh "./gradlew clean assembleRelease"
                sh "./gradlew bundleRelease"
            }
        }

        stage('Publish') {
            parallel {
                stage('Firebase Distribution') {
                    steps {
                        sh "./gradlew appDistributionUploadRelease"
                    }
                }

                stage('Google Play...') {
                    steps {
                        sh "./gradlew publishBundle"
                        sh "echo 'Test...'"
                    }
                }
            }
        }
    }
}

