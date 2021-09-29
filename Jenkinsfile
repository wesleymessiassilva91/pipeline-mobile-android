podTemplate(
    name: 'android',
    label: 'android',
    namespace: 'devops',
    containers: [
        containerTemplate(name: 'android-sdk', image: 'androidsdk/android-30', command: 'sleep', args: '99d')
    ],
    volumes: [hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')]
) 
{
    node('android') {
        def GIT_REPO_URL = "https://github.com/wesleymessiassilva91/pipeline-mobile-android.git"
        stage('Checkout git') {
            checkout([
              $class: 'GitSCM', 
              branches: [[name: '*/main']], 
              extensions: [], 
              userRemoteConfigs: [[credentialsId: '	Login-github', url: GIT_REPO_URL]]])
        }
        stage('Lint') {
            container('android-sdk'){
                sh "./gradlew lint"
            }
        }
        stage('Test') {
            container('android-sdk') {
                sh "./gradlew test --stacktrace"
            }
        }
        stage('Credentials') {
            container('android-sdk') {
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
            container('android-sdk') {
                sh "./gradlew clean assembleRelease"
                sh "./gradlew bundleRelease"
            }
        }
        stage('Firebase Distribution') {
            container('android-sdk') {
                sh "./gradlew appDistributionUploadRelease --track $Track"
            }
        }
        stage('Google Play') {
            container('android-sdk') {
                sh "./gradlew publishBundle --track $Track"
            }
        }
    }
}
