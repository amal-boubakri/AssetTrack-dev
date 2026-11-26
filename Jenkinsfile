pipeline {
    agent any

    options {
        timestamps()
    }

    stages {
        stage('1. Checkout') {
            steps {
                checkout scm
            }
        }

        stage('2. Backend (CI)') {
            steps {
                dir('backend') {
                    bat '''
                        npm ci
                        npm test --if-present
                    '''
                }
            }
        }

        stage('3. Frontend (CI)') {
            steps {
                dir('frontend') {
                    bat '''
                        npm ci
                        npm run build
                    '''
                }
            }
        }

        stage('4. Deploy (Backend + Frontend)') {
            steps {
                powershell '''
                    # 1. Arrete les anciens processus Node/Vite lies au projet
                    Get-CimInstance Win32_Process -Filter "Name = 'node.exe'" |
                        Where-Object { $_.CommandLine -like "*AssetTrack-dev-CI*" -or $_.CommandLine -like "*server.js*" -or $_.CommandLine -like "*vite*" } |
                        ForEach-Object { Stop-Process -Id $_.ProcessId -Force -ErrorAction SilentlyContinue }

                    Start-Sleep -Seconds 2

                    # 2. Copie le fichier .env local dans le workspace Jenkins
                    $localEnv = "C:\\Users\\DELL\\Documents\\AssetTrack-dev\\backend\\.env"
                    $targetEnv = "$pwd\\backend\\.env"
                    if (Test-Path $localEnv) {
                        Copy-Item $localEnv $targetEnv -Force
                        Write-Output "Fichier .env copie avec succes."
                    } else {
                        Write-Warning "Attention : $localEnv introuvable."
                    }

                    # 3. Lance le Backend et le Frontend via taches planifiees (independantes de Jenkins)
                    schtasks /Run /TN "AssetTrack-Backend"
                    schtasks /Run /TN "AssetTrack-Frontend"

                    Start-Sleep -Seconds 4
                    Write-Output "Taches AssetTrack-Backend et AssetTrack-Frontend declenchees."
                '''
            }
        }
    }

    post {
        success {
            echo 'AssetTrack CI/CD : build et deploiement complets (Backend + Frontend) reussis.'
        }
        failure {
            echo 'AssetTrack CI/CD : echec. Consulter la console Jenkins.'
        }
    }
}