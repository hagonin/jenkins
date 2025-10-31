pipeline {
  agent {
        label "reviewer-code"
    }

  stages {

    stage('Dependancies') {
      steps {
        sh '''
          set -euxo pipefail
          sudo apt-get update -y
          sudo apt-get install -y apache2 curl
          # Start Apache (systemd or SysV as fallback)
          (sudo systemctl start apache2 || sudo service apache2 start)
          # Enable on boot (non-fatal if unsupported)
          (sudo systemctl enable apache2 || true)
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
          set -euxo pipefail
          # Backup current web root (don’t fail if it doesn’t exist)
          sudo cp -r /var/www/html /var/www/html.backup || true
        '''
      }
    }

    stage('Deploy') {
      steps {
        // Copy workspace files into Apache web root
        sh '''
          set -euxo pipefail
          # Clean previous content
          sudo rm -rf /var/www/html/*
          # Copy everything from the Jenkins workspace to the web root
          sudo cp -r . /var/www/html/
          # Ensure index is world-readable (optional hardening)
          sudo chmod -R a+rX /var/www/html
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
        set -euxo pipefail
        # Example clean-up: remove copied files (keeps Apache logs/configs)
        sudo rm -rf /var/www/html/* || true
        # Optionally uninstall Apache if you want a clean VM after each run
        sudo apt-get remove -y apache2 || true
        sudo apt-get autoremove -y || true
      '''
    }
  }
}
