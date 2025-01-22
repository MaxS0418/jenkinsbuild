pipeline {
    agent any
    
    environment {
        MODEL_URL = ""
        DOWNLOAD_DIR = ""
        S3_FILE_NAME = ""
        
    }
    stages {
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
                            ),
                            string(
                                name: 'S3_FILE_NAME', 
                                defaultValue: '', 
                                description: 'Enter the file you want to save the output in S3'
                            )
                        ]
                    )
                    
                    // Save the user input values into the variables
                    MODEL_URL = userInputs['MODEL_URL']
                    DOWNLOAD_DIR = userInputs['DOWNLOAD_DIR']
                    S3_FILE_NAME = userInputs['S3_FILE_NAME']
                    
                    // Output the entered values
                    echo "User entered Model URL: ${MODEL_URL}"
                    echo "User entered Directory: ${DOWNLOAD_DIR}"
                    echo "User entered Directory: ${S3_FILE_NAME}"
                }
            }
        }
        
        stage('Setup') {
            steps {
                script {
                    // Ensure Python and necessary packages are available
                    sh """
                    python3 -m venv venv
                    source venv/bin/activate
                    pip install --upgrade pip
                    pip install picklescan
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
        stage('Upload to S3') {
            steps {
                withAWS(credentials: 'jenkinss3access', region: 'us-east-2'){
                    s3Upload (
                        acl: 'Private' , 
                        bucket: 'jenkinsoutput041891' ,
                        file: "${DOWNLOAD_DIR}",
                        path: "${S3_FILE_NAME}"
                    )
                }
                script {
                    // Check if PickleScan generated a report and verify the results
                    sh """
                    rm -r ${DOWNLOAD_DIR}
                    """
                }
            }
        }
        
        /*stage('Cleanup local') {
            steps {
               script {
                    // Check if PickleScan generated a report and verify the results
                    sh """
                    rm -r ${DOWNLOAD_DIR}
                    """
                }
            }
        }*/
    }

    post {
        always {
            echo "Pipeline finished."
        }
    }
}
