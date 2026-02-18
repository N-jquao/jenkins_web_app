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
