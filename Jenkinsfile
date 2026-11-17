pipeline {
    agent any

    tools {
        maven 'Maven310'
        jdk 'Java17'
    }

    environment {
        // >>> A MODIFIER PAR CHAQUE UTILISATEUR
        USER_ID   = 'user5'                            // votre login (user1 ... user5)
        MAIL_TO   = 'dhiaeddine.saria@sesame.com.tn'   // votre adresse mail

        // URLs publiques des serveurs
        NEXUS_URL = 'http://57.131.31.88:8097'
        SONAR_URL = 'http://57.131.31.88:9000'

        // Calculé automatiquement à partir de USER_ID
        NEXUS_REPO  = "${NEXUS_URL}/repository/maven-snapshots/"
        PROJECT_KEY = "${USER_ID}-jenkins-webapp"
    }

    stages {

        stage('1. Checkout') {
            steps {
                git branch: 'master',
                    credentialsId: 'Git_Secret_Mezghich',
                    url: 'https://github.com/MezghichGit/jenkins_webapp.git'
            }
        }

        stage('2. Build JAR') {
            steps {
                sh 'mvn -B clean package -DskipTests'
            }
        }

        stage('3. Push vers Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: "nexus-${USER_ID}",
                                                  usernameVariable: 'NEXUS_USER',
                                                  passwordVariable: 'NEXUS_PASS')]) {
                    // settings.xml minimal : Maven lit le login/mot de passe dans les variables d'environnement
                    writeFile file: 'nexus-settings.xml', text: '''<settings>
  <servers>
    <server>
      <id>nexus</id>
      <username>${env.NEXUS_USER}</username>
      <password>${env.NEXUS_PASS}</password>
    </server>
  </servers>
</settings>'''
                    sh 'mvn -B deploy -DskipTests -s nexus-settings.xml -DaltDeploymentRepository=nexus::$NEXUS_REPO'
                }
            }
        }

        stage('4. Analyse SonarQube') {
            steps {
                withCredentials([string(credentialsId: "sonar-token-${USER_ID}", variable: 'SONAR_TOKEN')]) {
                    sh '''mvn -B org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                          -Dsonar.host.url=$SONAR_URL \
                          -Dsonar.token=$SONAR_TOKEN \
                          -Dsonar.projectKey=$PROJECT_KEY \
                          -Dsonar.projectName=$PROJECT_KEY'''
                }
            }
        }
    }

    post {
        always {
            script {
                def statut  = currentBuild.currentResult
                def couleur = (statut == 'SUCCESS') ? '#2e7d32' : '#c62828'
                def duree   = currentBuild.durationString.replace(' and counting', '')
                def lienNexus = "${NEXUS_URL}/#browse/browse:maven-snapshots"
                def lienSonar = "${SONAR_URL}/dashboard?id=${PROJECT_KEY}"
                def bouton  = 'background:#1f4e79;color:#fff;padding:8px 14px;border-radius:4px;text-decoration:none'

                emailext(
                    to: "${MAIL_TO}",
                    subject: "[Jenkins] ${env.JOB_NAME} #${env.BUILD_NUMBER} : ${statut}",
                    mimeType: 'text/html',
                    attachLog: true,
                    body: """
<div style="font-family:Arial,sans-serif;max-width:600px;border:1px solid #ddd;border-radius:6px;overflow:hidden">
  <div style="background:${couleur};color:#fff;padding:16px 20px;font-size:20px;font-weight:bold">
    Pipeline ${statut}
  </div>
  <div style="padding:16px 20px;font-size:14px;color:#333">
    <p>Bonjour <b>${USER_ID}</b>,</p>
    <p>Voici le r&eacute;sultat de votre pipeline Jenkins :</p>
    <table style="border-collapse:collapse;width:100%">
      <tr><td style="padding:6px;color:#777;width:140px">Job</td><td style="padding:6px"><b>${env.JOB_NAME}</b></td></tr>
      <tr><td style="padding:6px;color:#777">Build</td><td style="padding:6px">#${env.BUILD_NUMBER}</td></tr>
      <tr><td style="padding:6px;color:#777">R&eacute;sultat</td><td style="padding:6px;color:${couleur}"><b>${statut}</b></td></tr>
      <tr><td style="padding:6px;color:#777">Dur&eacute;e</td><td style="padding:6px">${duree}</td></tr>
      <tr><td style="padding:6px;color:#777">Projet SonarQube</td><td style="padding:6px">${PROJECT_KEY}</td></tr>
    </table>
    <p style="margin-top:18px">
      <a href="${env.BUILD_URL}" style="${bouton}">Voir le build</a>&nbsp;
      <a href="${lienNexus}" style="${bouton}">Mon artefact Nexus</a>&nbsp;
      <a href="${lienSonar}" style="${bouton}">Mon rapport SonarQube</a>
    </p>
    <p style="margin-top:18px;color:#777">Connexion Nexus / SonarQube avec votre compte ${USER_ID}.<br>
    Le log complet du build est en pi&egrave;ce jointe (build.log).</p>
  </div>
</div>
"""
                )
            }
        }
    }
}
