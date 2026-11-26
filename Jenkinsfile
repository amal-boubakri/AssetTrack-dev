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
                    # 1. Arrete uniquement les anciens processus lies a AssetTrack-dev-CI
                    Get-CimInstance Win32_Process |
                        Where-Object {
                            ($_.Name -eq 'node.exe' -or $_.Name -eq 'cmd.exe') -and
                            ($_.CommandLine -like "*AssetTrack-dev-CI*" -or $_.CommandLine -like "*start-backend.bat*" -or $_.CommandLine -like "*start-frontend.bat*")
                        } |
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

                    # 3. Lance le Backend et le Frontend via taches planifiees
                    schtasks /Run /TN "AssetTrack-Backend"
                    schtasks /Run /TN "AssetTrack-Frontend"

                    Start-Sleep -Seconds 5
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