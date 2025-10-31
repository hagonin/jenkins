pipeline {
  agent {
        label "reviewer-code"
    }

stages {
    stage('Dependancies') {
      steps {
        sh '''
          set -eux
          sudo apt-get update -y
          sudo apt-get install -y apache2 curl
          sudo systemctl start apache2
'''
      }
    }

    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Backup current site') {
      steps {
        sh '''
          set -eux
          sudo rm -rf /var/www/html/*
          sudo cp -r . /var/www/html/
          '''
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          set -eux
          curl -f http://localhost/ || curl -f http://127.0.0.1/
          '''
      }
    }

    stage('Test') {
      steps {
        sh '''
          set -eu
          curl -f http://localhost/ || curl -f http://127.0.0.1/
          '''
      }
    }
  }

  post {
    success { echo 'Déploiement réussi : le site est opérationnel.' }
    failure { echo 'Échec du pipeline : voir les logs des étapes pour le détail.' }
    always {
      sh '''
        set +e
        echo "Cleaning up Apache installation..."
        sudo systemctl stop apache2 || true
        sudo apt-get purge -y apache2 apache2-bin || true
        sudo apt-get autoremove -y || true
        sudo rm -rf /var/www/html /var/lib/apache2 /etc/apache2 /var/log/apache2 || true
        echo "Cleanup done."
        '''
    }
  }
}
