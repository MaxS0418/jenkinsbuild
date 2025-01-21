pipeline {
    agent any
    
    environment {
        MODEL_URL = ""
        DOWNLOAD_DIR = ""
    }
    stages {
        stage('Setup') {
            steps {
                script {
                    // Ensure Python and necessary packages are available
                    sh """
                    python3 -m venv venv
                    source venv/bin/activate
                    pip install --upgrade pip
                    pip install picklescan
                    """
                }
            }
        }
        
         stage('User Input') {
            steps {
                script {
                    // Ask the user to input values for the variables
                    def userInputs = input(
                        message: 'Please provide values for the variables.',
                        parameters: [
                            string(
                                name: 'MODEL_URL', 
                                defaultValue: '', 
                                description: 'Enter the URL of the Model you want to scan'
                            ),
                            string(
                                name: 'DOWNLOAD_DIR', 
                                defaultValue: '', 
                                description: 'Enter the directory location you want to save the model'
                            )
                        ]
                    )
                    
                    // Save the user input values into the variables
                    MODEL_URL = userInputs['MODEL_URL']
                    DOWNLOAD_DIR = userInputs['DOWNLOAD_DIR']
                    
                    // Output the entered values
                    echo "User entered Model URL: ${MODEL_URL}"
                    echo "User entered Directory: ${DOWNLOAD_DIR}"
                }
            }
        }
        
        stage('Prepare Environment') {
            steps {
                script {
                    // Ensure the destination directory exists
                    sh """
                    mkdir -p ${DOWNLOAD_DIR}
                    """
                }
            }
        }

        stage('Download Model') {
            steps {
                script {
                    // Download the model file from Hugging Face using curl
                    sh """
                    echo 'Downloading model from Hugging Face...'
                    curl -L ${MODEL_URL} -o ${DOWNLOAD_DIR}/model.pkl
                    """
                }
            }
        }
        stage('Verify Download') {
            steps {
                script {
                    // Verify if the model file has been successfully downloaded
                    sh """
                    if [ -f "${DOWNLOAD_DIR}/model.pkl" ]; then
                        echo "Model downloaded successfully!"
                    else
                        echo "Model download failed!"
                        exit 1
                    fi
                    """
                }
            }
        }
        
         stage('Scan Model with PickleScan') {
            steps {
                script {
                    // Run PickleScan on the downloaded .pkl file to scan it
                    echo "Scanning the downloaded model with PickleScan..."
                    sh """
                    source venv/bin/activate
                    python -m picklescan -p ${DOWNLOAD_DIR} > ${DOWNLOAD_DIR}/scan_report.txt
                    """
                }
            }
        }

        stage('Verify Scan Results') {
            steps {
                script {
                    // Check if PickleScan generated a report and verify the results
                    sh """
                    if [ -f "${DOWNLOAD_DIR}/scan_report.txt" ]; then
                        echo "PickleScan report generated successfully:"
                        cat ${DOWNLOAD_DIR}/scan_report.txt
                    else
                        echo "PickleScan scan failed or no report found!"
                        exit 1
                    fi
                    """
                }
            }
        }
        
    }

    post {
        always {
            echo "Pipeline finished."
        }
    }
