pipeline {
    agent any

 

    environment {
        APIM_USERNAME = 'admin'
        APIM_PASSWORD = 'admin'
        API_NAME      = 'SwaggerPetstore'
        API_VERSION   = '1.0.7'
        GIT_REPO_SSH  = 'git@github.com:shashikab/teamP.git'
        TARGET_BRANCH = 'dev'
        OAS_FILE      = '/Users/shashika/.jenkins/workspace/InitializeProject/git-init/swagger.yaml'
    }

    stages {
        stage('Install apictl') {
            steps {
                echo "Installing the apictl tool..."
                sh '''
                    curl -L -o apictl.tar.gz \
                      "https://github.com/wso2/product-apim-tooling/releases/download/v4.4.1/apictl-4.4.1-darwin-arm64.tar.gz"
                    mkdir -p /tmp/apictl
                    tar -xzvf apictl.tar.gz -C /tmp/apictl

                    if [ -f "/tmp/apictl/apictl" ]; then
                      mv /tmp/apictl/apictl ~/apictl
                    else
                      mv /tmp/apictl/apictl/apictl ~/apictl
                    fi

                    chmod +x ~/apictl
                    export PATH=~/apictl:$PATH
                    ~/apictl version
                '''
            }
        }

        stage('Push to Main') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: '0974d876-1b01-46b2-bf2c-4f3cdecd3502',
                    usernameVariable: 'GIT_USERNAME',
                    passwordVariable: 'GIT_PASSWORD'
                )]) {
                    sh '''
                        # 1) Fresh clone
                        rm -rf git-init
                        git clone "${GIT_REPO_SSH}" git-init

                        # 2) Copy out swagger.yaml for init
                        mkdir -p "${WORKSPACE}/tmp"
                        cp -R "${WORKSPACE}/git-init/swagger.yaml" "${WORKSPACE}/tmp"

                        # 3) Initialize the API in the cloned workspace
                        ~/apictl init "${WORKSPACE}/git-init/SampleAPI" \
                          --oas "${OAS_FILE}" \
                          --force

                        # 4) Clean up old swagger.yaml from git-init
                        if [ -f "${WORKSPACE}/git-init/swagger.yaml" ]; then
                          echo "Removing old swagger.yaml from git-init…"
                          rm -f "${WORKSPACE}/git-init/swagger.yaml"
                        fi

                        # 5) Move into repo root for Git operations
                        cd "${WORKSPACE}/git-init"

                        # 6) Configure Git user
                        git config user.name  "${GIT_USERNAME}"
                        git config user.email "${GIT_USERNAME}@users.noreply.github.com"

                        # 7) Switch remote to HTTPS with embedded credentials and checkout branch
                        git remote set-url origin \
                          "https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/shashikab/teamP.git"
                        git fetch origin ${TARGET_BRANCH} || true
                        git checkout -B ${TARGET_BRANCH} origin/${TARGET_BRANCH}

                        # 8) Stage, commit & push changes
                        git add --all
                        git diff-index --quiet HEAD || \
                          git commit -m "Automated export of ${API_NAME} v${API_VERSION}, build #${BUILD_NUMBER}"
                        git push --set-upstream origin ${TARGET_BRANCH}
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
    }
}