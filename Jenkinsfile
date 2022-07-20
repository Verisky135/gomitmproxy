pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: go
            image: golang:1.16
            command:
            - sleep
            args:
            - 999999
            tty: true
            resources:
              limits: {}
              requests:
                memory: "100Mi"
                cpu: "100m"
          - name: semgrep
            image: returntocorp/semgrep:0.100.0
            command:
            - sleep
            args:
            - 999999
            tty: true
            resources:
              limits: {}
              requests:
                memory: "100Mi"
                cpu: "100m"
        '''
    }
  }
  stages {
    stage("Semgrep Test") {
      steps {
        container("semgrep") {
            sh 'semgrep ci --json --config=https://configmap.astronauts.id/devops/semgrep/dev/rules.yaml > gl-sast-report.json || true'
        }
        container("go") {
          dir("semgrep-api-elastic") {
            checkout([$class: 'GitSCM', branches: [[name: "main"]], userRemoteConfigs: [[url: 'https://github.com/wahyuhadi/semgrep-api-elastic']]])
            sh "go build"
            sh "cat ../gl-sast-report.json | ./semgrep-to-elastic -r $GIT_URL -h $SEMGREP_API_URI -b false"
          }
        }
      }
    }
    
    stage("Build and Test") {
      steps {
        checkout scm
        // container('go') {
        //   withCredentials([usernamePassword(credentialsId: 'github-token', usernameVariable: 'USERNAME', passwordVariable: 'GITHUB_TOKEN')]) {
        //     sh 'git config --global url."https://${GITHUB_TOKEN}:x-oauth-basic@github.com/".insteadOf "https://github.com/"'
        //   }
        //   sh 'ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime'
        //   sh 'go get -v golang.org/x/tools/cmd/goimports'
        //   sh 'go mod download golang.org/x/net'
        //   sh 'go mod tidy'
        //   sh 'go build -v -o platform_location_be_http ./cmd/http/main.go'
        //   sh 'go build -v -o platform_location_be_grpc ./cmd/grpc/main.go'
        //   sh "USE_DOCKER_TEST=false go test -timeout=300s -coverprofile=cover.out -race -gcflags=all=-l ./..."
        // }
      }
    }
    stage('Gosec Scan') {
      steps {
          echo ""
        // container('gosec') {
        //   withCredentials([usernamePassword(credentialsId: 'github-token', usernameVariable: 'USERNAME', passwordVariable: 'GITHUB_TOKEN')]) {
        //     sh 'git config --global url."https://${GITHUB_TOKEN}:x-oauth-basic@github.com/astronautsid".insteadOf "https://github.com/astronautsid"'
        //     sh 'gosec -no-fail -fmt=sonarqube -out gosec-results.json ./...'          
        //   } 
        // }
      }
    }
    stage("Sonar Scanner") {
      environment {
        scannerHome = tool 'sonar-scanner'
      }
      steps {
          echo ""
        // container('jnlp') {
        //   withSonarQubeEnv('astro-sonar') {
        //     sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=sonar.properties"
        //   }
        // }
      }
    }

    stage("Quality Gate") {
      steps {
        script {
            echo ""
        //   timeout(time: 1, unit: 'MINUTES') {
        //     sleep(10)
        //     waitForQualityGate abortPipeline: true   
        //   }
        }
      }
    }
  }
}