pipeline {
    agent any

    environment {
        CLIENT_ID = 'your_client_id'
        CLIENT_SECRET = 'your_client_secret'
        JAMF_URL = 'https://dummy.jamfcloud.com'
        BITBUCKET_REPO_URL = 'https://username:password@bitbucket.org/username/repository.git'
    }

    stages {
        stage('Get Access Token') {
            steps {
                script {
                    // Obtain access token
                    def accessToken = getAccessToken(CLIENT_ID, CLIENT_SECRET, JAMF_URL)
                    
                    // Fetch data using the access token
                    def fetchDataResponse = fetchData(accessToken)
                    
                    // Push data to Bitbucket
                    pushToBitbucket(fetchDataResponse)
                }
            }
        }
    }
    
    // Function to obtain access token
    def getAccessToken(clientId, clientSecret, jamfUrl) {
        def response = sh(script: """
            curl -i -X 'POST' '${jamfUrl}/api/oauth/token' \\
            -H 'accept: application/x-www-form-urlencoded' \\
            -d 'grant_type=client_credentials&client_id=${clientId}&client_secret=${clientSecret}'
        """, returnStdout: true).trim()
        
        // Extract access token from response
        def accessToken = response =~ /"access_token":"([^"]+)"/
        
        if (accessToken) {
            return accessToken[0][1]
        } else {
            error "Failed to obtain access token"
        }
    }
    
    // Function to fetch data using access token
    def fetchData(accessToken) {
        def fetchDataResponse = sh(script: """
            curl -s -H 'Authorization: Bearer ${accessToken}' \\
            ${JAMF_URL}/cloud-azure/id
        """, returnStdout: true).trim()
        
        return fetchDataResponse
    }
    
    // Function to push data to Bitbucket
    def pushToBitbucket(data) {
        sh """
            echo '${data}' > temp_data.txt
            git config --global user.email "jenkins@example.com"
            git config --global user.name "Jenkins"
            git init
            git add temp_data.txt
            git commit -m "Update data"
            git push ${BITBUCKET_REPO_URL} master
        """
    }
}
