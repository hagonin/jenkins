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
      steps {
        checkout scm
      }
    }

    stage('Backup current site') {
      steps {
        //backup before deploying
        sh '''
            set -eux
            sudo rm -rf /var/www/html/*
            # Backup current web root (don’t fail if it doesn’t exist)
            sudo cp -r . /var/www/html/
        '''
      }
    }

    stage('Deploy') {
      steps {
        // Copy workspace files into Apache web root
        sh '''
          set -eux
          curl -f http://localhost/ || curl -f http://127.0.0.1/
        '''
      }
    }

    stage('Test') {
      steps {
        // Verify deployment is serving content
        sh '''
          set -euxo pipefail
          # Fail (-f) if HTTP status is 4xx/5xx; try localhost then loopback
          curl -f http://localhost/ || curl -f http://127.0.0.1/
        '''
      }
    }
  }

  post {
    success {
      echo 'Déploiement réussi : le site est opérationnel.'
    }
    failure {
      echo 'Échec du pipeline : voir les logs des étapes pour le détail.'
    }
    always {
      sh '''
      set +e
        sudo rm -rf /var/www/html/* || true
        sudo apt-get remove -y apache2 || true
        sudo apt-get autoremove -y || true
      '''
    }
  }
}
