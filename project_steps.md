## In this project I utilized Jenkins to setup a CI/CD pipeline for a simple node.js application

<br>


1. I created a project directory and intialized it

```bash
mkdir /home/linux/myapp

cd /home/linux/myapp

npm init -y

npm install express

npm install --save-dev jest supertest
```

<br>

2. I then created a node.js application

```bash
vi app.js

# I pasted the following into the app.js file
    const express = require('express');
    const app = express();

    app.get('/', (req, res) => res.send('Hello from CI/CD pipeline!'));

    module.exports = app;

    if (require.main === module) {
    app.listen(3000, () => console.log('Running on port 3000'));
    }
```

<br>


3. I created an app.test.js file

```bash
vi app.js

# I pasted the following code into the file
    const request = require('supertest');
    const app = require('./app');

    test('GET / returns 200', async () => {
        const res = await request(app).get('/');
        expect(res.statusCode).toBe(200);
    });
```

<br>


4. I added a test script to the package.json file in the project directory

```bash
vi package.json

# Added the folliwing script
    "scripts": {
        "test": "jest",
        "start": "node app.js"
    }
    

    Create `.gitignore`:
    
    node_modules/
```

<br>

5. I then pushed the project dir to GitHub

```bash
git config --global user.name "name"

git config --global user.email "email"

git init

git add .

git commit -m "Initial commit"

git remote add origin https://github.com/yourusername/yourrepo.git

git push -u origin main
```

<br>


6. I installed Java and Jenkins on my ubuntu VM

```bash
sudo apt update

sudo apt install -y openjdk-25-jdk

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key -o /tmp/jenkins.key

gpg --dearmor < /tmp/jenkins.key | sudo tee /usr/share/keyrings/jenkins-keyring.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.gpg] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update

sudo apt install -y Jenkins

sudo systemctl start jenkis 
```

<br>

7. I installed Node.js and PM2 (for running the app) on the Jenkins server

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

sudo apt install -y nodejs

sudo apt install -g pm2

pm2 startup
```

<br>


8. Since my ubuntu VM didn't have a GUI, I accessed Jenkins UI via my local machine using an SSH tunnel

```bash
ssh -L 8080:localhost:8080 user@vm_ip_address
```

<br>


9. I opened  http://localhost:8080 in my local browser, getting the initial admin password from...

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

- Completed the setup wizard - installed the plugins when prompted


<br>


10. I then configured Jenkins

- Installed the NodeJS plugin
--> Go to manage Jenkins --> Plugins --> Available Plugins --> search NodeJS --> install --> restart Jenkins

- Configured NodeJS tool
--> Go to manage Jenkins --> Tools --> NodeJS installations --> Add NodeJS --> Name it exactly "Node25" (or whatever version chosen)

- I set the executors
--> Go to manage Jenkins --> Nodes --> Built-in Node --> Configure --> set Number of executors to 2 --> save

- I added GitHub credentials
--> Go to manage Jenkins --> Credentials --> Global --> Add Credentials --> Username with password

    Username: your GitHub username
    Password: a GitHub personal token (generate at GitHub --> settings --> Developer Settings --> Personal Access Tokens --> repo scope)
    ID: github-credentials

- Added deploy SSH credentials: Generated a key pair on the Jenkins server

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/deploy_key
```

- Copied the public key to the deploy server

```bash
cat ~/.ssh/deploy_key.pub >> ~/.ssh/authorized_keys
```

--> Go to Manage Jenkins --> Credentials --> Global --> Add Credentials --> SSH Username with private key

    ID: deploy-ssh-key
    Username: Ubuntu
    Private key: paste contents of ~/.ssh/deploy_key

<br>


11. I created the Jenkinsfile in project directory

```bash
vi Jenkinsfile

# Added the following to the Jenkinsfile
    pipeline {
        agent any

        tools {
            nodejs 'NodeJS'
        }

        environment {
            DEPLOY_SERVER = 'user@your-server-ip'
            DEPLOY_PATH = '/var/www/myyapp'
        }

        stages {
            stage('Checkout') {
                steps {
                    echo 'Pulling source code...'
                    checkout scm
                }
            }

            stage('Install Dependencies') {
                steps {
                    sh 'npm ci'
                }
            }

            stage('Run Tests') {
                steps {
                    sh 'np test'
                }
            }

            stage('Build Artifact') {
                steps {
                    sh 'tar --exclude=node_modules --exclude=.git --exclude=myapp.tar.gz -czf myapp.tar.gz .'
                    archiveArtifacts artifacts: 'myapp.tar.gz', fingerprint: true
                }
            }

            stage('Deploy') {
                when {
                    branch 'main'
                }
                
                steps {
                    sshagent(['deploy-ssh-key']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER} 'mkdir -p ${DEPLOY_PATH}'
                            scp myapp.tar.gz ${DEPLOY_SERVER}:${DEPLOY_PATH}/
                            ssh ${DEPLOY_SERVER} '
                                cd ${DEPLOY_PATH} &&
                                tar -xzf myapp.tar.gz &&
                                npm ci --production &&
                                pm2 restart myapp || pm2 start app.js --name myapp
                            '
                        """
                    }
                }
            }
        }

        post {
            success {
                echo 'Pipeline completed successfully!'
            }
            
            failure {
                echo 'Pipeline failed — check the logs above.'
            }
        }
    }
```


<br>


12. I pushed the Jenkinsfile to GitHub

```bash
git add Jenkinsfile

git status

git commmit -m "Added Jenkinsfile"

git push origin main
```


<br>


13. I created the Jenkins pipeline job

--> In Jenkins --> New Item --> Pipeline --> name it 'myapp-pipeline' --> OK

- Under Build Triggers select **Poll SCM** and set:
    H/5 * * * *

- Under Pipeline select Pipeline script from SCM --> Git --> paste the repo URL --> set branch to "*/main" --> set credentials to github-credentials --> save


<br>


14. Prepared the Deploy Target

- On the deploy server created the deployment directory

```bash
sudo mkdir -p /var/www/myapp

sudo chown username:username /var/www/myapp
```

<br>


15. Ran the pipeline

- Clicked Build Now in Jenkins (all 5 stages should complete successfully - Checkout, Install Dependencies, Run Tests, Build Artifact, Deploy)


** Now, everytime I push code to the main branch Jenkins will automatically detect the change withing 5 minutes, run the pipeline, and deploy the app if all tests pass


